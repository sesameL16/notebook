1. jar包工程部署

   下载java镜像

   docker pull sesamel16/java:8u111

   使用docker运行jar包

   docker run -d -p 8081:8081 -v /city/smart-core.jar:/smart-core.jar --name smart-core sesamel16/java:v1 java -jar /smart-core.jar

 

2. war包工程部署

   下载tomcat镜像

   docker pull sesamel16/tomcat:8.5