---
title: DataX-使用idea调试DataX源码
top: false
cover: false
date: 2020-04-27 14:01:24
categories: DataX
tags: 
  - java
  - 技术
---
### 一、本篇教程侧重点导读
> 使用idea调试DataX的源码；

### 二、本篇教程用的软件、技术和说明
> 1. jdk版本：1.8.0_202-b08；
> 2. Python版本：2.7.18（官方推荐2.6.X）；
> 3. Maven版本；3.6.0 ；
> 4. DataX是直接拉取的master分支上的源码；
> 5. 准备好一个json文件；

### 三、拉取源码在本机编译打包
1. 用git直接克隆到本地：https://github.com/alibaba/DataX.git
2. 通过maven打包：
````
cd  {DataX_source_code_home}
mvn -U clean package assembly:assembly -Dmaven.test.skip=true
````
 打包成功，日志显示如下：
 ````
[INFO] BUILD SUCCESS
[INFO] -----------------------------------------------------------------
[INFO] Total time: 08:12 min
[INFO] Finished at: 2015-12-13T16:26:48+08:00
[INFO] Final Memory: 133M/960M
[INFO] -----------------------------------------------------------------
 ````
 **<font color=red>防坑</font>**：DataX工程里面配置的是阿里的maven镜像，打起包来相对较快，但是插件较多，也是要好几分的，<font color=red>另外打包的时候有的插件老是报错，可以多执行几遍maven打包命令，如果用不到那个插件可以先从父工程里面注释掉其引用。</font>
 
### 四、用python执行DataX脚本
起一个cmd窗口执行脚本
````
E:\datax_source\DataX\target\datax\datax\bin\datax.py E:\datax_source\DataX\target\datax\datax\bin\debug.json -d
```` 
 命令执行后如下图所示：
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200427/4.1.png"  align=left/>
  说明：
  1. datax.py脚本是打包成功后，在target目录下的那个文件
  2. debug.json是用户配置同步的文件
  3. -d 是运行debug模式
  4. ip是本机的ip，端口9999是在datax.py脚本文件中，debug时的默认开放端口，可修改
  5. 此时，本机的这个端口已经开放了，等待程序调用
 
### 五、在idea中配置Remote
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200427/5.1.png"  align=left/>
 配置完成后，在DataX的主程序入口打个debug点（DataX的主程序入口在com.alibaba.datax.core.Engine类的main方法）：
 <img style="width:85%;height:85%" src="https://staticfile.erdongchen.top/blog/blogPicture/20200427/5.2.png"  align=left/>
 即可开始源码解析之旅！
 
### 六、乱码解决
 在cmd窗口debug的时候，DataX打印的日志可能会有乱码，可以在命令行键入`chcp 65001`即可；这个方法只可以在该cmd窗口有效，下次再打开cmd窗口就失效了，可以修改注册表，来永久修改，方法自行百度。