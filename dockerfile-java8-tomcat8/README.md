##构建镜像

进入到/javaweb-docker/dockerfile-java8-tomcat8目录下：

`docker build -t  [<仓库名>[:<标签>]] .  #创建镜像 `


仓库名：经常以 两段式路径 形式出现，比如 qianpangzi/javaweb:1.0.0，前者Docker账号用户名，后者则往往是对应的软件名。
    
标签：指定所需哪个版本的镜像，默认latest。
##使用方式
 1、新购买的服务器，安装docker，执行以下指令：

`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`
 2、创建/var/webapps/目录，并将war包放入该目录下

 3、运行以下命令，即可实现部署。

`docker run -d -p 80:8080 \
-v /var/webapps/:/var/apache-tomcat-8.5.35/webapps/ \
--name <容器名>  \
qianpangzi/javaweb:1.0.0`
 注意： 

  挂载路径 /var/webapps/  为当前war上传位置