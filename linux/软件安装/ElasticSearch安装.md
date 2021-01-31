1. 下载elasticsearch包并上传

2. 解压

   tar -zxvf elasticsearch-6.1.0.tar.gz

3. Elasticsearch配置文件config/elasticsearch.yml

   ~~~
   
   # ---------------------------------- Cluster -----------------------
   #
   # Use a descriptive name for your cluster:
   #
    cluster.name: synwayES
   #
   # ------------------------------------ Node ------------------------------------
   #
   # Use a descriptive name for the node:
   #
    node.name: 10.1.12.15
   #
   # Add custom attributes to the node:
   #
   #node.attr.rack: r1
   #
   # ----------------------------------- Paths ------------------------
   #
   # Path to directory where to store the data (separate multiple locations by comma):
   #
    path.data: /home/SynQbfxSystem/ElasticSearch/Data
   #
   # Path to log files:
   #
    path.logs: /home/SynQbfxSystem/ElasticSearch/Logs
   #
   # ----------------------------------- Memory -----------------------------------
   #
   # Lock the memory on startup:
   #
    bootstrap.memory_lock: false
    bootstrap.system_call_filter: false
   #
   # Make sure that the heap size is set to about half the memory available
   # on the system and that the owner of the process is allowed to use this
   # limit.
   #
   # Elasticsearch performs poorly when the system is swapping the memory.
   #
   # ---------------------------------- Network -----------------------------------
   #
   # Set the bind address to a specific IP (IPv4 or IPv6):
   #
    network.host: 10.1.12.15
    transport.tcp.port: 9300
   #
   # Set a custom port for HTTP:
   #
    http.port: 9200
   #
   # For more information, consult the network module documentation.
   #
   # --------------------------------- Discovery ----------------------------------
   #
   # Pass an initial list of hosts to perform discovery when new node is started:
   # The default list of hosts is ["127.0.0.1", "[::1]"]
   #
    discovery.zen.ping.unicast.hosts: ["10.1.12.15"]
   #
   # Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
   #
    discovery.zen.minimum_master_nodes: 1
   #
   # For more information, consult the zen discovery module documentation.
   #
   # ---------------------------------- Gateway -----------------------------------
   #
   # Block initial recovery after a full cluster restart until N nodes are started:
   #
   #gateway.recover_after_nodes: 3
   #
   # For more information, consult the gateway module documentation.
   #
   # ---------------------------------- Various -----------------------------------
   #
   # Require explicit names when deleting indices:
   #
   #action.destructive_requires_name: true
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    
    action.auto_create_index: true
    xpack.monitoring.collection.enabled: false
    xpack.graph.enabled: false
    xpack.ml.enabled: false
    xpack.monitoring.enabled: false
    xpack.security.enabled: false
    xpack.watcher.enabled: false
    
    
    #错误描述：An HTTP line is larger than 4096 bytes 
    #默认情况下ES对请求参数设置为4k
    http.max_initial_line_length: "8k"
    http.max_header_size: "16k"
   ~~~

4. 创建用户赋予权限（Elasticsearch不能用root用户来启动）

   useradd elkuser

   chown -R elkuser.elkuser elasticsearch-6.1.0

5. 修改linux中elasticsearch最大文件打开数太小

   * 修改/etc/security/limits.conf文件，添加或修改如下行：

     hard  nofile      65536

     soft  nofile      65536

   * 修改 /etc/sysctl.conf 文件,添加如下行

     vm.max_map_count=262144

   * 修改好了以后，运行/sbin/sysctl -p

6. 整内存大小config/jvm.options的参数

   vi jvm.options

   -Xms2g

   -Xmx2g

7. 启动

   用创建的用户登录

   cd bin

   nohup ./elasticsearch &

   浏览器访问 http://localhost:9200

