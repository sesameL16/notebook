1. 下载

   wget https://artifacts.elastic.co/downloads/kibana/kibana-6.0.0-linux-x86_64.tar.gz

2. 解压

   tar -zxvf kibana-6.0.0-linux-x86_64.tar.gz  -C /usr/local/

3. 创建log文件夹

   mkdir -p /opt/data/logs/kibana

   chmod 777 /opt/data/logs/kibana

4. 修改配置文件

    vim /usr/local/kibana/config/kibana.yml

   ~~~
   # 可通过 http://192.168.46.132:5601 在浏览器访问
   server.name: "MyKibana"
   server.host: "192.168.46.132"
   server.port: 5601
    
   # 指定elasticsearch节点
   elasticsearch.url: "http://192.168.46.132:9200"
    
   pid.file: /var/run/kibana.pid
    
   # 日志目录
   logging.dest: /opt/data/logs/kibana/kibana.log
    
   # 间隔多少毫秒，最小是100ms，默认是5000ms即5秒
   ops.interval: 5000
   ~~~

5. 启动

   ./bin/kibana

   在浏览器访问：http://192.168.46.132:5601

