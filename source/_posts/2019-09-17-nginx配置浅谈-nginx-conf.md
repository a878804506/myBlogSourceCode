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
4. nginx.conf中的负载均衡的配置详解；
5. nginx的访问权限配置(基于ip和账号密码的两种方式)；
6. https的访问配置；
7. 新增stream模块以支持tcp代理(2019.12.03)；

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

### 六、nginx.conf中的负载均衡的配置详解
负载均衡一般配置在upstream块中，负载均衡的几种方式：
1. 轮询（默认）
每个请求会按时间顺序逐一分配到不同的后端服务器。在轮询中，如果服务器down掉了，会自动剔除该服务器。<font color=red>缺省配置就是轮询策略</font>。此策略适合服务器配置相当，无状态且短平快的服务使用。
2. weight（权重）
在轮询策略的基础上指定轮询的几率。权重越高分配到需要处理的请求越多。此策略可以与ip_hash和least_conn结合使用。此策略比较适合服务器的硬件配置差别比较大的情况。
eg：
````
# 动态负载均衡服务器组
upstream dynamic_balance {
	server localhost:8080 weight=2;
	server localhost:8081 weight=5;
	server localhost:8082 weight=3;
}
````
3. ip_hash（根据ip分配）
指定负载均衡器按照基于客户端IP的分配方式，这个方法确保了相同的客户端的请求一直发送到相同的服务器，以保证session会话。这样每个访客都固定访问一个后端服务器，可以解决session不能跨服务器的问题。在Nginx版本1.3.1之前，不能在ip_hash中使用权重（weight）。ip_hash不能与backup同时使用。此策略适合有状态服务，比如session。当有服务器需要剔除，必须手动down掉。
eg:
````
upstream dynamic_balance {
	ip_hash;    # 保证每个访客固定访问一个后端服务器
	server localhost:8080 weight=2;
	server localhost:8081;
	server localhost:8082;
}
````
4. least_conn（最少连接）
把请求转发给连接数较少的后端服务器。轮询算法是把请求平均的转发给各个后端，使它们的负载大致相同；但是，有些请求占用的时间很长，会导致其所在的后端负载较高。这种情况下，least_conn这种方式就可以达到更好的负载均衡效果。此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。
eg:
````
upstream dynamic_balance {
	least_conn;    # 把请求转发给连接数较少的后端服务器
	server localhost:8080 weight=2;
	server localhost:8081;
	server localhost:8082;
}
````
5. fair（响应时间 - 第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
eg:
````
upstream resinserver{
	server server1;
	server server2;
	fair;
}
````
6. url_hash（根据url分配 - 第三方）
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
eg:
````
upstream resinserver{
	server squid1:3128;
	server squid2:3128;
	hash $request_uri;
	hash_method crc32;
}
````
参数说明：
|参数名称|参数含义|
|:---:|:---:|
|fail_timeout|与max_fails结合使用。|
|max_fails|设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了。|
|fail_time|服务器会被认为停机的时间长度，默认为10s。|
|backup|标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里。|
|down|标记服务器永久停机了。|

### 七、nginx的访问权限配置

1. 基于ip的配置
介绍： 访问权限可以通过配置基于ip的访问控制，达到让某些ip能够访问，限制哪些ip不能访问的效果<br/>
允许访问的配置方法
配置语法：allow address | CIDR | unix | all;
默认配置：没有配置
配置路径：http、server、location、limit_except下；<br/>
不允许访问的配置方法
配置语法：deny address | CIDR | unix | all;
默认配置：没有配置
配置路径：http、server、location、limit_except下；

例子：
````
location {
	# 拒绝此IP访问
	deny 192.168.1.1;
	# 允许该网段访问
	allow 192.168.1.0/24;
	# 拒绝所有
	deny all;
}
````
<font color=red>从上到下开始匹配，匹配到了则停止。</font>
2. 基于账号密码的配置
①. 安装软件httpd
````bash
yum -y install httpd
````
②. 创建密码文件
````bash
# /usr/local/nginx1.16.1/mypasswd 生成密码文件的全路径
# test 用户名
# 123456 密码
htpasswd -c -b /usr/local/nginx1.16.1/mypasswd  test  123456
````
③. 配置nginx.conf
需要配置的参数：**<font color=purple>auth_basic、auth_basic_user_file</font>**
参数说明：
|参数名|配置语法|默认配置|可配置的区域块|
|:---:|:---:|:---:|:---:|
|auth_basic|string or off|off|http、server、location|
|auth_basic_user_file|密码路径|/|http、server、location|

账号密码配置示例：
````
server {
    listen       80;
    server_name  staticfile.erdongchen.top;
    charset utf-8;
    
    # 目录
    root /usr/local/staticFiles;
    #开启目录文件列表
    autoindex on;
    # 显示出文件的确切大小，单位是bytes
    autoindex_exact_size on;
    # 显示的文件时间为文件的服务器时间
    autoindex_localtime on;
    
    location /web/excelAddr/ {
        # 这里是验证时的提示信息
        auth_basic "Please input password";
        # 密码文件所在的位置
        auth_basic_user_file /usr/local/mypasswd;
    }

    location /files_bak/ {
        deny all; # 不允许访问
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
````
④. 最终效果图
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20190917/authority_file.png"  align=left/>

#### 附：htpasswd命令及其参数含义说明
命令：
````bash
# 创建密码文件并且添加用户，
htpasswd -c  -b  文件名 用户名   密码
# 添加用户不创建文件
htpasswd  -b   用户名   密码
# 删除用户和密码
htpasswd -D  文件名   用户名
# 修改密码 ：就是删除用户然后创建用户
htpasswd -D  文件名   用户名
htpasswd  -b   用户名   密码
````
参数含义：

|参数名|配置语法|
|:---:|:---:|
| -c |创建加密文件|
| -n |不更新加密文件，只将加密的用户密码显示在屏幕上|
| -m |默认采用MD5算法进行加密|
| -d |采用CRYPT算法对密码进行加密|
| -p |不对密码进行加密 ，即明文密码|
| -s |采用SHA算法对密码进行加密|
| -b |在命令行中一并输入用户名和密码而不是根据提示输入密码|
| -D |删除指定的用户|

### 八、https的访问配置
1. 查看是否有ssl模块
````bash
nginx -V
````
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20190917/qianzhi.png"  align=left/>
2. 如果没有上面这个就需要添加此模块：
nginx解压目录执行：
````bash
./configure --with-http_ssl_module
make
````
此时，在objs下回生成新的nginx文件，覆盖到安装目录的sbin目录下面
 **<font color=red>防坑：在执行完make命令后，如果不执行make install则是添加模块，就需要把新的nginx文件覆盖到安装目录的sbin目录下！！如果接着执行make install，则表示重新安装nginx！</font>**

3. SSL证书申请，并放置到服务器上
证书申请可以在阿里云上申请或者腾讯云上也可以申请，有免费的，实在不行还可以自己创建证书；
阿里云申请地址：[**<font color=purple>申请</font>**](https://www.aliyun.com/product/cas "阿里云")

4. nginx.conf配置中配置SSL实现https访问
配置示例：
````
# http/https 静态文件访问地址
server {
    listen       80;
    listen       443 ssl;
    server_name  staticfile.erdongchen.top;
    charset utf-8;
    
    # ssl证书地址
    ssl_certificate     /usr/local/staticfile.pem;  # pem文件的路径
    ssl_certificate_key  /usr/local/staticfile.key; # key文件的路径
    
    # ssl验证相关配置
    #缓存有效期
    ssl_session_timeout  5m;
    #加密算法
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #安全链接可选的加密协议
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #使用服务器端的首选算法
    ssl_prefer_server_ciphers on;
    
    # 目录
    root /usr/local/staticFiles;
    #开启目录文件列表
    autoindex on;
    # 显示出文件的确切大小，单位是bytes
    autoindex_exact_size on;
    # 显示的文件时间为文件的服务器时间
    autoindex_localtime on;
    
    location /web/excelAddr/ {
        # 这里是验证时的提示信息
        auth_basic "Please input password";
        # 密码文件所在的位置
        auth_basic_user_file /usr/local/mypassword;
    }

    location /files_bak/ {
        deny all; # 不允许访问
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
````

5. 访问https
[**<font color=purple>https://staticfile.erdongchen.top/</font>**](https://staticfile.erdongchen.top/ "我的博客")
此配置也可以使用http访问
[**<font color=purple>http://staticfile.erdongchen.top/</font>**](http://staticfile.erdongchen.top/ "我的博客")

### 九、新增stream模块以支持tcp代理(2019.12.03)
由于个人项目上需要用到tcp代理的需求，这里记录一下通过配置nginx来实现tcp的代理转发：
1. 检查自己已经安装的nginx有没有支持stream模块：
````bash
nginx -V
````
 如果在configure arguments 栏没有`--with-stream`模块，则表明nginx没有此模块；
 
2. 上述命令执行的结果里面有configure arguments，把里面参数复制一份，停掉nginx，然后在nginx安装目录执行命令 
````bash
# 备注：--with-http_ssl_module是之前带的模块（有多少模块带多少），再在后面追加--with-stream
./configure --with-http_ssl_module --with-stream
````

3. 编译完成之后，在执行`make`命令；

4. 如果没有报错，会在nginx安装目录下的objs目录生成nginx文件，把该文件覆盖到sbin目录

5. 配置stream模块，例如：
 nginx.conf 主配置文件：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20190917/9.1.png"  align=left/>

 引入的外部文件配置：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20190917/9.2.png"  align=left/>

6. 文件配置完毕之后，先检查配置是否正确：`nginx -t`,在启动nginx；这时便可以使用ngixn所在的服务器ip+端口（我这里配置的是80端口），nginx会自动将请求转发到192.168.1.184的2181端口上；步骤到这里就已经配置完毕了，实现了ip+端口的方式代理tcp请求，此配置也可以实现像mysql或者redis等等的tcp转发；






