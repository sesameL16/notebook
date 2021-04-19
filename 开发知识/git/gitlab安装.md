#### 安装和配置必要的依赖项

~~~
yum install -y curl policycoreutils-python openssh-server
systemctl enable sshd
systemctl start sshd

firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
systemctl reload firewalld


sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
~~~

#### 添加GitLab软件包存储库并安装软件包

~~~
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
~~~

#### 安装

~~~
//使用yum安装gitlab
yum install -y gitlab-ee
//可以看下gitlab-ee包的内容，看到gitlab安装在/opt/gitlab目录下
rpm -ql gitlab-ee | less
~~~

#### 修改配置

~~~
gitlab的配置文件在/etc/gitlab/目录下，主要配置文件为gitlab.rb
修改为域名或者ip地址
external_url 'http://gitlab.ai-he.me'
# 系统端口冲突，我把端口改为了82，默认为80
nginx['listen_port'] = 82
~~~

#### 运行

~~~
//重新配置gitlab
gitlab-ctl reconfigure
//重启gitlab
gitlab-ctl restart 
~~~

