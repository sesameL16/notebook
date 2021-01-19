1. 反向代理

    ~~~
    在http段下加入
    	upstream hello{
    	        server 47.101.155.121:8080 weight=1 max_fails=3 fail_timeout=60s; 
    	        server 47.101.69.82:8080 weight=1 max_fails=3 fail_timeout=60s;
    	        server 47.102.125.242:8080 weight=1 max_fails=3 fail_timeout=60s;
    	 }
    
    server端配置
         location / {
            #root   html;
            #index  index.html index.htm;
    	    least_conn；#最少连接
    	    proxy_pass   http://hello;
    	    proxy_set_header Host $host;
                 proxy_set_header X-Real-IP $remote_addr;
         }
    
    ~~~

    

2. 安装ssl模块

    ~~~
    yum -y install openssl openssl-devel
    ./configure --prefix=/usr/local/nginx --with-http_ssl_module
    make&&make install
    ~~~

3. 日志配置

    ~~~
    Nginx 日志
    access.log 记录哪些用户,哪些页面以及用户浏览器,IP等访问信息；
    error.log 记录服务器错误的日志
    
    配置日志存储路径
    location / {
    	access_log          /usr/local/nginx/logs/access.log;
    	error_log           /usr/local/nginx/logs/error.log;
    }
    按自己要求配置日志格式
    http {
    	include       mime.types;
    	default_type  application/octet-stream;
    	sendfile        on;
    	keepalive_timeout  60;
    	include  /usr/local/nginx/vhost/*.conf;
    	log_format main '$remote_addr -$remote_user [$time_local] "request"'
    					'$status $body_bytes_sent "$http_referer"'
    					'"$http_user_agent" "$http_x_forwarded_for"'
    					'"$gzip_ratio" $request_time $request_length' ;
    	open_log_file_cache max=1000 inactive=60s;
    }
    
    操作完上面的，日志就按自己的要求格式存储在指定位置
    
    ~~~

    

4. 日志切割(按天进行日志切割)

    ~~~shell
    #编写脚本
    #!/bin/bash
    year=`date +%Y`
    month=`date +%m`
    day=`date +%d`
    logs_backup_path="/usr/local/nginx/logs_backup/$year$month"    #日志存储路径
    logs_path="/usr/local/nginx/logs/"                             #要切割的日志路径
    logs_access="access"                                           #要切割的日志
    logs_error="error"
    pid_path="/usr/local/nginx/logs/nginx.pid"                     #nginx的pid
    [ -d $logs_backup_path ]||mkdir -p $logs_backup_path
    rq=`date +%Y%m%d`
    #mv ${logs_path}${logs_access}.log ${logs_backup_path}/${logs_access}_${rq}.log
    mv ${logs_path}${logs_error}.log ${logs_backup_path}/${logs_error}_${rq}.log
    kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
    
    #做定时任务
    crontab –e
    59 23 * * * bash /usr/local/nginx/shell/cut_ngnix_log.sh   #每天23：59分开始执行；
    
    ~~~

    