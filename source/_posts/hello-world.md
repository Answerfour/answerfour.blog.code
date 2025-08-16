---
title: Hello World 
date: 2024-02-05
tags: 
categories: 
cover: /media/featureimages/0.jpg
description: My first blog!
---
### The idea was initially conceived.

> 事情起源于一个小小的念头。但这并不是一时念起。起初是因为遗憾吧，我的收藏夹里收藏了很多技术，但是我一直都只是在外面观望驻足，备考研究生考试的这一年，我几乎忘记了所有的编程技巧，我对技术的热爱因为现实的自欺而一直停留在过去，直到现在我才敢说我以及走出来了，形而上学上的自我纠结让我好一段时间开始反思自己，这个问题对内上，它解释为自我的发展上限和在社会竞争关系上的意识对待问题，对内上则表现为存在主义上的迷茫，所幸书籍给了我答案，我就像加缪书里的西西弗，想通之前我推石上山，想通之后我还是推石，只不过上不上山无所谓了。



个人博客的第一次搭建是我大一时候的事情了，那个时候我匆匆交了一份问卷上去，现在我终于能好好地再把这张试卷再做一篇。借此顺便记录一下我这几天的学习笔记吧。

### 1 安装Nodejs/git，搭建github仓库

```bash
node -v	#查看node版本
npm -v	#查看npm版本
npm install -g cnpm --registry=http://registry.npm.taobao.org	#安装淘宝的cnpm 管理器
cnpm -v	#查看cnpm版本
cnpm install -g hexo-cli    #安装hexo框架
hexo -v	#查看hexo版本
mkdir blog	#创建blog目录
cd blog	 #进入blog目录
sudo hexo init 	#生成博客 初始化博客
hexo s	#启动本地博客服务
http://localhost:4000/	#本地访问地址

hexo n "我的第一篇文章" #创建新的文章 

#Github创建一个新的仓库 YourGithubName.github.io
```

### 2 检测4000端口是否被占用

```cmd
netstat -ano #列出所有端口的使用情况
			 #列出计算机上所有端口的使用情况，包括TCP和UDP端口，以及这些端口对应的进程ID（PID）。-a 参数表示显示所有连接和侦听端口，-n 参数表示以数字形式显示地址和端口号，-o 参数显示拥有的进程ID。
netstat -aon|findstr "4000" #查看被占用端口对应的 PID
tasklist|findstr "30806" #查看指定 PID 的进程
taskkill /T /F /PID 30806  #结束进程#强制（/F参数）杀死 pid 为 9088 的所有进程包括子进程（/T参数）
						   #之后我们就可以结束掉这个进程，这样我们就可以释放该端口来使用了
```

### 3 配置_config.yml 

```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: 'git' 
  repo: 'https://github.com/Answerfour/Answerfour.github.io.git' #个人仓库名
  branch: 'master'
```

### 4 部署仓库

```bash
cnpm install --save hexo-deployer-git #在blog目录下安装git部署插件

hexo d	#部署到Github仓库里
```

### 5 修改主题

```bash
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia  #下载yilia主题到本地

#修改hexo根目录下的 _config.yml 文件 ： theme: yilia

hexo c	#清理一下
hexo g	#生成
hexo d	#部署到远程Github仓库

https://YourGithubName.github.io/  #查看博客
```
