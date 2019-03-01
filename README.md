# Dockerfile-java-web
一、mysql - Docker Hub
参考资料：https://hub.docker.com/_/mysql/

参考资料：https://store.docker.com/images/mysql

1.下载mysql镜像
docker pull mysql:5.7
2.启动mysql容器
docker run -d -p 3306:3306 --name dbmysql -e MYSQL_ROOT_PASSWORD=password -v /mysql/datadir:/var/lib/mysql -v /mysql/conf:/etc/mysql/conf.d  docker.io/mysql:5.7


#-e MYSQL_ROOT_PASSWORD=password ：指定root密码  
#-v /mysql/datadir:/var/lib/mysql ：指定数据库本地存储路径，如果系统没有关闭SELinux，会启动失败，原因是本地目录不允许挂载到容器，需要先执行chcon -Rt svirt_sandbox_file_t /mysql/datadir  
#-v /mysql/conf:/etc/mysql/conf.d ：指定使用自定义的mysql配置文件启动数据库，比如在该路径下创建一个my-config.cnf  

#提示：killall -9 mysqld 立即杀死进程 （多执行几次）
  

#vi my-config.cnf  

#[mysqld]  
#port=3306  
#character-set-server=utf8  
#wait_timeout=288000  # 链接超时，默认为8小时，单位为秒
#lower_case_table_names=1 # 不去分大小写
3.Docker-Compose方式
dbmysql:  
  image: docker.io/mysql:5.7
  ports:  
    - 3306:3306  
  environment:  
    MYSQL_ROOT_PASSWORD: password  
  volumes:  
    - /mysql/datadir:/var/lib/mysql  
    - /mysql/conf:/etc/mysql/conf.d:ro
[root@centos-linux-agent mysql]# docker-compose up -d  
Creating mysql_dbmysql_1  
……
[root@centos-linux-agent mysql]# docker ps -a  
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                    NAMES  
087f4e32cd29        docker.io/mysql:latest   "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        0.0.0.0:3306->3306/tcp   mysql_dbmysql_1
二、tomcat - Docker Hub
参考资料：https://hub.docker.com/_/tomcat/

参考资料：https://store.docker.com/images/tomcat

1.下载tomact镜像
docker pull tomcat
2.启动tomact容器
docker run -d -p 8080:8080 -v /tomcat/webapps:/usr/local/tomcat/webapps tomcat:8.5.35-jre8
这样，只需要将war包拷贝到宿主机的/tomcat/webapps下即可

3.Docker-Compose方式
version: '2'  
services:  
   db:  
     image: docker.io/mysql:latest  
     volumes:  
       - /mysql/datadir_tomcat:/var/lib/mysql  
       - /mysql/conf:/etc/mysql/conf.d:ro  
     restart: always  
     environment:  
       MYSQL_ROOT_PASSWORD: password  

   tomcat01:  
     depends_on:  
       - db  
     image: docker.io/tomcat:latest  
     volumes:  
       - /tomcat01/webapps:/usr/local/tomcat/webapps  
     links:  
       - db:db  
     ports:  
       - "8081:8080"  
     restart: always  

   tomcat02:  
     depends_on:  
       - db  
     image: docker.io/tomcat:latest  
     volumes:  
       - /tomcat02/webapps:/usr/local/tomcat/webapps  
     links:  
       - db:db  
       - tomcat01:tomcat01  
     ports:  
       - "8082:8080"  
     restart: always
上面这种方式，随便进入任何一个容器，执行ping命令，都可以互相ping通

ping tomcat01

ping db
但是查看各自的/etc/hosts，却看不到相应的配置，就是这么神奇。

所以war里使用上面的链接别名配置好互相要访问的地址，然后拷贝到对应的部署路径下，并重启。

[root@izwz9eftauv7x69f5jvi96z tomcat]# docker-compose restart
Restarting tomcat_tomcat02_1 ... done
Restarting tomcat_tomcat01_1 ... done
Restarting tomcat_db_1       ... done
三、实战中
也可以使用Dockerfile，将war包等直接封装为一个新的镜像。

sudo mkdir /dockerfile

sudo vi Dockerfile

FROM tomcat:8.5.35-jre8
MAINTAINER "wwx <wuweixiang.alex@gmail.com>"  
ADD web.war /usr/local/tomcat/webapps/
将web.war拷贝到当前路径下

# 生成镜像
docker build -t wuweixiang/tomcat8.5.35-jre8 .     # 注意最后面那个点，代表当前路径

# 启动
docker run -p 8080:8080 -d wuweixiang/tomcat8.5.35-jre8
docker-compose

tomcat01:  
     depends_on:  
       - db  
     build: /dockerfile  
     links:  
       - db:db  
     ports:  
       - "8081:8080"  
     restart: always
docker-compose up -d ：第一次执行会自动创建一个镜像，并启动容器

如果该镜像已经被创建了，再次执行时要加上--build：docker-compose up --build -d，此时会重新创建该镜像。
