1. docker中安装vim

   apt-get update

   apt-get install -y vim

 

2. docker 中的java web 不能访问oracle 数据库

   进入docker

   vim etc/timezone 

   把 timezone 的容器改为 ：

   Etc/CST

3. Docker容器时间与主机时间不一致

   * 共享主机的localtime

     docker run --name <name> -v /etc/localtime:/etc/localtime

   * 复制主机的localtime

     docker cp /etc/localtime:【容器ID或者NAME】/etc/localtime

   * 创建自定义的dockerfile

     FROM redis

     FROM tomcat

     ENV CATALINA_HOME /usr/local/tomcat

     \#设置时区

     RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \

      && echo 'Asia/Shanghai' >/etc/timezone \

      

     