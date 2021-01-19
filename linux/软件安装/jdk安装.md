1. 解压缩命令：

    tar  -zxvf jdk-8u131-linux-x64.tar.gz  -C /usr/local

2. 配置java环境变量

    vim /etc/profile

    \# java

    export JAVA_HOME=/usr/local/jdk1.8.0_131

    export JRE_HOME=/usr/local/jdk1.8.0_131/jre

    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib

    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

3. 环境变量起作用

    source /etc/profile

4. 测试环境变量

    java -version   javac