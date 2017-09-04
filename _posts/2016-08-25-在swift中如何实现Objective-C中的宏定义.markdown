---
layout: post
title:  "在swift中如何实现Objective-C中的宏定义"
date:   2016-08-25 23:40:23.000000000 +09:00
tags: [swift 学习笔记]
---    
在Objective-C开发中，经常会使用到宏定义来定义一些全局变量和一些简单的全局方法。但在swift中并不支持宏定义语法，但可以利用swift项目共享命名空间这一特点，在项目中使用Const.swift来定义这些全局变量和公用方法

1.不带参数的宏定义    
{% highlight Objective-C %}
//Objective-C写法
#define kScreenWidth CGRectGetWidth([UIScreen mainScreen].bounds)
#define kScreenHeight CGRectGetHeight([UIScreen mainScreen].bounds)
{% endhighlight %}
{% highlight Swift %}
//swif写法
let kScreenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)
let kScreenHeight = CGRectGetHeight(UIScreen.mainScreen().bounds)
{% endhighlight %}
2.带参数的宏定义    
{% highlight Objective-C %}
//Objective-C写法
#define RGBCOLOR(r,g,b,a) [UIColor colorWithRed:(r)/255.0 green:(g)/255.0 blue:(b)/255.0 alpha:a]
{% endhighlight %}
{% highlight Swift %}
//swift写法
func RGBCOLOR(r:CGFloat,_ g:CGFloat, _ b:CGFloat, _ a:CGFloat) -> UIColor
{
    return UIColor(red: (r)/255.0, green: (g)/255.0, blue: (b)/255.0, alpha: a)
}
{% endhighlight %}

