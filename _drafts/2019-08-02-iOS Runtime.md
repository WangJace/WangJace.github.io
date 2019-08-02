---
layout: post
title:  "iOS Runtime"
tags: [iOS 笔记]
---
## 关于Runtime
> Objective-C 扩展了 C 语言，并加入了面向对象特性和 Smalltalk 式的消息传递机制。而这个扩展的核心是一个用 C 和 编译语言 写的 Runtime 库。它是 Objective-C 面向对象和动态机制的基石。  

> Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。理解 Objective-C 的 Runtime 机制可以帮我们更好的了解这个语言，适当的时候还能对语言进行扩展，从系统层面解决项目中的一些设计或技术问题。了解 Runtime ，要先了解它的核心 - 消息传递 （Messaging）。  

Runtime其实有两个版本: “modern” 和 “legacy”。我们现在用的 Objective-C 2.0 采用的是现行 (Modern) 版的 Runtime 系统，只能运行在 iOS 和 macOS 10.5 之后的 64 位程序中。而 macOS 较老的32位程序仍采用 Objective-C 1 中的（早期）Legacy 版本的 Runtime 系统。这两个版本最大的区别在于当你更改一个类的实例变量的布局时，在早期版本中你需要重新编译它的子类，而现行版就不需要。  

Runtime 基本是用 C 和汇编写的，可见苹果为了动态系统的高效而作出的努力。你可以在[这里](https://link.juejin.im/?target=http%3A%2F%2Fwww.opensource.apple.com%2Fsource%2Fobjc4%2F)下到苹果维护的开源代码。苹果和GNU各自维护一个开源的 runtime 版本，这两个版本之间都在努力的保持一致。  

平时的业务中主要是使用[官方Api](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.apple.com%2Freference%2Fobjectivec%2Fobjective_c_runtime%23%2F%2Fapple_ref%2Fdoc%2Fuid%2FTP40001418-CH1g-126286)，解决我们框架性的需求。  

高级编程语言想要成为可执行文件需要先编译为汇编语言再汇编为机器语言，机器语言也是计算机能够识别的唯一语言，但是OC并不能直接编译为汇编语言，而是要先转写为纯C语言再进行编译和汇编的操作，从OC到C语言的过渡就是由runtime来实现的。然而我们使用OC进行面向对象开发，而C语言更多的是面向过程开发，这就需要将面向对象的类转变为面向过程的结构体。  
## Runtime消息传递
一个对象方法的调用如```[obj walk]```，编译器会编译成消息发送```objc_msgSend(obj, foo)```，Runtime时执行的流程如下：  

1. 先判断 *obj* 是否为 *nil*，如果是的话则直接返回，否则进入下一步  

2. 通过 *obj* 的isa指针找到它的 *class*  

3. 从 *class* 的缓存 *objc_cache* 中寻找是否有 *foo* 这个函数指针，如果有执行此函数，如果没有则进入下一步  

4. 从 *class* 的函数列表 *methodLists* 里寻找是否有 *foo* 这个函数指针，如果有执行此函数，如果没有则进入下一步  

5. 从父类 *super_class* 的缓存 *objc_cache* 中寻找是否有 *foo* 这个函数指针，如果有执行此函数，如果没有则进入下一步  

6. 从父类的函数列表 *methodLists* 里寻找是否有 *foo* 这个函数指针，如果有执行此函数，如果没有且再往上还有父类，则回到上一步，直至没有父类即到达根类。如果在根类仍然没有找到 *foo* 函数，则进入下一步  

7. 查看是否有进行函数的动态解析（即是否实现了```resolveInstanceMethod:```或```resolveClassMethod:```，这两个的区别在于一个是实例方法的动态解析，一个是类方法的动态解析），如果有，重新进行类和父类的函数查询，如果没有，则进入下一步  

8. 查看是否有进行消息转发（即是否实现了```forwardingTargetForSelector:```或通过实现```methodSignatureForSelector:```进行方法注册，然后在```forwardInvocation:```方法中进行方法的实现或者转发），如果没有进行消息转发，则直接报错：*"...: unrecognized selector sent to instance ......"*  
![流程图](/assets/article_images/2019-08-02-Objective-C方法调用内部原理.png)

这个过程中的函数缓存是将经常调用的函数放到 *objc_cache* 中，提高函数查询的效率。
