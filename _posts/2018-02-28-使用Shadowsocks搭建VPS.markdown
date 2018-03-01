---
layout: post
title:  "使用Shadowsocks搭建VPS"
date:   2018-02-28 16:10:00.000000000 +09:00
tags: [Others]
---
首先是选择云端服务器平台，[知乎]上有各个VPS平台的对比，这里个人推荐用[Digitalocean]和[Vutrl]。本文选用的是Vutrl。    
首先是在平台上注册账号，并绑定信用卡或者Paypal账号。（注：绑定完信用卡或者Paypal账号才能创建虚拟服务器）。    
**• 创建云服务器**    
1.选择服务器的区域
![image-01]
至于选择哪个区域比较快，可以在Vutrl提供的[服务器测速地址]来测试一下。
![image-02]
点击各个测试地址可以看到具体的ip地址
![image-03]
可以在本地的终端ping一下，测试下本地网络是否能够访问该区域的服务器。
![image-04]
本文测试了一下ping纽约节点的服务器，丢包率6.7%，访问延时200毫秒。可以根据这个来判断哪个节点回更快，一般东京、新加坡的节点会比较快一点。    
2.选择服务器系统：Ubuntu 17.10 x64
![image-05]
3.选择服务器大小：最低配置的$5/mo就够了
![image-06]
4.其他的设置，hostname根据个人随意设置
![image-07]
5.点击Deploy Now创建服务器。当Status显示Running的时候，就证明服务器已经创建成功并启动
![image-08]
**• 修改Root密码**     
点击服务器进入查看服务器信息
![image-09]
点击右上角进入console窗口（会弹出一个新窗口），注意远程端口出现的时间可能会需要一段时间，如果等了很久都没出现，就关闭重试一次
![image-10]
输入root，然后Enter，输入root密码（在服务器信息中查找）。ps:输入的时候是不会出现输入结果的，直接按顺序准确输入就行。成功登录后，输入 *passwd root* 修改root密码，密码需输入两次进行确认。
![image-11]
**• 安装Shadowsocks server端**    
使用root用户登录，运行以下命令：
{% highlight shell %}
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
{% endhighlight %}
安装完成后，需要设置服务器ip、端口(8989)、密码、加密方式（rc4-md5），脚本提示如下：
{% highlight shell %}
Congratulations, Shadowsocks-python server install completed!
Your Server IP        :your_server_ip
Your Server Port      :your_server_port
Your Password         :your_password
Your Encryption Method:your_encryption_method
Welcome to visit:https://teddysun.com/342.html
Enjoy it!
{% endhighlight %}
卸载方法：
使用root用户登录，运行以下命令：
{% highlight shell %}
./shadowsocks.sh uninstall
{% endhighlight %}
配置文件：/etc/shadowsocks.json，运行以下命令编辑配置文件：
{% highlight shell %}
vi /etc/shadowsocks.json
{% endhighlight %}
配置文件使用vim编辑器进行编辑，关于vim如何使用，请[移步]    
单用户配置文件示例：
{% highlight shell %}
{
    "server":"0.0.0.0",
    "server_port":your_server_port,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"your_password",
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
{% endhighlight %}
多用户配置文件示例：
{% highlight shell %}
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
{% endhighlight %}
**• Shadowsocks的使用命令**    
启动：/etc/init.d/shadowsocks start    
停止：/etc/init.d/shadowsocks stop    
重启：/etc/init.d/shadowsocks restart    
状态：/etc/init.d/shadowsocks status    
**• 使用Shadowsocks客户端**    
Shadowsocks的客户端支持各大主流平台，而且客户端的配置一般都很简单，只需要配置一下服务器的ip地址和之前设置好的连接密码，加密方式即可。Shadowsocks各个平台的客户端下载链接如下：    
+ [OS X]
+ [Windows]
+ [Android]
+ [iOS]
+ [Others]    

参考：[Shadowsocks Python版一键安装脚本] 

[知乎]:https://www.zhihu.com/question/20800554/answer/71397836
[Digitalocean]:https://www.digitalocean.com/
[Vutrl]:https://www.vultr.com/
[服务器测速地址]:https://www.vultr.com/faq/
[移步]:https://www.vpser.net/manage/vi.html
[Shadowsocks Python版一键安装脚本]:https://teddysun.com/342.html
[OS X]:https://github.com/shadowsocks/ShadowsocksX-NG/releases
[Windows]:https://github.com/shadowsocks/shadowsocks-windows/releases
[Android]:https://github.com/shadowsocks/shadowsocks-android/releases
[iOS]:https://github.com/shadowsocks/shadowsocks-iOS/wiki/Help
[Others]:https://github.com/shadowsocks/shadowsocks/wiki/Ports-and-Clients
[image-01]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS01.png
[image-02]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS02.png
[image-03]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS03.png
[image-04]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS04.png
[image-05]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS05.png
[image-06]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS06.png
[image-07]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS07.png
[image-08]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS08.png
[image-09]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS09.png
[image-10]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS10.png
[image-11]:     http://blog.wangjace.site/images/article_images/2018-02-28-使用Shadowsocks搭建VPS11.png


