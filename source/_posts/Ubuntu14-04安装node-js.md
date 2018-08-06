---
title: Ubuntu 14.04 安装 node.js
date: 2017-03-16 22:17:49
tags: 
  - Linux
  - Ubuntu
categories: 技术
---
在工作的机器上想装个nodejs新版的，用`apt-get install`装上的nodejs有问题，摸索了一个靠谱的安装方法，操作起来也比较快。
<!--more-->

1. 先在 https://npm.taobao.org/mirrors/node/ 中找个最新版的编译好的压缩包，我下载的是 `node-v6.10.0-linux-x64.tar.gz`
2. 解压，我直接用图形界面解压了，得到一个文件夹
3. 切进去测试一下：
   ```shell
   cd node-v6.10.0-linux-x64/bin
   ./node -v 
   ```
   打印出来版本号，没问题。
4. 接下来把这个文件夹里的node和npm设置到全局：
   ```shell
   ln -s /home/username/下载/node-v6.10.0-linux-x64/bin/node /usr/local/bin/node
   ln -s /home/username/下载/node-v6.10.0-linux-x64/bin/npm /usr/local/bin/npm
   ```
5. 最后测试一下：
   ```shell
   node -v
   npm -v
   ```
   分别打印出来：
   > v6.10.0
   > 3.10.10

完成!