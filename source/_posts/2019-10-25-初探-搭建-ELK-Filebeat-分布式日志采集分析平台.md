---
title: 初探-搭建 ELK+Filebeat 分布式日志采集分析平台
date: 2019-10-25 17:58:20
categories: linux
tags: 
  - 日志分析平台
  - 环境搭建
  - 技术
  - ELK
---

### * 本篇教程侧重点导读
1. 教程涉及到的软件，技术，
2. ELK日志采集分析平台的技术框架介绍；
3. elasticsearch的安装部署单机、集群环境；
4. kibana的安装部署；
5. logtash的安装部署；
6. Filebeat日志采集的安装部署；
7. 打通链路，日志采集->日志过滤分析->日志存储搜索->日志分析展示；
8. 数据库数据的采集；
9. elasticsearch-head插件安装；

### 一、教程涉及到的软件，技术
1. 使用到linux系统：CentOS 7；
2. 安装ELK的前提是服务器需要java环境，我用的是：1.8.0_112;
3. elasticsearch、kibana、logtash和Filebeat都要使用同一版本（我使用的是7.4.0）；
4. ELK官方地址；[www.elastic.co](https://www.elastic.co "go")
5. 本篇使用了三台服务器，IP地址分别为192.168.1.220、192.168.1.221、192.168.1.222，并且在每台服务器上创建了一个账号和组（elk）；
6. 此篇教程是分享个人理解的ELK技术框架，以及搭建过程，如有错误，敬请指出！

### 二、ELK日志采集分析平台的技术框架介绍
话不多说，先上一张网络图！
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/elk架构图.png"  align=left/>

### 三、elasticsearch的安装部署集群环境
1. 在三台服务器上都丢一个es(elasticsearch)安装包，并解压；
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/3.1.png"  align=left/>
2. 在三台服务器上都创建一个账号并把elasticsearch-7.4.0文件夹的所属用户所属组改一下（elk官方不建议使用root运行ELK）
````bash
# 创建组
groupadd elk
# 创建用户
useradd elk -g elk -p 123456
# 修改文件夹权限
chonw -R elk:elk elasticsearch-7.4.0
````
3. 我把192.168.1.220作为es主节点，221和222作为数据节点，集群环境配置如下：
在服务器220上的es解压目录进入config目录，编辑elasticsearch.yml文件，新增如下内容：
````bash
# 集群中的名称
cluster.name: es-master-node
# 该节点名称
node.name: master
# 该节点设置为主节点
node.master: true
# 该节点不是数据节点
node.data: false
# 监听全部ip，在实际环境中应设置为一个安全的ip
network.host: 0.0.0.0
# es服务端口号
http.port: 9200
# 配置自动发现
discovery.seed_hosts: ["192.168.1.220","192.168.1.221", "192.168.1.222"]
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
# 初始化主节点
cluster.initial_master_nodes: 192.168.1.220
````

4. 服务器221和222上的elasticsearch.yml文件配置为：
````bash
# 集群中的名称
cluster.name: es-master-node
# 该节点名称(node.name 可以自己起名字  只要三台服务器的节点明恒不一样就行)
node.name: 
# 该节点从节点
node.master: false
# 该节点是数据节点
node.data: true
# 监听全部ip，在实际环境中应设置为一个安全的ip
network.host: 0.0.0.0
# es服务端口号
http.port: 9200
# 配置自动发现
discovery.seed_hosts: ["192.168.1.220","192.168.1.221", "192.168.1.222"]
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
````

5. 在es主节点服务器上启动es：
````bash
./bin/elasticsearch &
````
 **<font color=red>启动报错</font>**：
 <font color=red>ERROR: [1] bootstrap checks failed
 [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]</font>
 **<font color=green>解决方案</font>**：
 <font color=green>切换都root用户下执行</font>
````bash
# 查看硬限制
ulimit -Hn
# 编辑文件
vim /etc/security/limits.conf
# 添加如下配置(elk为启动es的用户名)
elk soft nofile 65536
elk hard nofile 65536
````
 <font color=green>需退出用户后重新登录，再次查看elk用户的硬限制，如果变为65536，说明设置成功！</font>
 
6. 主节点启动成功后，在启动两个从节点，其中9200是数据传输时端口，9300是集群通信端口

7. 验证：
````bash
# es集群健康检查
curl '192.168.1.220:9200/_cluster/health?pretty'
# 返回的内容
{
  "cluster_name" : "es-master-node",
  "status" : "green", # 为green则代表健康没问题，如果是yellow或者red则是集群有问题
  "timed_out" : false, # 是否有超时
  "number_of_nodes" : 3, # 集群中的节点数量
  "number_of_data_nodes" : 2, # 集群中data节点的数量
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
# 查看集群详细信息
curl '192.168.1.220:9200/_cluster/state?pretty'
````


### 四、kibana的安装部署（我是部署在220上,部署服务器随意选择，只要配置对就好）
1. 将安装包解压至/usr/local/elk目录下，并修改权限到elk用户上
2. 修改kibana解压目录下config里面的配置文件：kibana.yml，新增如下内容：
````bash
# 配置kibana的端口
server.port: 5601
# 配置监听ip
server.host: 192.168.1.220
# 配置es服务器的ip，如果是集群则配置该集群中主节点的ip
elasticsearch.hosts: ["http://192.168.1.220:9200"]
# 配置kibana的日志文件路径，不然默认是messages里记录日志
logging.dest: /usr/local/elk/kibana-7.4.0-linux-x86_64/logs/kibana.log
i18n.locale: "zh-CN"
````
3. 浏览器访问；192.168.1.220:5601

### 五、logtash的安装部署（我是部署在221上，任意服务器上都可部署）
1. 将安装包解压至/usr/local/elk目录下，并修改权限到elk用户上
2. 在config目录下新建一个文件myLogstash.conf
````bash
# 创建文件myLogstash.conf
touch myLogstash.conf
# 增加如下内容
input { #定义日志源
  beats {
    port => 9900 #监听端口
  }
}
output { #日志输出源
  if "nginxlog" in [tags]{ #这里的配置是接收Filebeat传过来的日志做分类处理，存到不同的es表里
   elasticsearch { # 输出到es
    hosts => ["http://192.168.1.220:9200"] # es主节点所在IP
    index => "nginx-%{+YYYY.MM.dd}" # 定义索引
   }
  }
  if "tomcatlog" in [tags]{
   elasticsearch {
    hosts => ["http://192.168.1.220:9200"]
    index => "tomcat-%{+YYYY.MM.dd}"
   }
  }
}
````

3. 检测配置文件是否正确
````bash
./bin/logstash --path.settings /config -f config/myLogstash.conf --config.test_and_exit
 #参数说明
 --path.settings # 指定logstash的配置文件所在目录
 -f # 指定需要被检测的配置文件
 --config.test_and_exit # 检测完之后就退出，不启动logstash
````

4. 启动logstash
````bash
./bin/logstash -f config/myLogstash.conf &
````

### 六、将Filebeat日志采集部署至待采集的机器上
需求：需采集192.168.1.182上的nginx和tomcat日志,182服务器是windows server服务器，所以我下载的Filebeat是zip格式包
1. 将Filebeat程序放到182上，并解压
2. 修改配置文件filebet.yml:
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/6.2.png"  align=left/>
3. 启动Filebeat
````bash
# Filebeat目录执行
filebet.exe -c filebeat.yml
````

### 七、链路打通
1. 随便访问一下nginx和tomcat，使其产生日志
2. 浏览器访问kibana服务查看es中的索引：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/7.2.png"  align=left/>
3. 或者访问 curl 192.168.1.220:9200/_cat/indices?v 同样可以查看到有索引建立
4. 如上第二步或者第三步，可以看到，在logstash配置文件中定义的两个索引成功获取到了，证明配置没问题，logstash与es通信正常
5. 在浏览器界面打开kibana服务，配置es中的两个索引：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/7.5.png"  align=left/>
6. 在Discover界面查看采集的日志数据：
<img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20191025/7.6.png"  align=left/>

### 八、数据库数据的采集
logstash同样也可以采集数据库中的数据到es，只需要改input数据源中的相关配置就ok了，这里不再叙述详细步骤

### 九、elasticsearch-head插件安装
参考链接：[elasticsearch-head插件安装](https://blog.csdn.net/qq924862077/article/details/79994565 "go")


