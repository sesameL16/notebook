java环境下修改配置文件

vim $JAVA_HOME/jre/lib/security/java.security 

securerandom.source=file:/dev/random

改为 

securerandom.source=file:/dev/urandom