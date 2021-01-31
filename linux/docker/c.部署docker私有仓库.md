1. 用官方的镜像来创建docker私有仓库：

   docker run -d -v /opt/registry:/var/lib/registry -p 5000:5000 --restart=always --name docker-registry registry

2. 查看一下私有仓库已经起来了：

   | [root@sesame1 ~]# docker ps -a                               |
   | ------------------------------------------------------------ |
   | CONTAINER ID    IMAGE        COMMAND         CREATED       STATUS       PORTS          NAMES  98df2913e31e    registry      "/entrypoint.sh /etc…"  9 seconds ago    Up 6 seconds    0.0.0.0:5000->5000/tcp  docker-registry |

3. 配置镜像源为国内官方源

   | vim /etc/docker/daemon.json                                  |
   | ------------------------------------------------------------ |
   | {     "registry-mirrors": [ "https://registry.docker-cn.com"],     "insecure-registries": [ "192.168.23.10:5000"]  } |

4. 重启docker服务

   systemctl restart docker

5. 查看本地已有的镜像：docker images

   将my/centos标记为 192.168.23.10:5000/centos;

   使用命令：docker tag 90a93df7436d 192.168.23.10:5000/centos

 

6. 接下来，我们可以到另一台机器192.168.23.20下载160上传的192.168.23.10:5000/centos镜像：

   docker pull 192.168.23.10:5000/centos

 

7. 查看Registry中所有镜像信息
    $ curl -XGET http://192.168.23.10:5000/v2/_catalog
    {"repositories":["smart-core","smart-gateway"]}
    下载镜像
    $ docker pull 192.168.23.10:5000/smart-core
    $ docker pull 192.168.23.10:5000/smart-gateway

 

8. 开启docker远程服务

   vi /usr/lib/systemd/system/docker.service

   在[Service]部分的最下面添加下面两行:

~~~
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
~~~



9. 然后让docker重新读取配置文件,并重启docker服务.

   systemctl daemon-reload

   systemctl restart docker

10. 查看docker进程,发现docker守护进程在已经监听2375的tcp端口了

    [root@bogon ~]# ps -ef|grep docker

    root   1956   1 3 16:32 ?    00:00:00 /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

    root   1959 1956 0 16:32 ?    00:00:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc

    root   2056 1956 0 16:32 ?    00:00:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 5000 -container-ip 172.17.0.2 -container-port 5000

    root   2062 1959 0 16:32 ?    00:00:00 docker-containerd-shim ffac4cc26314b42610071b521cbd9c39e00618f4277ca967c1fee31193ff4041 /var/run/docker/libcontainerd/ffac4cc26314b42610071b521cbd9c39e00618f4277ca967c1fee31193ff4041 docker-runc

    root   2075 2062 2 16:32 ?    00:00:00 registry serve /etc/docker/registry/config.yml

    root   2103 1259 0 16:32 pts/0  00:00:00 grep --color=auto docker