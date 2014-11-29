---
layout: post
title: "GithubPage页面国内打开缓慢的问题"
date: 2014-11-29 12:23:23 +0800
comments: true
categories: web
---
　　其实早在两年前，我在[博客园](http://www.cnblogs.com/showmylym "博客园")创建过个人博客，零零散散写了些很业余的东西，没有系统性地写，与编程也没有太大关系，没能坚持。在最近的工作中，有了分享意识，并且发现一个知识点如果能够给别人讲懂，知识点会融入进自己的知识体系，理解程度会有质的变化。因此决定重新做一个专业的博客，一来给自己总结，二来可能会帮助到其他人。虽然国内技术博客站不少，但大多都掺杂太多广告，不够纯粹，考虑到平时频繁用到github找代码，遂决定在github上建博客，与自己的开源代码结合。这次一定可以坚持下去:)  
　　不过一上来就碰到国内打开博客页面慢的问题，且翻墙时灵时不灵。跟了一下页面加载的过程，发现首页有四个链接加载超时，如下图1-1。  
![图1-1](/images/1/1-1.png "图1-1")  
图1-1  
　　Copy出来，四个URL如下：  
　　`https://fonts.googleapis.com/css?family=Open+Sans:400,700`  
　　`http://platform.twitter.com/widgets.js`  
　　`http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js`  
　　`https://fonts.googleapis.com/css?family=Noto+Serif:400,700`  
　　首先关掉twitter账户关联。按照[HTML页面的加载过程](http://wenku.baidu.com/view/8a146ba2d0d233d4b14e69ae.html "HTML页面的加载过程")，浏览器首先解析页面结构，再加载外部CSS和script。如果在`<head>`标签里碰到`<script>`，则解析页面结构时就会加载脚本。此时若网络延迟较大，会出现长时间的白屏。猜测首页打开缓慢应当是这个原因，因此参考[不同javascript加载方式对页面性能的影响](http://blog.csdn.net/chang_yuan_2011/article/details/7881606 "不同javascript加载方式对页面性能的影响")，首先尝试将script放在body结尾前执行。  
　　打开octopress博客源代码的source目录，使用grep命令搜索内容中带有"script"的文件(i为忽略大小写，r为递归查找，l为以文件列表形式显示)。  
　　`grep -irl script | grep htm`  
　　搜索结果如下图1-2。  
![图1-2](/images/1/1-2.png "图1-2")  
图1-2  
　　文件有点多，不好找。考虑过后，决定直接搜索图1-1中标红的那几个URL。搜索命令分别为：  
　　`grep -irl jquery | grep htm`   
　　`grep -irl https://fonts.googleapis.com/css?family=Noto+Serif:400,700 | grep htm`   
　　`grep -irl https://fonts.googleapis.com/css?family=Open+Sans:400,700 | grep htm`   
　　根据结果了解到，调用这三条URL的执行语句，在_includes/head.html和_includes/custom/head.html中，对其做如下图1-3和图1-4的修改。  
![图1-3](/images/1/1-3.png "图1-3")  
图1-3  
![图1-4](/images/1/1-4.png "图1-4")  
图1-4  
　　修改完毕刷新页面。页面显示很快，但在Chrome的网络控制台上，显示还有4个URL访问缓慢，如下图1-5。  
![图1-5](/images/1/1-5.png "图1-5")  
图1-5  
　　继续搜索关键字"gstatic"  
　　`grep -irl gstatic`  
　　搜索结果如下图1-6  
![图1-6](/images/1/1-6.png "图1-6")  
图1-6  
　　打开这两个刚刚创建的css样式文件，很明显可以看到其引用了不少gstatic.com上存储的woff2格式的web开放字体文件。目前还没发觉其对页面显示有何影响，花点时间将它们定位到本地吧。  
　　哈哈，问题到此已解决，打开首页显示速度快多了:)  
