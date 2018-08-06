title: 建立自己的hexo博客
date: 2016-03-06 09:12:00
tags: 
  - hexo 
  - github
categories: 技术
---
# 前言
这段时间建立了这个博客，参考了很多人写的教程，也踩过了很多坑，最终是搭建成功。鉴于hexo和Github都是免费使用的，我有必要帮忙宣传一下hexo和Github。另外，也分享一点自己的经验给网友参考。  
**注意：**此文仅适合windows平台  
<!--more-->
# 准备环境
## 安装 GitHub Desktop 
[GitHub Desktop](https://desktop.github.com/)  
安装过程很简单，没什么好说的  
**注意：**如果你没有稳定的"科学上网"的梯子，这个客户端估计安装不了，因为它是在线安装的，不过还好，网上有人提供了离线安装版。我这里把资源分享在我的[百度云](http://pan.baidu.com/s/1nu4k3NF)里,提取码:z5oz
## 安装Node.js
[Node.js](https://nodejs.org/en/)  
安装完成后可能需要添加`Path`环境变量，防止`npm`命令无效  
> ;C:\Program Files\nodejs\node_modules\npm  

## 安装hexo
以上步骤都完成后，打开桌面或者所有程序里的`Git shell`,输入以下命令即可安装
> npm install -g hexo-cli  
> npm install hexo --save  

# 初始化
## 创建Hexo文件夹
新建一个文件夹用于存放hexo的所有文件，如:`D:\Github\hexo`，进入`Git shell`，先切换到hexo文件夹，如:`cd d:\Github\hexo`，接着执行以下命令进行初始化：
> hexo init  

## 安装hexo插件
> npm install hexo-generator-index --save  
> npm install hexo-generator-archive --save  
> npm install hexo-generator-category --save  
> npm install hexo-generator-tag --save  
> npm install hexo-server --save  
> npm install hexo-deployer-git --save  
> npm install hexo-deployer-heroku --save  
> npm install hexo-deployer-rsync --save  
> npm install hexo-deployer-openshift --save  
> npm install hexo-renderer-marked@0.2 --save  
> npm install hexo-renderer-stylus@0.2 --save  
> npm install hexo-generator-feed@1 --save  
> npm install hexo-generator-sitemap@1 --save  

## 本地查看
cd到你的init目录，执行如下命令，生成静态页面至hexo\public\目录，然后到浏览器输入`localhost:4000`看看  
> hexo generate
> hexo server  

# 部署博客到Github上
## 注册github帐户
[Github](https://github.com/)
(已有的请忽略这一步)
## 创建Repository
Repository的名字必须是`your_name.github.io`，比如github的账号是jack，那么Repository的名字就应该是`jack.github.io`
## 修改配置文件
+ 到你刚刚创建的仓库中，复制HTTPS中的地址
+ 找到你的hexo根目录下的`_config.yml`
+ 加入如下代码，其中的repository就改成你刚刚复制的地址，your_name是你的github账号，保存 　　
```
deploy:  
　　type: git  
　　repository: https://github.com/your_name/your_name.github.io.git  
　　branch: master
```
## 设置SSH keys
1. 查看你的用户文件夹下的`.ssh`文件夹，里面要是有文件的话，都删掉
2. 输入以下指令，引号中输入你注册github时用的邮箱，一路回车
   > ssh-keygen -t rsa -C "xxxxxxx@qq.com"　　

3. 再输入以下指令
   > ssh-agent -s  

4. 接着输入以下指令
   > ssh-add ~/.ssh/id_rsa  
  
5. 打开你的github，在设置里找到`SSH keys`，点击`Add SHH key`,`title`中随便输入什么，`key`里输入`id_rsa.pub`中的内容，`id_rsa.pub`这个文件生成在用户下.ssh文件夹中  
**这一步结束之后就快完成啦**  

## 完成部署
> hexo g
> hexo d  
  
现在在浏览器中输入你的gitub的仓库名，比如我的是：  
    howshea.github.io  
是不是成功啦，接下来就是定制主题什么的，这些很简单，网上教程一大堆，我就不写了，so easy.  
# 参考文献
1. [hexo你的博客|不如](http://ibruce.info/2013/11/22/hexo-your-blog/)
2. [使用GitHub和Hexo搭建免费静态Blog](https://segmentfault.com/a/1190000003101692)
3. [hexo系列教程：（二）搭建hexo博客](http://zipperary.com/2013/05/28/hexo-guide-2/)
4. [史上最详细“截图”搭建Hexo博客并部署到Github](http://jingyan.baidu.com/article/d8072ac47aca0fec95cefd2d.html)

