---
title: nginx配置浅谈-configure的参数配置说明
date: 2019-09-20 13:59:03
categories: nginx
tags: 
  - 说明书
  - linux
  - 技术
---
### 参数说明
|序号|参数格式|参数含义|
|:---:|:---:|:---------:|
|1|--prefix=path|定义安装目录|
|2|--sbin-path=path|设置nginx可执行文件的路径|
|3|--modules-path=path|定义将安装nginx动态模块的目录|
|4|--conf-path=path|设置nginx.conf配置文件路径名|
|5|--error-log-path=path|设置主要错误，警告和诊断文件的路径名|
|6|--pid-path=path|设置nginx.pid将存储主进程的进程ID 的文件的路径名|
|7|--lock-path=path|为锁定文件的名称设置前缀|
|8|--user=name|设置非特权用户的名称，其凭据将由工作进程使用|
|9|--group=name|设置其凭据将由工作进程使用的组的名称|
|10|--build=name|设置编译名|
|11|-builddir=path|置编译目录|
|12|--with-select_module、--without-select_module|启用或禁用select事件驱动模型|
|13|--with-poll_module、--without-poll_module|启用或禁用poll事件驱动模型|
|14|--with-threads|启用线程池功能|
|15|--with-file-aio|我们就可以启用线程池功能，让请求排队等待处理，并且可以充分利用CPU提高处理效率。开启线程池需要AIO的支持。 启用异步文件IO（AIO）支持。一般用于大文件传输的场景下。|
|16|--with-http_ssl_module|将HTTPS协议支持添加到HTTP服务器。默认情况下不构建此模块。需要OpenSSL库来构建和运行此模块。|
|17|--with-http_v2_module|启用HTTP_V2模块，新的HTTP协议，相比HTTP1更优更快。默认情况下不构建此模块。|
|18|--with-http_realip_module|启用HTTP_Realip模块，用于修改客户端请求头中客户端IP地址值，一般用于反向代理中，将真实的客户端IP传送给后端的应用服务器。默认情况下不构建此模块。|
|19|--with-http_addition_module|启用HTTP_Addition模块，用于在响应之前和之后添加文本。默认情况下不构建此模块。|
|20|--with-http_xslt_module |启用HTTP_Xslt模块，这个模块是一个过滤器，它可以通过XSLT模板转换成XML响应。需要ibxml2和libxslt库的支持。默认情况下不构建此模块。|
|21|--with-http_xslt_module=dynamic|启用HTTP_Xslt动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|22|--with-http_image_filter_module|启用HTTP_Image_Filter模块，这个模块是一个集成图片处理器，我们可以使用它转换JPEG、GIF、PNG和WEBP格式的图像，验证这些格式图像的有效型（是不是此格式的图像），输出JSON格式的图像信息，旋转图像，按比例缩放图像，剪切图片等。默认情况下不构建此模块。|
|23|--with-http_image_filter_module=dynamic|启用HTTP_Image_Filter动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|24|--with-http_geoip_module|启用HTTP_Geoip模块，这个模块用于处理不同地区的访问，当来自某一个区域的访问时将其重定向到对应的服务或者项目上，需要MaxMind GeoIP库的支持。默认情况下不构建此模块。|
|25|--with-http_geoip_module=dynamic|启用HTTP_Geoip动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|26|--with-http_sub_module|启用HTTP_Sub模块，这个模块是一个过滤器，用于修改响应的内容，可以将一个指定的字符串替换成另一个字符串。默认情况下不构建此模块。|
|27|--with-http_dav_module|启用HTTP_DAV模块，用于通过WEBDAV协议提供WEB的文件管理功能，类似于一个WEB的文件管理器，可以对服务器的文件进行管理。默认情况下不构建此模块。|
|28|--with-http_flv_module|启用HTTP_FLV模块，用于为Flash Video（FLV）文件提供伪流视频服务端支持，开启它则允许在网页上播放FLV格式的视频。默认情况下不构建此模块。|
|29|--with-http_mp4_module|启用HTTP_MP4模块，用于为MP4格式的视频文件提供伪流视频服务端支持，开启它则允许在网页上播放MP4格式的视频。默认情况下不构建此模块。|
|30|--with-http_gunzip_module|启用HTTP_Gunzip模块，用于为不支持"gzip"编码方式的客户端解压响应。|
|31|--with-http_gzip_static_module|启用HTTP_Gzip_Static模块，用于将静态内容压缩成".gz"为文件扩展名的预压缩文件，并缓存在本地，在响应时会将此文件发送以替代普通文件。|
|32|--with-http_auth_request_module|启用HTTP_Auth_Request模块，此模块是一个请求验证模块，可以使用外部服务器或服务对网站的每个请求进行身份验证。当用户访问时，Nginx会向用于验证请求的外部服务器发出验证请求，若返回的状态码为200，则通过允许访问，若返回401或403，则访问会被拒绝。默认情况下不构建此模块。|
|33|--with-http_random_index_module|启用HTTP_Random_Index模块，随机主页模块，当用户访问时，随机响应一个主页，而并非由"index"指令定义的一个主页，而是从主页池中随机选中一个主页面返回。默认情况下不构建此模块。|
|34|--with-http_secure_link_module|启用HTTP_Secure_Link模块，防盗链模块，用于检查请求链接的权限以及是否过期，多用于文件下载防盗链。默认情况下不构建此模块。|
|35|--with-http_degradation_module|启用HTTP_Degradation模块，用于当主机剩余内存较低时，用户请求访问，Nginx会对某些"location"的请求返回204或444的响应码。默认情况下不构建此模块。|
|36|--with-http_slice_module|启用HTTP_Slice模块，此模块是一个过滤器，用于将一个大的完整的文件分割成多个小块文件，分段传送给用户，一般用于大文件传输的场景下，使用它可以让用户快速的得到响应。默认情况下不构建此模块。|
|37|--with-http_stub_status_module|启用HTTP_Stub_Status模块，状态信息统计模块，用于返回一个Nginx状态信息统计信息页面，管理员访问这个页面可以获取Nginx的请求处理、当前连接、等待连接等统计信息，一般用于监控Nginx的运行状态。默认情况下不构建此模块。|
|38|--without-http_charset_module|禁用HTTP_Charset模块，此模块用于将指定的字符集添加到"Content-Type"响应头字段中。此外此模块还可以将数据从一个字符集转换为另一个字符集，此模块用于字符集设置。不建议禁用。|
|39|--without-http_gzip_module|禁用HTTP_Gzip模块，此模块用于HTTP响应内容传输压缩，可以将响应内存在传输时将其压缩成Gzip编码格式的响应传送给客户端，使用Gzip编码格式响应内容体积会变小，会提高传输效率。不建议禁用。|
|40|--without-http_ssi_module|禁用HTTP_SSI模块，此模块是一个过滤器，用于处理通过它响应中的SSI（Server Side Includes）命令。目前支持的SSI命令列表并不完整，SSI指令是一种可以嵌入WEB页面的一种语法指令。|
|41|--without-http_userid_module|禁用HTTP_Userid模块，此模块用于识别客户端的Cookie。可以使用嵌入变量"$uid_got"和"$uid_set"记录已接受和设置的Cookie。|
|42|--without-http_access_module|禁用HTTP_Access模块，此模块用于限制对某些客户端地址的访问，Allow or Deny。不建议禁用。|
|43|--without-http_auth_basic_module|禁用HTTP_Auth_Basic模块，该模块用于HTTP基本身份验证，使用用户名和密码来限制对资源的访问。|
|44|--without-http_mirror_module|禁用HTTP_Mirror模块，该模块用于将正式环境的流量拷贝到镜像（测试）环境下，一般用于测试环境引入真实环境的流量实现对测试环境的压力测试。|
|45|--without-http_autoindex_module|禁用HTTP_Autoindex模块，该模块用于在处理以斜杠字符（'/'）结尾的请求，并在找不到索引文件的情况下生成目录列表。|
|46|--without-http_geo_module|禁用HTTP_Geo模块，该模块用于从指定变量中获取客户端的IP地址，并将其嵌入到另外一个变量中。默认情况下从"$remote_addr"变量中取得客户端的IP地址。我们可以通过它结合"HTTP_Upstream"实现对来源客户端的负载均衡，当来自不同的客户端请求时，将其负载均衡给后端的不同的服务器处理；还可以使用它结合"HTTP_Map"+"HTTP_Limit_Conn"模块实现对来源客户端的限速功能。|
|47|--without-http_map_module|禁用HTTP_Map模块，该模块用于创建一个变量的映射表，结果变量可以是一个字符串也可以是另外一个变量。|
|48|--without-http_split_clients_module|禁用HTTP_Splic_Clients模块，该模块用于创建适用于A/B测试的变量，AB测试也称之为拆分测试，也就是将一个项目的两个不同版本发布，看用户更喜欢用于那个版本，若版本A受欢迎则发布版本A。|
|49|--without-http_referer_module|禁用HTTP_Referer模块，该模块用于防盗链，用于阻止对请求头部"referer"字段具有无效值的请求访问，可以设置一个白名单，非白名单的无效来源网址的连接则会拒绝请求，使用此模块我们还需考虑到，即使对于有效的请求，常规浏览器也可能不发送"referer"字段。不建议禁用。|
|50|--without-http_rewrite_module|禁用HTTP_Rewerte模块，该模块用于地址重写，用于将来源请求地址重定向到指定的地址上，可以保护真实的地址，增加安全性，该模块需要PCRE库的支持。不建议禁用。|
|51|--without-http_proxy_module|禁用HTTP_Proxy模块，该模块用于将请求代理传递到另外一台WEB服务器去处理，Nginx的核心模块。不建议禁用。|
|52|--without-http_fastcgi_module|禁用HTTP_FastCGI模块，该模块用于将请求代理传递到另外一台FastCGI服务器去处理，一般用于反代PHP。不建议禁用。|
|53|--without-http_uwsgi_module|禁用HTTP_UwSGI模块，该模块用于将请求代理传递给另外一台UwSGI服务器去处理。|
|54|--without-http_scgi_module|禁用HTTP_SCGI模块，该模块用于将请求代理传递给另外一台SCGI服务器去处理。|
|55|--without-http_grpc_module|禁用HTTP_Grpc模块，该模块用于将请求代理传递给另外一台Grpc服务器去处理。|
|56|--without-http_memcached_module|禁用HTTP_Memcached模块，该模块用于Nginx从Memcached服务器获取响应内容。|
|57|--without-http_limit_conn_module|禁用HTTP_Limit_Conn模块，该模块用于限制并发连接数量以及下载带宽限制。|
|58|--without-http_limit_req_module|禁用HTTP_Limit_Req模块，该模块用于限制请求数量，可以限制请求的频率。|
|59|--without-http_empty_gif_module|禁用HTTP_Empty_Gif模块，该模块会在内容中常驻的一个1X1的透明空白的GIF图像，当用户请求时，返回该图像，一般用于测试。|
|60|--without-http_browser_module|禁用HTTP_Browser模块，该模块用于创建变量，变量的值取决于请求头中"user-agent"的值，一般用于区别新式或者旧式浏览器，若新式浏览器则将请求重定向到新式的WEB页面中，呈现新页面，若为旧式浏览器则将返回旧式的WEB页面。|
|61|--without-http_upstream_hash_module|禁用HTTP_Upstream_Hash模块，该模块提供了由"Upstream"指令定义的一组服务器的负载均衡方法"Hash"，该方法基于散列键值（hash），它会将客户端+服务端的映射关系存放到一个散列键值表中，当客户端第二次请求时则会匹配关系将请求转发至后端的同一台服务器上，实现会话保持功能。该模块提供指令"hash",在会话保持中，我们唯一能标识客户端的标志就是SessionID，所以我们可以使用指令"hash $cookie_jsession"实现会话保持功能。不建议禁用。|
|62|--without-http_upstream_ip_hash_module|禁用HTTP_Upstream_IP_Hash模块，该模块提供了由"Upstream"指令定义的一组服务器的负载均衡方法"ip_hash"，该方法也用于会话保持，不过它是基于客户端IP的Hash方法，由于用户可能是ADSL接入的网络，所以客户端可能受动态IP影响会发生变化，所以一般不建议采用这种方法。|
|63|--without-http_upstream_least_conn_module|禁用HTTP_Upstream_Least_Conn模块，该模块提供了由"Upstream"指令定义的一组服务器的负载均衡方法"least_conn"，该方法用于将请求传递到具有最少活动连接、权重较高（性能最好）的后端服务器上去处理。|
|64|--without-http_upstream_keepalive_module|禁用HTTP_Upstream_Keepalive模块，该模块可以为由"Upstream"指令定义的一组服务器提供保持长连接的功能，使用它则会为每个Worker工作进程与后端服务器保持空闲的长连接，连接数由"keepalive"指令指定，当空闲的长连接数量超过指定值时，将关闭最近最少使用的连接。|
|65|--without-http_upstream_zone_module|禁用HTTP_Upstream_Zone模块，该模块可以将由"Upstream"指令定义的服务器组运行时的状态存储在共享内存区域中。|
|66|--with-http_perl_module、--with-http_perl_module=dynamic|启用HTTP_Perl模块，用于在Perl中实现位置和变量处理程序，并可以将Perl调用到SSI中。默认情况下不构建此模块。 启用HTTP_Perl动态模块，允许在配置文件中通过"load_module"指定手动启用和禁用模块的使用。默认情况下不构建此模块。|
|67|--with-perl_modules_path=PATH|置一个用于保留Perl模块的目录路径。|
|68|--with-perl=PATH|设置Perl可执行命令文件的路径。|
|69|--http-log-path=PATH|设置访问日志文件存放目录路径。安装后，可以在主配置文件中使用"access_log"指令修改。|
|70|--http-client-body-temp-path=PATH|设置用于存储客户端请求主体的临时文件存放目录路径。安装后，可以在主配置文件中使用"client_body_temp_path"指令修改。|
|71|--http-proxy-temp-path=PATH|设置用于存储从代理服务器接受的数据临时文件存放目录路径。安装后，可以在主配置文件中使用"proxy_temp_path"指令修改。|
|72|--http-fastcgi-temp-path=PATH|设置用于存储从FastCGI服务器接受的数据临时文件存放目录路径。安装后，可以在主配置文件中使用"fastcgi_temp_path"指令修改。|
|73|--http-uwsgi-temp-path=PATH|设置用于存储从UwSGI服务器接受的数据临时文件存放目录路径。安装后，可以在主配置文件中使用"uwsgi_temp_path"指令修改。|
|74|--http-scgi-temp-path=PATH|设置用于存储从SCGI服务器接受的数据临时文件存放目录路径。安装后，可以在主配置文件中使用"scgi_temp_path"指令修改。|
|75|--without-http|禁用HTTP_Core模块，该模块为Nginx的核心模块，用于提供HTTP服务所有核心功能。|
|76|--without-http-cache|禁用HTTP缓存。|
|77|--with-mail|启用HTTP_Mail_Core模块，该模块为Nginx的核心模块，用于提供POP3/IMAP4/SMTP邮件代理服务。默认情况下不构建此模块。|
|78|--with-mail=dynamic|启用HTTP_Mail_Core动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|79|--with-mail_ssl_module|启用Mail_SSL模块，用于邮件代理服务支持SSL/TLS协议，需要OpenSSL库的支持。默认情况下不构建此模块。|
|80|--without-mail_pop3_module|禁用Mail_POP3模块，当启用HTTP_Mail_Core模块时，若你不想使用POP3协议，则可以考虑单独禁用此模块。不建议禁用。|
|81|--without-mail_imap_module|禁用Mail_IMAP模块，当启用HTTP_Mail_Core模块时，若你不想使用IMAP4协议，则可以考虑单独禁用此模块。不建议禁用。|
|82|--without-mail_smtp_module|禁用Mail_SMTP模块，当启用HTTP_Mail_Core模块时，若你不想使用SMTP协议，则可以考虑单独禁用此模块。不建议禁用。|
|83|--with-stream|启用Stream_Core模块，Nginx的核心模块，用于实现TCP/UDP代理和四层负载均衡功能。默认情况下不构建此模块。此模块自Nginx1.9.0版本开始可用。|
|84|--with-stream=dynamic |启用Stream_Core动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|85|--with-stream_ssl_module|启用Stream_SSL模块，用于提供SSL/TLS协议支持，需要OpenSSL库的支持。该模块用于Nginx四层负载功能中使用，需要开启Stream_Core模块。默认情况下不构建此模块。|
|86|--with-stream_realip_module|启用Stream_Realip模块，用于修改客户端请求头中客户端IP地址值，一般用于反向代理中，将真实的客户端IP传送给后端的应用服务器。该模块用于Nginx四层负载功能中使用，需要开启Stream_Core模块。默认情况下不构建此模块。|
|87|--with-stream_geoip_module|启用Stream_Geoip模块，用于处理不同地区的访问，当来自某一个区域的访问时将其重定向到对应的服务或者项目上，需要MaxMind GeoIP库的支持。该模块用于Nginx四层负载功能中使用，需要开启Stream_Core模块。默认情况下不构建此模块。|
|88|--with-stream_geoip_module=dynamic|启用Stream_Geoip动态模块，允许在配置文件中通过"load_module"指令手动启用和禁用模块的使用。默认情况下不构建此模块。|
|89|--with-stream_ssl_preread_module|启用Stream_SSL_Preread模块，用于从客户端Hello消息中提取信息，而不会终止SSL/TLS。该模块用于Nginx四层负载功能中使用，需要开启Stream_Core模块。默认情况下不构建此模块。|
|90|--without-stream_limit_conn_module|禁用Stream_Limit_Conn模块，该模块用于限制并发连接数量以及下载带宽限制功能。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|91|--without-stream_access_module|禁用Stream_Access模块，该模块用于限制对某些客户端地址的访问。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|92|--without-stream_geo_module|禁用Stream_Geo模块，该模块用于从指定变量中获取客户端的IP地址，并将其嵌入到另外一个变量中。默认情况下从"$remote_addr"变量中取得客户端的IP地址。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|93|--without-stream_map_module|禁用Stream_Map模块，该模块用于创建一个变量的映射表，结果变量可以是一个字符串也可以是另外一个变量。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|94|--without-stream_split_clients_module|禁用Stream_Splic_Clients模块，该模块用于创建适用于A/B测试的变量，AB测试也称之为拆分测试，也就是将一个项目的两个不同版本发布，看用户更喜欢用于那个版本，若版本A受欢迎则发布版本A。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。|
|95|--without-stream_return_module|禁用Stream_Return模块，该模块用于向客户端发送指定值，然后关闭连接。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|96|--without-stream_upstream_hash_module|禁用Stream_Upstream_Hash模块，该模块提供四层负载均衡的一种调度方法，一般用于基于SessionID的会话保持场景下，当开启Stream_Core模块时自动开启此功能。不建议禁用。|
|97|--without-stream_upstream_least_conn_module|禁用Stream_Upstream_IP_Hash模块，该模块提供四层负载均衡的一种调度方法，基于来源IP的会话保持方法，由于来源IP的不稳定性，我们一般很少采用此种方法。当开启Stream_Core模块时自动开启此功能。|
|98|--without-stream_upstream_zone_module|禁用Stream_Upstream_Zone模块，该模块可以将由"Upstream"指令定义的服务器组运行时的状态存储在共享内存区域中。该模块用于Nginx四层负载功能中使用，当开启Stream_Core模块时自动开启此功能。|
|99|--with-google_perftools_module|启用Google_Perftools模块，用于可以使用Google Performance Tools分析Nginx的工作进程，分析程序性能瓶颈。该模块适用于Nginx开发人员，默认情况下不构建此模块。|
|100|--with-cpp_test_module|启用Cpp_Test模块，用于C++测试。该模块适用于Nginx开发人员，默认情况下不构建此模块。|
|101|--add-module=PATH|添加第三方模块，需要指定第三方模块所在目录路径。|
|102|--add-dynamic-module=PATH|添加第三方动态模块，需要指定第三方动态模块所在目录路径。|
|103|--with-compat|启用动态模块兼容性。|
|104|--with-cc=PATH|设置GCC编译器所在目录路径。|
|105|--with-cpp=PATH|设置GCC-C++编译器所在目录路径。|
|106|--with-cc-opt=OPTIONS|设置将添加到CFLAGS变量的其他参数|
|107|--with-ld-opt=OPTIONS|设置将在连接期间使用的其他参数|
|108|--with-cpu-opt=CPU|设置CPU型号，为特定的CPU执行编译操作，有效的值：pentium, pentiumpro, pentium3, pentium4, athlon, opteron, sparc32, sparc64,ppc64。|
|109|--without-pcre|禁用PCRE库的使用。|
|110|--with-pcre|启用PCRE库的使用。PCRE库是一个Perl库，包含Perl兼容的正则表达式。|
|111|--with-pcre=DIR|若你是源码安装的PCRE库，则需要通过此项设置PCRE库的所在目录路径。|
|112|--with-pcre-opt=OPTIONS|为PCRE设置其他要编译的选项。|
|113|--with-pcre-jit|启用"即时编译"的支持，开启此项，则会利用"pcre_jit"指令快速编译PCRE库。|
|114|--with-zlib=DIR|若你是源码安装的Zlib库，则需要通过此项设置Zlib库的所在目录路径。当启用HTTP_Gzip模块的时候需要此库的支持。|
|115|--with-zlib-opt=OPTIONS|为Zlib设置其他要编译的选项。|
|116|--with-zlib-asm=CPU|为Zlib库的编译设置特定CPU，会加快编译速度，有效值：pentium, pentiumpro。|
|117|--with-libatomic|启用Libatomic_Ops库的使用。|
|118|--with-libatomic=DIR|若你是源码安装的Libatomic_Ops库，则需要通过此项设置Libatomic_Ops库的所在目录路径。|
|119|--with-openssl=DIR|若你是源码安装的OpenSSL库，则需要通过此项设置OpenSSL库的所在目录路径。|
|120|--with-openssl-opt=OPTIONS|为OpenSSL设置其他要编译的选项。|
|121|--with-debug|启用调试级别的日志。也可以手动修改主配置文件，使用"error_log /path/to/log debug;"指令设置调试级别的日志。|