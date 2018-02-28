---
layout: post
title:  "使用Shadowsocks搭建VPS"
date:   2018-02-28 16:10:00.000000000 +09:00
tags: [Others]
---
首先是选择云端服务器平台，[知乎]上有各个VPS平台的对比，这里个人推荐用[Digitalocean]和[Vutrl]。本文选用的是Vutrl。    
首先是在平台上注册账号，并绑定信用卡或者Paypal账号。（注：绑定完信用卡或者Paypal账号才能创建虚拟服务器）。    
+ 创建云服务器    
1.选择服务器的区域
[image-01]
至于选择哪个区域比较快，可以在Vutrl提供的[服务器测速地址]来测试一下。
[image-02]
点击各个测试地址可以看到具体的ip地址
[image-03]
可以在本地的终端ping一下，测试下本地网络是否能够访问该区域的服务器。
[image-04]
本文测试了一下ping纽约节点的服务器，丢包率6.7%，访问延时200毫秒。可以根据这个来判断哪个节点回更快，一般东京、新加坡的节点会比较快一点。    
2.选择服务器系统：Ubuntu 17.10 x64
[image-05]
3.选择服务器大小：最低配置的$5/mo就够了
[image-06]
4.其他的设置，hostname根据个人随意设置
[image-07]
5.点击Deploy Now创建服务器。当Status显示Running的时候，就证明服务器已经创建成功并启动
[image-08]
+ 修改Root密码     
点击服务器进入查看服务器信息
[image-09]
点击右上角进入console窗口（会弹出一个新窗口），注意远程端口出现的时间可能会需要一段时间，如果等了很久都没出现，就关闭重试一次
输入root，然后Enter，输入root密码（在服务器信息中查找）。ps:输入的时候是不会出现输入结果的，直接按顺序准确输入就行。
成功登录后，输入

[知乎]:https://www.zhihu.com/question/20800554/answer/71397836
[Digitalocean]:https://www.digitalocean.com/
[Vutrl]:https://www.vultr.com/
[服务器测速地址]:https://www.vultr.com/faq/

[image-01]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS01.png
[image-02]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS02.png
[image-03]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS03.png
[image-04]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS04.png
[image-05]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS05.png
[image-06]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS06.png
[image-07]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS07.png
[image-08]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS08.png
[image-09]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS09.png
[image-10]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS10.png
[image-11]:     http://blog.wangjace.site/image/2018-02-28-使用Shadowsocks搭建VPS11.png


