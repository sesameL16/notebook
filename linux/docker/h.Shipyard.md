**安装Shipyard**

1. 关闭主机防火墙

   [root@docker-218 ~]# systemctl disable firewalld.service

   [root@docker-218 ~]# systemctl stop firewalld.service

   [root@docker-218 ~]# iptables -F

   [root@docker-218 ~]# firewall-cmd --state

   not running

 

2. 安装docker

   [root@docker-218 ~]# yum install docker

 

3. 修改docker配置文件，添加下面一行，进行docker加速设置

   [root@node-1 ~]# vim /etc/sysconfig/docker        //在文件底部添加下面一行, (这里就是直接写： xxx.mirror.aliyuncs.com)

   ADD_REGISTRY='--add-registry xxx.mirror.aliyuncs.com'

 

4. 启动docker服务

   [root@docker-218 ~]# systemctl start docker

 

5. 下载相关镜像

   这些镜像如果不提前下载,则在下面一键安装部署时会自动下载,不过要等待一段时间. 所以最好提前下载,一键部署时就很快了

   [root@docker-218 ~]# docker pull rethinkdb

   [root@docker-218 ~]# docker pull m*icrobox/etcd*

   [root@docker-218 ~]# docker pull shipyard/docker-proxy

   [root@docker-218 ~]# docker pull swarm

   [root@docker-218 ~]# docker pull dockerclub/shipyard

 

6. 下载官方一键部署脚本

   最新下载地址: https://pan.baidu.com/s/1ATM32S7tLA35Q-xK7-TgzQ  

   提取密码: kgqi

 

7. 接着执行一键部署

   替换Controller为中文版

   [root@docker-213 ~]# chmod 755 shipyard-deploy

   [root@docker-213 ~]# sh shipyard-deploy

   Deploying Shipyard

    -> Starting Database

    -> Starting Discovery

    -> Starting Cert Volume

    -> Starting Proxy

    -> Starting Swarm Manager

    -> Starting Swarm Agent

    -> Starting Controller

   Waiting for Shipyard on 172.16.60.213:8080

    

   Shipyard available at http://172.16.60.213:8080

   Username: admin Password: shipyard

 

8. 部署后,可以看到相应的shipyard容器已经创建好了

   [root@docker-218 ~]# docker ps

   CONTAINER ID    IMAGE             COMMAND         CREATED       STATUS       PORTS                      NAMES

   0cc242b4d90b    dockerclub/shipyard:latest   "/bin/controller -..."  19 seconds ago   Up 15 seconds    0.0.0.0:8080->8080/tcp              shipyard-controller

   ce08a7f0f62f    swarm:latest          "/swarm j --addr 1..."  20 seconds ago   Up 19 seconds    2375/tcp                     shipyard-swarm-agent

   9d2dd2bd5bff    swarm:latest          "/swarm m --replic..."  20 seconds ago   Up 19 seconds    2375/tcp                     shipyard-swarm-manager

   3435b5e2d13a    shipyard/docker-proxy:latest  "/usr/local/bin/run"   21 seconds ago   Up 20 seconds    0.0.0.0:2375->2375/tcp              shipyard-proxy

   315ca39f00dd    alpine             "sh"           21 seconds ago   Up 21 seconds                            shipyard-certs

   564f25ac8130    microbox/etcd:latest      "/bin/etcd -addr 1..."  22 seconds ago   Up 21 seconds    0.0.0.0:4001->4001/tcp, 0.0.0.0:7001->7001/tcp  shipyard-discovery

   bff634944376    rethinkdb           "rethinkdb --bind all"  22 seconds ago   Up 22 seconds    8080/tcp, 28015/tcp, 29015/tcp          shipyard-rethinkdb

    

   最后访问http://172.16.60.218:8080,使用admin/shipyard用户名和密码登录即可. (注意:一键部署之后,需要稍等一会儿,8080端口才能起来)

 

9. 如果想要修改web访问端口,则操作如下:

   [root@docker-218 ~]# cat shipyard-deploy |grep 8080

     echo " PORT: specify the listen port for the controller (default: 8080)"

   SHIPYARD_PORT=${PORT:-8080}

    

   比如将脚本中默认的8080端口改为80端口

   [root@docker-218 ~]# sed -i 's/8080/80/g' shipyard-deploy

    

   然后重新部署即可



**执行脚本删除Shipyard**

[root@docker-218 ~]# cat shipyard-deploy |ACTION=remove bash      

Removing Shipyard

 -> Removing Database

 -> Removing Discovery

 -> Removing Cert Volume

 -> Removing Proxy

 -> Removing Swarm Agent

 -> Removing Swarm Manager

 -> Removing Controller

Done

 

[root@docker-218 ~]# docker ps

CONTAINER ID    IMAGE        COMMAND       CREATED       STATUS       PORTS 



**添加其他节点主机**

添加节点时,一键脚本需要运行在被添加的节点主机上,而不是shipyard部署节点的机器上

cat shipyard-deploy| ACTION=node DISCOVERY=etcd://172.16.60.218:4001 bash



**删除节点机**

cat shipyard-deploy |ACTION=remove bash -s