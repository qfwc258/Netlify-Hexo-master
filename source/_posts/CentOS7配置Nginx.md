title: CentOS7配置Nginx 
date: 2017/12/20
comments: true 
tags:
 - CentOS
 - Nginx
 - Tomcat
Java categories: CentOS

---------
## 安装准备

由于nginx的一些模块依赖一些lib库,所以在安装nginx前,必须先安装这些lib库.

这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel,执行如下命令进行安装

![](http://oih7sazbd.bkt.clouddn.com/nginx1.png)

![](http://oih7sazbd.bkt.clouddn.com/2.png)

<!-- more -->

![](http://oih7sazbd.bkt.clouddn.com/3.png)

![](http://oih7sazbd.bkt.clouddn.com/4.png)

## 安装Nginx

安装之前,首先检查是否已经安装有nginx

```
# find -name nginx
```

如果已安装了nginx,则先卸载

```
# yum remove nginx
```

进入/usr/local目录

```
# cd /usr/local
```

从官网下载最新版本的nginx

```
# wget http://nginx.org/download/nginx-1.9.9.tar.gz
```

下载完成后解压nginx压缩包

```
# tar -zxvf nginx-1.9.9.tar.gz
```

解压完成后会产生一个名为nginx-1.9.9的目录,这时进入这个目录

```
# cd nginx-1.9.9
```

进入后就开始进行安装,使用--prefix参数指定nginx安装的目录,make、make install进行安装

```
# ./configure
# make
# make install
```

默认安装到了/usr/local/nginx里. 如果没有出错,则可以通过whereis查看nginx的安装目录

```
# whereis nginx
```

这个时候,可以继续下一步,设置安装服务实现自启动,首先建立服务文件

```
# vim /lib/systemd/system/nginx.service
```

在文件中输入以下内容

```
[Unit]
Description=nginx
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

为服务文件设置权限

```
# chmod 754 /lib/systemd/system/nginx.service
```

设置开机自启动

```
# systemctl start nginx.service
# systemctl enable nginx.service
```

## Nginx映射到Tomcat

找到nginx的配置文件并打开

```
# cd /usr/local/nginx/conf
# vi nginx.conf
```

添加如下配置

```
server {    
    listen          80;//监听的服务器端口    
    server_name     www.xx.com;//要监听的域名     
    location / {     
        proxy_pass http://localhost:8080;//将域名解析到的Tomcat路径
        proxy_set_header   Host    $host;     
        proxy_set_header   X-Real-IP   $remote_addr;     
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;     
        
    }  
  }
```

对nginx进行如上配置后,重启nginx就可以看到效果了 :)
