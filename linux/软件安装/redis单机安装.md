1. 线安装gcc

   yum install gcc-c++

2. 先把redis-4.0.2.tar.gz使用xftp上传到/usr/software/下

3. 解压缩

   tar -zxvf /usr/software/redis-4.0.2.tar.gz -C /usr/local/

4. 编译。

   进入解压缩的路径下执行命令    make

5. 安装

   make install PREFIX=/usr/local/redis 

   PREFIX参数指定redis的安装目录。一般软件安装到/usr目录下

6. Redis后台启动

   修改配置文件redis.conf，将daemonize的值设为yes

   ./redis-server  redis.conf

7. 客户端连接

   ./redis-cli -h 192.168.25.210 -p 6379

8. 修改密码

   ~~~
   127.0.0.1:6379> config get requirepass
   1) "requirepass"
   2) ""
   127.0.0.1:6379> config set requirepass 531676
   OK
   127.0.0.1:6379> quit
   ~~~

   

