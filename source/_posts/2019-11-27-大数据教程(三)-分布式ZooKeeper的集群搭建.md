---
title: 大数据教程(三)-分布式ZooKeeper的集群搭建
date: 2019-11-27 18:57:12
top: false
cover: false
categories: 大数据
tags: 
  - zookeeper
  - linux
---
### 一、本篇教程侧重点导读
1. 机器准备； 
2. zoo.cfg文件配置；
3. 集群操作； 
4. zk常用命令；
5. zk选举机制；


### 二、本篇教程用的软件、技术和说明
1. 沿用第一篇大数据教程中的环境及软件；
2. ZooKeeper安装包版本为：3.5.6

### 三、机器准备
按照第一篇的集群规划，将在slave3、slave4、slave5上部署zk集群
1. 这三台机器上下载zk安装包：
````bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6-bin.tar.gz
````
2. 解压并改名
````bash
# 解压
tar -xzvf apache-zookeeper-3.5.6-bin.tar.gz -C /usr/local
#改名（有强迫症）
mv apache-zookeeper-3.5.6-bin zookeeper-3.5.6
````

### 四、zoo.cfg文件配置
1. zk集群运行需要存储数据到磁盘目录，在zookeeper-3.5.6目录下新建一个目录
````bash
# 在/usr/local/zookeeper-3.5.6 目录下执行
mkdir zkDataDir
````
2. 将conf目录下的zoo_sample.cfg文件复制一份，并重命名为zoo.cfg
````bash
cp zoo_sample.cfg zoo.cfg
````
3. 我的配置文件示例
````bash
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/usr/local/zookeeper-3.5.6/zkDataDir
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=192.168.6.103:2888:3888
server.2=192.168.6.104:2888:3888
server.3=192.168.6.105:2888:3888
````
4. 主要有是修改了数据目录dataDir和最后三行的集群配置，其他用默认就好了

5. 最后三行的集群配置解析
 `server.1=192.168.6.103:2888:3888`
 server.1中的数字1表示这个是第几号服务器；
 192.168.6.103是这个服务器的ip地址；
 2888是这个服务器与集群中的Leader服务器交换信息的端口(集群通讯端口)；
 3888是集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口（选举端口）。

6. 在zkDataDir目录中新建一个myid文件（三个服务器都需要此文件），文件内容就是对应服务器的编号，
  在配置文件中的最后三行配置的server.1是指定192.168.6.103服务器的，所以要在myid文件中写入编号`1`，在192.168.6.104服务器上myid文件中写入编号`2`，105服务器上写入`3`

**<font color=red>防坑</font>**：以上步骤除了第6步骤外，集群的其他配置都一样；

### 五、集群操作
1. 启动zk：`bin/zkServer.sh start`
2. 停止zk：`bin/zkServer.sh stop`
3. 重启zk：`bin/zkServer.sh restart`
4. 查看状态：`bin/zkServer.sh status`
5. 这里我编写了一个shell脚本，可以方便的管理zk集群：
 **<font color=green>说明一下</font>**：这个脚本可以在master、slave1-5的任意一台机器上运行，但前提是运行该脚本的机器需能够免密登陆到slave3、4、5上
````bash
#!/bin/bash
echo '你执行的命令是：'$1
for host in slave3 slave4 slave5
do
echo '开始在'$host‘上的执行命令’
ssh $host "source /etc/profile;/usr/local/zookeeper-3.5.6/bin/zkServer.sh $1"
echo '执行完毕！退出'$host
done
````

 脚本运行效果：
 <img style="width:85%;height:85%" src="http://staticfile.erdongchen.top/blog/blogPicture/20191128/5.1.png"  align=left/>

### 六、zk常用命令
|序号|命令|含义|
|:-:|:---:|:---------:|
|1|create /a '我是数据'|创建默认永久节点(注意父节点必须要存在)|
|2|create -e /b '我是数据'|创建临时节点(注意父节点必须要存在)|
|3|create -s /a/c '我是数据'|创建带顺序编号的永久节点(注意父节点必须要存在)|
|4|create -e -s /a/d '我是数据'|创建临时节点带顺序编号(注意父节点必须要存在)|
|5|get /a |获取znode的数据|
|6|set /a '我是数据' |设置znode的数据|
|7|ls /a|查看znode的树下的节点|
|8|delete /a|只能删除没有子节点的znode|
|9|rmr /a|不管里面有多少子节点znode，统统删除|
|10|stat /a|查看节点信息|
|11|quit|退出zookeeper|
|12|connect 192.168.6.105:2181|连接集群中的别的节点|
|13|ls2 /a|查看当前节点的子节点及当前节点的信息|
|14|get /a  watch |a节点的数据变化事件注册了监听|
|15|ls /a watch |对a节点的子节点变化事件注册了监听|


附：zk节点数据结构

|节点名称|含义|
|:---:|:---------:|
|czxid|节点创建时的事务id|
|mzxid|节点修改时的事务id|
|pzxid|最近一次修改子节点的事务id|
|ctime|节点创建时间|
|mtime|节点修改时间|
|dataversion|数据版本，每修改一次加1|
|cversion|子节点版本号，每修改一次子节点，加1|
|aversion|ACL版本号|
|ephemeralOwner|是否是临时节点，0代表永久节点|
|dataLength|数据长度|
|numChildren|子节点个数|


### 七、zk选举机制
zk的选举机制蛮有意思的，先上一张网络配图（流程图）：
<img style="width:85%;height:85%" src="http://staticfile.erdongchen.top/blog/blogPicture/20191128/7.1.jpg"  align=left/>

[**zookeeper的选举机制**](https://blog.csdn.net/wyqwilliam/article/details/83537139 "zookeeper的选举机制"),这篇博文写的非常详细，值得一看。

选举出现的情况分为2种：
1. 服务器初始化启动。
 无论是哪种情况，都将优先对比ZXID（事务ID），其次是SID（myid里面配置的serverId）
 由于第一此启动的时候ZXID都是0 ，所以会对比myid，也就是说SID大的，被选举的几率就大，
 例如：
 在slave3、slave4、slave5上面部署zk集群，myid配置文件中分别是1,2,3，假设zk集群的启动顺序是slave4-slave3-slave5：
 1). slave4启动后，把票投给自己，广播后发现集群中只有自己一个，集群中有三台zk（配置文件中配置了三台），票数没有过半，zk就会处于等待状态（LOOKING）；
 2). slave3启动，投自己一票，广播后发现有个兄弟在线，接收到他的投票，首先会对比ZXID都是0，在对比myid，发现slave4的myid比自己大，slave3就会更新自己的投票，把票投给slave4；而对于slave4而言，无须更新自己的投票，只是再次向集群中所有机器广播上一次投票信息即可；
 3). 投票统计slave4票数过半，slave4变成leader（LEADING），slave3变成小弟（FOLLOWING）；
 4). slave5启动后发现集群中已经存到leader，自动变成小弟。
 
 
2. 服务器运行期间无法和Leader保持连接。
 1). 服务器运行期间，leader挂掉之后，集群中的FOLLOWING都会变成（LOOKING）状态，并进入选举过程；
 2). 每个zk都会为自己投一票，运行期间每个zk上的ZXID是有可能不同的，选举线程会将收到的票数进行统计比较，还有优先比较ZXID，这时候ZXID大的zk成为leader的几率就大
 
选举算法和选举具体细节上面那篇博文已经很清楚了，推荐阅读！












