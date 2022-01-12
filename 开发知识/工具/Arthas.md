### 安装

* 下载

  wget https://alibaba.github.io/arthas/arthas-boot.jar

* 启动

  java -jar arthas-boot.jar

* 选择进程

  运行 `arthas-boot` 后，控制台会显示所有 Java 进程，选择一个你需要诊断的进程。

  如第二步所示，这里有只有一个 Java 进程，输入序号1，回车，Arthas会附到目标进程上，并输出日志：

  ~~~
  [INFO] Start download arthas from remote server: https://maven.aliyun.com/repository/public/com/taobao/arthas/arthas-packaging/3.1.1/arthas-packaging-3.1.1-bin.zip
  [INFO] Download arthas success.
  [INFO] arthas home: /root/.arthas/lib/3.1.1/arthas
  [INFO] Try to attach process 13062
  [INFO] Attach process 13062 success.
  [INFO] arthas-client connect 127.0.0.1 3658
    ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.                       
   /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'                    
  |  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.                     
  |  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |                    
  `--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'                      
  
  wiki      https://alibaba.github.io/arthas                                 
  tutorials https://alibaba.github.io/arthas/arthas-tutorials                
  version   3.1.1                                                             
  pid       13062                                                             
  time      2019-07-30 14:49:34
  ~~~

### 实战使用

1. dashboard

   显示当前系统的实时数据面板，按 ctrl+c 即可退出。

   ![20190730154952](H:\笔记\图片\Arthas\20190730154952.png)

2.  thread

   查看当前 JVM 的线程堆栈信息。

   thread id， 显示指定线程的运行堆栈：thread 20

   ![img](H:\笔记\图片\Arthas\20190730163759.png)

   显示当前最忙的前N个线程并打印堆栈：thread -n 3

   ![img](H:\笔记\图片\Arthas\20190730163734.png)

3.  sc

   查看 JVM 已加载的类详细信息:sc -d *Test

   ![img](H:\笔记\图片\Arthas\20190730155704.png)

4.  sm

   查看已加载类的方法信息:

   sm -d cn.javastack.springbootbestpractice.SpringBootBestPracticeApplication main

   ![img](H:\笔记\图片\Arthas\20190730160555.png)

5.  jad

   反编译指定已加载类的源代码:

   jad cn.javastack.springbootbestpractice.SpringBootBestPracticeApplication

   ![img](H:\笔记\图片\Arthas\20190730155942.png)

6.  trace

   显示方法内部调用路径，非实时返回的命令并输出方法路径上的总耗时，以及的每个节点上的详细耗时:trace -j cn.javastack.springbootbestpractice.web.JsonTest getUserInfo

   -j：表示跳过 JDK 中的方法路径

   ![img](H:\笔记\图片\Arthas\20190730165157.png)

7.  monitor

   对某个方法的调用进行定时监控:

   monitor cn.javastack.springbootbestpractice.web.JsonTest getUserInfo -c 5

   -c 5：表示每5秒统计一次，统计周期，默认值为120秒

   ![img](H:\笔记\图片\Arthas\20190730170442.png)

   监控维度说明：

   | 监控项    | 说明         |
   | --------- | ------------ |
   | timestamp | 时间戳       |
   | class     | 类名         |
   | method    | 方法名       |
   | total     | 调用次数     |
   | success   | 成功次数     |
   | fail      | 失败次数     |
   | rt        | 平均响应时间 |
   | fail-rate | 失败率       |

8. watch

   观测方法执行数据，能方便的观察到指定方法的调用情况，如：返回值、抛出异常、入参等

   watch cn.javastack.springbootbestpractice.web.JsonTest getUserInfo '{params, returnObj}' -x 2 -b

   ![img](H:\笔记\图片\Arthas\20190730171659.png)

   以上监控的是一个方法的入参情况，在方法执行前监控：-b，遍历深度：-x 2

9. quit/exit

   退出当前 Arthas。

   这个命令仅退出当前连接的客户端，附到目标进程上的 Arthas 会继续运行，端口不会关闭，下次连接时可以直接连接使用

10. shutdown

	关闭 Arthas 服务端，退出所有 Arthas 客户端。
    
    以上演示了 10 个命令的基本使用，各种命令的使用详情可以在命令带 `--help` 进行查阅
    
    ![img](H:\笔记\图片\Arthas\20190730172757.png)

总结下来，使用 Arthas 可以很方便的诊断一个 Java 应用程序，如：系统数据面板、JVM实时运行状态、类加载情况、监控方法执行情况、显示方法执行路径等。

Arthas这些实用的功能确实可以帮助我们解决一些常见的线上问题，也能独立于应用程序代码，但仅局限于在一个 JVM 进程内，如果是分布式系统，Arthas就有点难了。