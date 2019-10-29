---
title: 开通博客的第一天干嘛？当然是手把手教你如何搭建博客啦！
date: 2019-08-15 17:21:12
categories: 其他教程
tags: 
  - 博客
  - 环境搭建
  - 技术
---
### 本套博客搭建教程的前置条件：
1. 域名一个；
2. github账号一个；
3. 本机已安装node.js；
4. 本机已安装git，并且本地仓库已关联到自己github上的一个repository，且repository的名字必须为‘你的github账号.github.io’

### 下面进入本教程：
**基础部分**
1. 切换国内源  `npm config set registry="http://registry.cnpmjs.org"`
2. 安装hexo  `npm install -g hexo`
3. 初始化Hexo  `hexo init  `
4. 安装必要的模块  `npm install `
5. 生成静态文件  `hexo g  或者  hexo generate `
6. 本地测试（浏览器查看：<http://localhost:4000>）  `hexo s  或者  hexo server`

**个性化设置**
1. hexo配置文件修改，在根目录下（执行hexo init的那个目录）会有一个_config.yml的配置文件，修改如下内容：
```java
# Hexo Configuration
# Docs: http://hexo.io/docs/configuration.html
# Source: https://github.com/hexojs/hexo/
Site
   title: 换成你的主页标题
   subtitle: 主页副标题
   description: 主页介绍的一句话
   author: 你的名字
   language: zh-CN #语言
   timezone: Asia/Shanghai #时区
#
#    # URL
#    ## If your site is put in a subdirectory,
#    ##set url as 'http://yoursite.com/child' and root as '/child/'
#    # 更换域名是需要配置的参数
   url: http://voidking.com
   root: /
#
#    # Extensions
#    ## Plugins: http://hexo.io/plugins/
#    ## Themes: http://hexo.io/themes/
   theme: 你用的主题文件夹名字 # themes下的文件
#
#    # Deployment
#    ## Docs: http://hexo.io/docs/deployment.html
   deploy:
       type: git
       # GitHub仓库地址，此地址一定要是前置条件第四条的那个地址
       # 地址格式：https://github.com/你的github账号/你的github账号.github.io.git
       # 例如
       repository: https://github.com/a878804506/a878804506.github.io.git
       branch: master
```
 **<font color=red>防坑：_config.yml配置参数时，注意冒号后面一定要有一个空格</font>**

2. 修改主题
找一个自己喜欢的主题，然后clone到hexo根目录下的themes目录下
eg：切换到根目录下文件下，cmd执行
`git clone -b master https://github.com/lewis-geek/hexo-theme-Aath.git themes/aath`
下载好后，你在_config.yml（主题是根目录下的）中的theme:处配置成你下在的主题名字（就是下载的主题文件夹名字 ）
        
3. 上传的github上（会上传到前置条件第四条的那个地址下）
如果想把原来的清除 `hexo clean`
重新生成静态文件 `hexo g`
上传到提交文件 `hexo d`
此时就可以用浏览器访问前置条件第四条的那个地址，会就出现你的博客首页
        
**附加:域名配置**
1. 到你域名提供商那里配置域名解析规则,记录类型为：CNAME
2. 记录值为：前置条件第四条的那个地址
3. Hexo根目录下会有一个source目录，在source下新建一个CNAME文件，文件内容是你刚刚配置好的域名
4. 再次修改Hexo根目录下的_config.yml配置文件，填写url
5. 再次执行打包上传命令 `hexo clean && hexo g && hexo d`
 **<font color=red>防坑：域名访问404，但是xxx.github.io能访问，那就说明还是你的域名配置有问题，仔细检查</font>**