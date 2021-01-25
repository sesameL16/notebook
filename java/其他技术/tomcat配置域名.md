1. 首先我们打开tomcat的安装路径找到webapps文件夹，将其中的ROOT文件夹删除
2. 然后去conf文件夹当中找到server.xml

![tomcat](../../图片/tomcat配置域名/tomcat-1611491131802.png)

![tomcat1](../../图片/tomcat配置域名/tomcat1-1611491136390.png)

![tomcat2](../../图片/tomcat配置域名/tomcat2.png)



我们需要修改这几个地方

![tomcat3](../../图片/tomcat配置域名/tomcat3.jpg)

![tomcat4](../../图片/tomcat配置域名/tomcat4.jpg)

由上往下其中第一处port为：修改8080端口为80端口

第二处defaultHost=""与第三处name=""，填写的是你要修改的域名

第四处为docBase为你的项目名称。

附修改后的server.xml源码

![tomcat5](../../图片/tomcat配置域名/tomcat5.png)