1. 集群部署模式

   * 单 master 模式

     也就是只有一个 master 节点，称不上是集群，一旦这个 master 节点宕机，那么整个服务就不可用，适合个人学习使用。

   * 多 master 模式

     多个 master 节点组成集群，单个 master 节点宕机或者重启对应用没有影响。

     优点：所有模式中性能最高

     缺点：单个 master 节点宕机期间，未被消费的消息在节点恢复之前不可用，消息的实时性就受到影响。

     注意：使用同步刷盘可以保证消息不丢失，同时 Topic 相对应的 queue 应该分布在集群中各个节点，而不是只在某各节点上，否则，该节点宕机会对订阅该 topic 的应用造成影响。

   * 多 master 多 slave 异步复制模式

     在多 master 模式的基础上，每个 master 节点都有至少一个对应的 slave。master

     节点可读可写，但是 slave 只能读不能写，类似于 mysql 的主备模式。

     优点： 在 master 宕机时，消费者可以从 slave 读取消息，消息的实时性不会受影响，性能几乎和多 master 一样。

     缺点：使用异步复制的同步方式有可能会有消息丢失的问题。

   * 多 master 多 slave 同步双写模式

     同多 master 多 slave 异步复制模式类似，区别在于 master 和 slave 之间的数据同步方式。

     优点：同步双写的同步模式能保证数据不丢失。

     缺点：发送单个消息 RT 会略长，性能相比异步复制低10%左右。

     刷盘策略：同步刷盘和异步刷盘（指的是节点自身数据是同步还是异步存储）

     同步方式：同步双写和异步复制（指的一组 master 和 slave 之间数据的同步）

     注意：要保证数据可靠，需采用同步刷盘和同步双写的方式，但性能会较其他方式低。

   

2. 双master 模式、同步刷盘的集群部署

   假设两台服务器分别为: 192.168.1.11和192.168.1.12

   将 conf/broker.conf 拷贝到bin目录，并修改配置文件

   * [192.168.1.11] broker-a.conf

     ~~~
     #所属集群名字
     brokerClusterName=rocketmq-cluster
     #broker名字，注意此处不同的配置文件填写的不一样
     brokerName=broker-a
     #0 表示 Master， >0 表示 Slave
     brokerId=0
     #nameServer地址，分号分割
     namesrvAddr=192.168.1.11:9876;192.168.1.12:9876
     #当前broker监听的IP
     brokerIP1=192.168.1.11
     #存在broker主从时，在broker主节点上配置了brokerIP2的话,
     #broker从点会连接主节点配置的brokerIP2来同步。
     #默认不配置brokerIP1和brokerIP2时，都会根据当前网卡选择一个IP使用
     brokerIP2=192.168.1.11
     #刷盘方式 ASYNC_FLUSH - 异步刷盘SYNC_FLUSH - 同步刷盘
     flushDiskType=SYNC_FLUSH
     ~~~

   * [192.168.1.12] broker-b.conf

     ~~~
     #所属集群名字
     brokerClusterName=rocketmq-cluster
     #broker名字，注意此处不同的配置文件填写的不一样
     brokerName=broker-b
     #0 表示 Master， >0 表示 Slave
     brokerId=0
     #nameServer地址，分号分割
     namesrvAddr=192.168.1.11:9876;192.168.1.12:9876
     #当前broker监听的IP
     brokerIP1=192.168.1.12
     #存在broker主从时，在broker主节点上配置了brokerIP2的话,
     #broker从点会连接主节点配置的brokerIP2来同步。
     #默认不配置brokerIP1和brokerIP2时，都会根据当前网卡选择一个IP使用
     brokerIP2=192.168.1.12
     #刷盘方式 ASYNC_FLUSH - 异步刷盘SYNC_FLUSH - 同步刷盘
     flushDiskType=SYNC_FLUSH
     ~~~

   * 启动

     * 分别启动两个服务器上的 namesrv:

       进入bin目录执行: ./mqnamesrv &

     * 依次启动两个服务器上的broker：

        ./mqbroker -c brokcer-a.conf &

        ./mqbroker -c brokcer-b.conf &

     注: 一定要先把两个namesrv启动起来之后，才能启动broker

   * 关闭

     * 依次关闭两个服务器上的broker：

       ./mqshutdown broker

     * 依次关闭两个服务器上的namesrv：

        ./mqshutdown namesrv

     注: 一定要先把两个broker关闭之后，才能关闭namesrv

   * 端口开通

     RocketMQ的namesrv服务默认端口为: 9876；

     RocketMQ的broker服务默认端口为: 10911；

     RocketMQ的haService服务默认端口为: broker端口+1，即:如果broker端口为10911，则haService端口为10912。

     RocketMQ的fastRemotingServer服务默认端口为: 10909。

     以上端口最好使用默认，不要修改