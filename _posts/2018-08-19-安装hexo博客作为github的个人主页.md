---
layout: post
title: 安装hexo博客作为github的个人主页
tags:
- github
- hexo
---

<!--*-->
<!--more-->

## 安装hexo

### 安装依赖环境
1. [git](https://git-scm.com/download/win)
2. [nodejs](https://nodejs.org/en/)

### 安装[hexo](https://hexo.io/zh-cn/)
```bash
npm install hexo-cli -g
#blog 是一个未创建的博客根目录
hexo init blog
cd blog
npm install
#这是用于部署到github上的插件
npm install hexo-deployer-git --save

```

### 注册并配置github  

#### 1.新建项目
注册完github账号以后新建一个和你自己用户名(例如tom)相同前缀的仓库(tom.github.io)
#### 2.创建RSA密钥
```bash
# 配置本地git 
#yourname, youremail替换成自己的
git config --global user.name yourname
git config --global user.email youremail

#生成密钥 youremail使用上一步配置的
ssh-keygen -t rsa -C youremail
#在~/.ssh/下找到后缀为pub的文件，复制其内容
#在github的设置界面中找到ssh keys将复制的密钥粘贴进去
```

### 编写配置文件
```bash
#在 _config.yml的最后复制以下内容并替换原有的deploy开始到最后的内容
#其中repo需要替换成你自己github个人主页的ssh链接
deploy:
  type: git
  repo: git@github.com:rivaen/rivaen.github.io.git
  branch: master
```

### 选择并配置好看的hexo皮肤
我这个主题是[Archer](https://github.com/fi3ework/hexo-theme-archer),请按照其文档配置


## hexo命令
```bash
hexo g #生成网页模板到根目录下的public
hexo s [-p port]#在本地的port端口开启hexo服务，默认4000端口
hexo d #通过配置文件的git地址部署到该git地址
```
