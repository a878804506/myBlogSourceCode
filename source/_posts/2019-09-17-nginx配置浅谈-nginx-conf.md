---
title: nginx配置浅谈-nginx.conf
date: 2019-09-17 18:25:40
categories: nginx
tags: 
  - 环境配置
  - linux
  - 技术
---
### 一、本篇教程侧重点导读
1. nginx.conf配置总体说明以及示例；
2. nginx安装；
3. 源码编译构建nginx时配置参数详细说明；
4. nginx模块的添加和卸载；
5. nginx.conf中的负载均衡的配置详解；
6. nginx的访问权限配置(基于ip和账号密码的两种方式)；
7. https的访问配置；
8. nginx文件上传下载功能；

### 二、本篇教程用的软件、技术和说明
1. 使用到linux系统：CentOS 7.2；
2. 使用到的nginx版本：1.15.2

### 三、nginx.conf配置总体说明以及示例
1. **<font color=green>全局块</font>**：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2. **<font color=green>events块</font>**：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. **<font color=green>http块</font>**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. **<font color=green>server块</font>**：配置虚拟主机的相关参数，一个http中可以有多个server。
5. **<font color=green>location块</font>**：配置请求的路由，以及各种页面的处理情况。
6. **<font color=green>upstream块</font>**：用来配置后台服务器负载均衡用的。
[**<font color=purple>nginx.conf配置文件示例下载</font>**](http://staticfile.erdongchen.top/download/config.example.conf?n=nginx "点击下载nginx.conf")
<img style="width:85%;height:85%" src="http://staticfile.erdongchen.top/blog/blogPicture/20190917/config.example.png"  align=left/>

### 四、nginx安装
1. 安装编译环境
````bash
yum -y install gcc
yum -y install gcc++
yum -y install gcc-c++
yum -y install wget
yum -y install pcre-devel
yum -y install zlib zlib-devel
# https配置需要
yum -y install openssl openssl-devel
````
2. 下载nginx安装包
建议下载稳定版本（Stable version）：[nginx官网下载](http://nginx.org/en/download.html "点击下载"),然后把包上传到linux上
<font color=red>或者</font>在linux使用如下命令下载nginx-1.16.1安装包：
````bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
````
3. 解压nginx安装包
````bash
tar -xzvf nginx-1.16.1.tar.gz -C /usr/local/
````
<font color=green>参数 -C 解压到指定路径下</font>
4. 源码编译安装nginx
````bash
# 进入nginx目录
cd /usr/local/nginx-1.16.1
# 小白推荐执行命令
bash configure
# 老鸟推荐执行脚本(带https配置、可自定义配置各类参数)
bash configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --error-log-path=/usr/local/nginx/logs/error.log --http-log-path=/usr/local/nginx/logs/access.log --pid-path=/usr/local/nginx/logs/nginx.pid --lock-path=/usr/local/nginx/lock/nginx.lock --user=root --group=root --with-http_ssl_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module  --with-debug --http-client-body-temp-path=/usr/local/nginx/temp --with-stream
# 执行命令
make
# 执行make install命令
make install
````
5. 配置环境变量
````bash
# 编辑  /etc/profile
vim /etc/profile
# 在末尾追加
export NGINX_HOME=/usr/local/nginx
export PATH=$PATH:$NGINX_HOME/sbin
# 重新编译 /etc/profile 文件
source /etc/profile
````
<font color=red>防坑：/usr/local/nginx 是你安装nginx的目录</font>
6. 安装验证,利用nginx -v 来查看安装是否正确，以及相关的nginx信息。
````bash
nginx -v
````
7. 根据具体项目所需配置nginx.conf文件
8. nginx相关命令
````bash
# Nginx检测
nginx -t
# 启动
nginx
# 平滑重启
nginx -s reload
# 快速停止（立即停止服务,这种方法比较强硬，无论进程是否在工作，都直接停止进程。）
nginx -s stop
# 正常停止（从容停止服务,这种方法较stop相比就比较温和一些了，需要进程完成当前工作后再停止。）
nginx -s quit
````

### 五、源码编译构建nginx时配置参数详细说明
1. 在解压的目录有个文件configure，运行./configure –-help 可以看到大量的参数显示。
2. configure的参数分为四大类：路径相关、编译相关、依赖软件相关、模块相关
[**<font color=red>configure参数配置说明书</font>**](https://myblog.erdongchen.top/2019/09/20/nginx配置浅谈-configure的参数配置说明/ "查看详情")

### 五、
nginx模块的添加和卸载；




