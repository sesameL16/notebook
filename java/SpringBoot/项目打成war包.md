1. maven项目，修改pom包

~~~xml
将
<packaging>jar</packaging>
改为
<packaging>war</packaging>
~~~



2. 打包时排除tomcat

   在这里将scope属性设置为provided，这样在最终形成的WAR中不会包含这个JAR包，因为Tomcat或Jetty等服务器在运行时将会提供相关的API类。

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
~~~



3. 注册启动类

   创建**SesameApplication** .java，继承SpringBootServletInitializer ，覆盖configure()，把启动类Application注册进去。外部web应用服务器构建Web Application Context的时候，会把启动类添加进去。

~~~java
@SpringBootApplication
Publicclass SesameApplication extends SpringBootServletInitializer{
    @Override
    Protected SpringApplicationBuilder configure(
        SpringApplicationBuilder builder){
   		return builder.sources(SesameApplication.class);
    }
}

~~~



 