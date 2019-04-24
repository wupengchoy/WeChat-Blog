[TOC]

## 命令

- mac使用ssh远程连接到服务器：运行sudo su(可省略)，ssh -p 22 root@112.74.60.217，输入密码
- 本地上传文件到服务器：scp /path/file（这部分为本地的路径） user（远端目标用户名）@host（远端目标IP）:/pathorfile（文件存储路径）
- 下载服务器文件到本地：scp user（远端用户名）@host（远端IP）:/path/file（下载文件在远端的路径） localpathorfile（本地文件存放路径）例如：scp -r Documents/MyBlog root@1:/root/blog    注解：-r可传文件夹
- 查看服务器上文件或目录：ssh user@host command ls "/path/*.tgz"
- 重启防火墙：systemctl restart firewalld
- 查看端口是否启用：firewall-cmd --query-port=80/tcp
- 添加端口：firewall-cmd --add-port=80/tcp



### 

## net123

- nat123客户端Linux版安装启动

apt install mono-complete
cd /mnt
wget http://www.nat123.com/down/nat123linux.tar.gz 登陆网站下载安装包，如没有帐号，可以进入nat123网站进行注册。
tar -zxvf nat123linux.tar.gz
mono nat123linux.sh 运行客户端，并按提示依次输入自己的帐号和密码
mono nat123linux.sh service & ——后台服务方式启动，自动读取上次成功登录帐号
3，开机自动登录启动
把启动程序的命令添加到/etc/rc.local文件中，此文件内容如下，

```text
# !/bin/sh -e
# rc.local
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
# In order to enable or disable this script just change the execution
# bits.
# By default this script does nothing.
cd /data/tools/net123
screen -S net123 确保本地可执行screen，通过它单独调用当后台进程运行
mono nat123linux.sh service &
exit 0
```



