**软件安装**

1. 上传或下载 zookeeper-3.4.10.tar.gz

2. 使用命令行解压缩

   tar -zxvf zookeeper-3.4.10.tar.gz -C /usr/local

3. 创建data文件夹

   mkdir data 

   创建logs文件夹

   mkdir logs

4. 修改配置文件

   cd conf 

   mv zoo_sample.cfg zoo.cfg 

   修改zoo.cfg 

   dataDir=/usr/local/zookeeper-3.4.10/data

   dataLogDir=/usr/local/zookeeper-3.4.10/logs

   修改后关闭并保存

5. 配置zookeeper的环境变量

   #zookeeper

   export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.10

   export PATH=$ZOOKEEPER_HOME/bin:$PATH

6. 启动zookeeper

   cd bin 

    ./zkServer.sh start 

   显示

   ZooKeeper JMX enabled by default 

   Using config: /usr/local/zookeeper/zookeeper-3.4.10/bin/../conf/zoo.cfg 

   Starting zookeeper ... STARTED 

7. 查看zookeeper状态

   ./zkServer.sh status

   ZooKeeper JMX enabled by default 

   Using config: /usr/local/zookeeper/zookeeper-3.4.10/bin/../conf/zoo.cfg 

   Mode: standalone 

8. 登录zookeeper

   ./zkCli.sh -server localhost:2181



**设置zookeeper开机自启动**

1. 直接修改/etc/rc.d/rc.local文件

   编辑/etc/rc.d/rc.local文件，加入java环境变量和zookeeper启动命令

   vim /etc/rc.d/rc.local 

   ~~~
   #!/bin/sh
   #
   # This script will be executed *after* all the other init scripts.
   # You can put your own initialization stuff in here if you don't
   # want to do the full Sys V style init stuff.
    
   touch /var/lock/subsys/local
   export JAVA_HOME=/usr/java/jdk1.8.0_112
   /usr/local/zookeeper-3.4.5/bin/zkServer.sh start
   ~~~

2. 把zookeeper做成服务

   * 进入到/etc/rc.d/init.d目录下，新建一个zookeeper脚本

     [root@zookeeper ~]# cd /etc/rc.d/init.d/

     [root@zookeeper init.d]# pwd

     /etc/rc.d/init.d

     [root@zookeeper init.d]# touch zookeeper

   * 给脚本添加执行权限

     [root@zookeeper init.d]# chmod +x zookeeper

   * 编辑脚本

     [root@zookeeper init.d]# vim zookeeper

     ~~~
     #!/bin/bash
     #chkconfig:2345 20 90
     #description:zookeeper
     #processname:zookeeper
     export JAVA_HOME=//usr/local/jdk1.8.0_131
     case $1 in
             start) su root /usr/local/zookeeper-3.4.10/bin/zkServer.sh start;;
             stop) su root /usr/local/zookeeper-3.4.10/bin/zkServer.sh stop;;
             status) su root /usr/local/zookeeper-3.4.10/bin/zkServer.sh status;;
             restart) su /usr/local/zookeeper-3.4.10/bin/zkServer.sh restart;;
             *) echo "require start|stop|status|restart" ;;
     esac
     ~~~

   * 测试服务

     ~~~
     [root@zookeeper init.d]# service zookeeper start
     JMX enabled by default
     Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
     Starting zookeeper ... STARTED
     [root@zookeeper init.d]# service zookeeper status
     JMX enabled by default
     Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
     Mode: standalone
     [root@zookeeper init.d]# service zookeeper stop
     JMX enabled by default
     Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
     Stopping zookeeper ... STOPPED
     ~~~

   * 添加到开机自启

     [root@zookeeper init.d]# chkconfig --add zookeeper 

     [root@zookeeper init.d]# chkconfig --list

     netconsole       0:关    1:关    2:关    3:关    4:关    5:关    6:关

     network            0:关    1:关    2:开    3:开    4:开    5:开    6:关

     zookeeper        0:关    1:关    2:开    3:开    4:开    5:开    6:关