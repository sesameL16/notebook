1. Lombok提供注解方式来提高代码的简洁性,常用注解有:

    @Data

    @Setter @Getter

    @Synchronized

    @ToString

    @EqualsAndHashCode

    @Slf4j

    @Builder

2. 在 pom.xml 文件中添加相关依赖

    ~~~xml
    <lombok.version>1.16.20</lombok.version>
    
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <scope>provided</scope>
    </dependency>
    
    ~~~

    

3. 安装插件

    由于 Lombok 采取的注解形式的，在编译后，自动生成相应的方法，需要下载插件了支持它。 

    以 idea 为例：查找插件 lombok plugin 安装即可。

4. @Data 注解

    就可以有下面几个注解的功能： @ToString、@Getter、@Setter、@EqualsAndHashCode、@NoArgsConstructor

5. @Builder

    bulder 模式构建对象