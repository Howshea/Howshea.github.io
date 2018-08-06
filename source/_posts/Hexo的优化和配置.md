---
title: Hexo 的优化和配置
date: 2016-03-06 21:10
tags: 
  - hexo
  - 域名解析
categories: 技术
---
<big><b>序：</b></big> 这个博客已经加上了多说评论系统，修改了评论框的css样式，好看了一些；今天在阿里云那里买了.com的域名，折腾了半天终于也弄好了域名解析，嗯……还是想把经验分享一下
<!--more-->
# <small>购买域名以及完成域名解析</small>  

- 域名是在[阿里云万网](http://wanwang.aliyun.com/)那里买的，看网上大多数人都推荐在[GoDaddy](https://sg.godaddy.com/zh/)，我觉得都行吧。买域名的时候注意，有些域名在国内好像不能解析，.cn要备案才能使用，所以选择.com和.org等老牌域名还是比较省心的。  
- 域名解析也是用的阿里云万网的免费域名解析服务，这里要注意几点  
1. 我一开始像一个教程里设置成如下这样，结果没解析成功
![](http://ww4.sinaimg.cn/large/8127619agw1f1nj4qpk9kj20nq031jrc.jpg)
后来把主机记录设置`@`、记录类型改成`A`、记录值改成了我的github pages的ip地址就ok了
2. 解析前要在仓库里建一个`CNAME`文件，里面内容是你的域名，这里有个问题，你建了这个文件之后，以后一旦`hexo d -g`，这个文件就没了，所以还要在hexo文件夹下source下也建个CNAME文件，里面的内容也一样。  

# <small>添加多说评论系统</small>  
+ 这个没什么难的，在多说网址里设置一下就行，这里分享一下自定义css样式，因为默认的实在太丑  

```css  
#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current {border:0px;color:#848568;text-shadow:none;background:#dddfc2}
#ds-thread #ds-reset .ds-highlight {font-family:Arial, Helvetica, sans-serif;font-size:14px;font-weight:bold;color:#848568 !important;}
#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current:hover {color:#696a52;background:#d4d6ba}
#ds-thread #ds-reset a.ds-highlight:hover {color:#696a52 !important;}

#ds-thread {padding-left:30px;}
#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset #ds-hot-posts {overflow:visible}
#ds-thread #ds-reset .ds-post-self {padding:10px 0 10px 10px;}
#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset .ds-post-self {border:0 !important;}
#ds-reset .ds-avatar, #ds-thread #ds-reset ul.ds-children .ds-avatar {position:absolute;top:26px;left:-14px;padding:5px;width:36px;height:36px;box-shadow:-1px 0 1px rgba(0,0,0,.15) inset;border-radius:46px; background:#E5E6D0;}
#ds-thread #ds-reset ul.ds-children .ds-avatar {left:-23px;}
#ds-thread .ds-avatar a {display:inline-block;padding:1px; width:32px;height:32px;border:1px solid #b9baa6;border-radius:50%; background-color:#fff !important}
#ds-thread .ds-avatar a:hover {border-color:#de5a4e}
#ds-thread .ds-avatar > img {margin:2px 0 0 2px}
#ds-thread #ds-reset .ds-replybox {box-shadow:none;}
#ds-thread #ds-reset ul.ds-children .ds-replybox.ds-inline-replybox a.ds-avatar,
#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar {left: 0;top: 0; padding: 0;width: 32px !important;height: 32px !important; background: none;box-shadow: none; } 
#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar img {width: 32px !important;height: 32px !important; border-radius:50%;} 
#ds-reset .ds-replybox a.ds-avatar,
#ds-reset .ds-replybox .ds-avatar img { padding:0;width:50px !important;height:50px !important; border-radius:5px; }
#ds-reset .ds-avatar img {width:32px !important;height:32px !important;border-radius:32px;box-shadow:0 1px 3px rgba(0, 0, 0, 0.22);
							-webkit-transition:.4s all ease-in-out;-moz-transition:.4s all ease-in-out;-o-transition:.4s all ease-in-out;-ms-transition:.4s all ease-in-out;transition:.4s all ease-in-out;
							}
.ds-post-self:hover .ds-avatar img {-webkit-transform:rotate(360deg);-moz-transform:rotate(360deg);-o-transform:rotate(360deg);-ms-transform:rotate(360deg);transform:rotate(360deg);}

#ds-thread #ds-reset .ds-comment-body {background: #F2F2F2;padding:15px 15px 12px 32px;border-radius:5px; box-shadow:0 1px 2px rgba(0,0,0,.15), 0 1px 0 rgba(255,255,255,.75) inset;}

#ds-thread #ds-reset .ds-comment-body p{color:#787D7B}
#ds-thread #ds-reset .ds-comments a.ds-user-name {font-weight:bold;color:#696A52 !important;}
#ds-thread #ds-reset .ds-comments a.ds-user-name:hover {color:#D32 !important;}

#ds-thread #ds-reset #ds-hot-posts {border:0}
#ds-reset #ds-hot-posts .ds-gradient-bg {background:none;}

#ds-reset #ds-bubble #ds-ctx .ds-ctx-entry {padding:0;}
#ds-reset #ds-bubble .ds-avatar, #ds-reset #ds-bubble #ds-ctx-bubble .ds-avatar a {position:static;padding:0;border:0; background:none;box-shadow:none;}
#ds-reset #ds-bubble .ds-avatar img, #ds-reset #ds-bubble #ds-ctx-bubble .ds-avatar a {width:45px !important;height:45px !important;}
#ds-reset #ds-bubble .ds-user-name{padding-left:13px;}

#ds-reset .ds-comment-body #ds-ctx {border-left:1px solid #b9baa6;background-color:#e8e8dc !important}
#ds-reset #ds-ctx {margin-right:-15px}
#ds-reset #ds-ctx .ds-ctx-entry {position:relative;padding:10px 30px 10px 10px;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-avatar {top:6px;left:5px;background:none;box-shadow:none;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-body {margin-left:46px;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-content {color:#787968}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-head a {color:#696A52;font-weight:bold}

```
