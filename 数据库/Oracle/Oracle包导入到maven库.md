1. 安装 JDK 并正确设置了 Java 环境变量。

2. Maven 的下载地址为：http://maven.apache.org/download.cgi  将下载下来的文 件解压到指定的目录中

3. 配置maven环境变量。

    ![pache Maven 3. 5. O (ff8f5e7444045639af65f6095c62210b5713f426,  aven home: D: \apache-maven-3. 5. O\bin\..  ava version: 1. 8. 0 101, vendor: Oracle Corporation  ava home: D: \jre  efault locale: Zh CN,  S name:  windows 10  platform encoding: GBK  version: "10. 0  arch:  amd64'  family:  2017-04-04T03  windows ](../../图片/Untitled/clip_image001.png)

4. mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.3.0 -Dpackaging=jar -Dfile=jar包路径

    ![maven](../../图片/Untitled/maven.png)

~~~xml
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.3.0</version>
</dependency>
~~~

