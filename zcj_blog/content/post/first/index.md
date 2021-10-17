---
title: "我的第一份博客"
date: 2021-09-04T10:05:40+08:00
draft: true
categories:
    - computer
---
###  为什么写博客
-  总结开发中遇到的问题。工作过后发现自己并不擅长对知识点的总结，导致总是遇到相同的问题，过段时间需要重新查找解决方案
-  记录学习的知识，不断的温习。学的东西过于碎片化，导致知识不成体系。时间长了，碎片的知识也忘记了
-  提升自己的专业技能。通过写博客提升自己的能力
-  形成自己的技术栈，遇到的志同道合的朋友
###   为什么选择hugo来搭建自己的博客
- Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。  
-  操作简单，使用Markdown直接生成静态网页  
-  免费且以维护, 在github上就可供他人访问，无需购买服务器，维护简单
-  发表文章直接push到自己仓库即可
### 下载hogo的源码
 >git clone  https://github.com/gohugoio/hugo.git  
   >git branch 查看单前代码的分支  
   >git branch -a 查看全部分支  
   >git checkout branch 切换分支  
   >git branch 分支名  创建自己的本地分子
### 编译源码
- 在master分支下，在main.go 的目录下使用命令: go build  在目录下生成hugo.exe
- 在cmd下使用hugo 查看是否编译成功 编译成功 会打印hugo的版本 
- 安装成功
### 生成站点
 - 使用命令：hugo new site /目录  
 - cd /目录 查看到  
 -                 ▸ archetypes/  
                   ▸ content/  
                   ▸ layouts/  
                   ▸ static/  
                   config.toml
- 创建站点成功
### 创建md文章
- 使用命令: hugo  new 文章名.md  在content/ 下生成该md文件
### 选择博客主题模板
- hugo 提供很多的主题博客模板：https://themes.gohugo.io/
- 创建theme文件夹，将主题模板放在里面 ：mkdir themes
- 进入该文件夹：cd themes
- 下载主题，使用git clone 主题模板 ：git clone https://github.com/spf13/hyde.git

### 配置config.toml文件
- config.toul 文件hugo 的配置文件，可以配置主题模板，个人信息等(主题模板中相应的配置文件)如
-      baseurl = "http://****.com/"  //发布的网站
       languageCode = "ja"           //使用的语言
       title = "xxxx.COM"      //网站名称等
        [Params]
        subtitle = "I would like to be a layer 3 switch."
        facebook = "https://facebook.com/foobar"
        twitter = "https://twitter.com/foobar"
        github = "https://github.com/foobar"
        profile = "/images/profile.png"
        copyright = "Written by Asuka Suzuki"
        analytics = "UA-XXXXXXXX-X"
        
###  运行
#### 本地运行
-  使用命令：hugo server --buildDrafts 配置正确则会出现： http://localhost:1313/ (bind address 127.0.0.1) 点击在浏览器中运行
#### 推送到gitgub
- 首先在GitHub上创建一个Repository，命名为：github用户名.github.io 
- 修改config.toml 配置文件：将baseurl = "http://github用户名.github.io" 
- 使用命令：hugo  --buildDrafts 在本地生成public的文件夹
-   --buildDrafts 参数的主用是将你的文章在主题中出现
-       cd public 进入到public文件夹
        $ git init  初始化本地仓库
        $ git remote add origin https://github.com/github用户名/github用户名.github.io //添加原创仓库
       或者直接 git clone 
        $ git add -A
        $ git commit -m "first commit"
        $ git push -u origin master  //推到远端
- 使用 "http://github用户名.github.io"就可访问


