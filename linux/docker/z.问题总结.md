1. 问题一

~~~
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: failed (Result: start-limit) since 一 2019-03-11 09:43:55 CST; 7s ago
     Docs: https://docs.docker.com
  Process: 11870 ExecStart=/usr/bin/dockerd (code=exited, status=1/FAILURE)
 Main PID: 11870 (code=exited, status=1/FAILURE)
~~~

​	解决：

~~~
查看文件系统 /etc/docker/daemon.json 有没有这个文件，没有测创建它包括二级目录 docker
在daemon.json文件中输入以下内容:
{
"storage-driver"
:
"devicemapper"
}
~~~

