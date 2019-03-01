# 版本信息
FROM centos:7
MAINTAINER wuweixiang <wuweixiang.alex@gmail.com>
# 设置工作目录
WORKDIR /var/
# 添加jdk、tomcat
ADD jdk-8u191-linux-x64.tar.gz .
#ADD http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.35/bin/apache-tomcat-8.5.35.tar.gz .
ADD apache-tomcat-8.5.35.tar.gz .

# 设置环境变量
ENV JAVA_HOME /var/jdk1.8.0_191
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin
ENV TIME_ZONE Asia/Shanghai

# 更改时区
RUN set -x \
&& echo "${TIME_ZONE}" > /etc/timezone \
&& ln -sf /usr/share/zoneinfo/${TIME_ZONE} /etc/localtime

# 开启内部服务端口
EXPOSE 8080

# 启动tomcat服务器
CMD ["/var/apache-tomcat-8.5.35/bin/catalina.sh","run"] && tail -f /var/apache-tomcat-8.5.35/logs/catalina.out