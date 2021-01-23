1. 解压

    tar -zxvf nexus-3.6.0-02-unix.tar.gz -C /usr/local/

2. 启动nexus3

    cd /usr/local/nexus-3.6.0-02/bin/

    ./nexus run &

    

    稍等一会(首次启动会比较慢),当出现以下日志的时候表示启动成功!

    \-------------------------------------------------

    Started Sonatype Nexus OSS 3.6.0-02

    \-------------------------------------------------

3. 开启远程访问端口

    firewall-cmd --zone=public --add-port=8081/tcp --permanent

    firewall-cmd --reload

4. 访问

    nexus3默认端口是:8081

    nexus3默认账号是:admin

    nexus3默认密码是:admin123

    

5. 设置开机自启动(systemctl方式)

    创建一个服务

    vim /usr/lib/systemd/system/nexus.service

    填入相关内容

    ~~~
    [Unit]
    Description=nexus service
    [Service]
    Type=forking
    LimitNOFILE=65536 #警告处理
    ExecStart=/usr/local/nexus/nexus-3.7.1-02/bin/nexus start
    ExecReload=/usr/local/nexus/nexus-3.7.1-02/bin/nexus restart
    ExecStop=/usr/local/nexus/nexus-3.7.1-02/bin/nexus stop
    Restart=on-failure
    [Install]
    WantedBy=multi-user.target
    ~~~

    将服务加入开机启动

    systemctl enable nexus.service

    重新加载配置文件

    systemctl daemon-reload

    

6. 修改nexus3的运行用户为root

    vim nexus.rc

    run_as_user="root"

    

7. 修改nexus3启动时要使用的jdk版本

    vim nexus

    INSTALL4J_JAVA_HOME_OVERRIDE=/usr/local/java/jdk1.8.0_144

8. 修改nexus3默认端口(可选)

    cd /usr/local/nexus-3.6.0-02/etc/

    vim nexus-default.properties

    默认端口:8081

    application-port=8081

9. 修改nexus3数据以及相关日志的存储位置(可选)

    cd /usr/local/nexus-3.6.0-02/bin/

    vim nexus.vmoptions 

     

    -XX:LogFile=./sonatype-work/nexus3/log/jvm.log

    -Dkaraf.data=./sonatype-work/nexus3

    -Djava.io.tmpdir=./sonatype-work/nexus3/tmp

10. 配置本地maven的setting.xml文件

    ~~~~xml
    <localRepository>E:\repository</localRepository>
    
    <servers>
      <!--设置 Nexus 认证信息-->
    	<server>
    	      <id>nexus-public</id>
    	      <username>sesame</username>
    	      <password>bejwt531676</password>
    	    </server>
    	  	<server>
    	      <id>nexus-releases</id>
    	      <username>sesame</username>
    	      <password>bejwt531676</password>
    	    </server>
    		<server>
    	      <id>nexus-snapshots</id>
    	      <username>sesame</username>
    	      <password>bejwt531676</password>
    	</server>
      </servers>
    
    <mirrors>
      <mirror>
          <id>nexus-public</id>
          <mirrorOf>*</mirrorOf>
          <url>http://192.168.3.150:8081/repository/maven-public/</url>
        </mirror>
    <mirror>
          <id>nexus-releases</id>
          <mirrorOf>*</mirrorOf>
          <url>http://192.168.3.150:8081/repository/maven-releases/</url>
        </mirror>
    <mirror>
          <id>nexus-snapshots</id>
          <mirrorOf>*</mirrorOf>
          <url>http://192.168.3.150:8081/repository/maven-snapshots/</url>
        </mirror>
      </mirrors>
    
    <profiles>
    	<profile>
    		<id>jdk-1.8</id>
    　　　　<activation>
    	<activeByDefault>true</activeByDefault>
    　　　　	<jdk>1.8</jdk>
    	</activation>
    	</profile>
    	<profile>
    		<id>nexusRepository</id>  
    		<!-- jar包仓库配置 -->  
    		<repositories>  
    			<repository>  
    				<id>nexus-public</id>  
    				<url>http://192.168.3.150:8081/repository/maven-public/</url>  
    				<releases>  
    					<enabled>true</enabled>  
    				</releases>  
    				<snapshots>  
    					<enabled>true</enabled>  
    				</snapshots>  
    			</repository>  
    			<repository>  
    				<id>nexus-releases</id>  
    				<url>http://192.168.3.150:8081/repository/maven-releases/</url>  
    				<releases>  
    					<enabled>true</enabled>  
    				</releases>  
    				<snapshots>  
    					<enabled>true</enabled>  
    				</snapshots>  
    			</repository> 
    			<repository>  
    				<id>nexus-snapshots</id>   
    				<url>http://192.168.3.150:8081/repository/maven-snapshots/</url>  
    				<releases>  
    					<enabled>true</enabled>  
    				</releases>  
    				<snapshots>  
    					<enabled>true</enabled>  
    				</snapshots>  
    			</repository>  
    		</repositories>  
    		
    		 <pluginRepositories>  
                <pluginRepository>  
                    <id>nexus-public</id>  
                    <url>http://192.168.3.150:8081/repository/maven-public/</url>  
    	<releases>  
    		<enabled>true</enabled>  
    	</releases>  
    	<snapshots>  
    		<enabled>true</enabled>  
    	  </snapshots>  
               </pluginRepository>  
    	</pluginRepositories>  
    
    	</profile>
    
      </profiles>
    
      <activeProfiles>
        <activeProfile>jdk-1.8</activeProfile>
        <activeProfile>nexusRepository</activeProfile>
      </activeProfiles>
    
    ~~~~

11. 项目pom文件修改

    ~~~~xml
    <distributionManagement>
    	<snapshotRepository>
    		<id>nexus-snapshots</id>
    		<url>http://192.168.3.150:8081/repository/maven-snapshots/</url>
    	</snapshotRepository>
    	<repository>
            <!--这个id和settings.xml中的servier标签下的对应-->
    		<id>nexus-releases</id>
            <!--nexus的仓储地址-->
    		<url>http://192.168.3.150:8081/repository/maven-releases/</url>
    	</repository>
    </distributionManagement>
    ~~~~

    