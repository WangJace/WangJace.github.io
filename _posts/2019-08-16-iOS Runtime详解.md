---
layout: post
title:  "iOS Runtime详解"
tags: [iOS 笔记]
---
### 关于Runtime
> Objective-C 扩展了 C 语言，并加入了面向对象特性和 Smalltalk 式的消息传递机制。而这个扩展的核心是一个用 C 和 编译语言 写的 Runtime 库。它是 Objective-C 面向对象和动态机制的基石。  

> Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。理解 Objective-C 的 Runtime 机制可以帮我们更好的了解这个语言，适当的时候还能对语言进行扩展，从系统层面解决项目中的一些设计或技术问题。了解 Runtime ，要先了解它的核心 - 消息传递 （Messaging）。  

Runtime其实有两个版本: “modern” 和 “legacy”。我们现在用的 Objective-C 2.0 采用的是现行 (Modern) 版的 Runtime 系统，只能运行在 iOS 和 macOS 10.5 之后的 64 位程序中。而 macOS 较老的32位程序仍采用 Objective-C 1 中的（早期）Legacy 版本的 Runtime 系统。这两个版本最大的区别在于当你更改一个类的实例变量的布局时，在早期版本中你需要重新编译它的子类，而现行版就不需要。  

Runtime 基本是用 C 和汇编写的，可见苹果为了动态系统的高效而作出的努力。你可以在[这里](https://link.juejin.im/?target=http%3A%2F%2Fwww.opensource.apple.com%2Fsource%2Fobjc4%2F)下到苹果维护的开源代码。苹果和GNU各自维护一个开源的 runtime 版本，这两个版本之间都在努力的保持一致。  

平时的业务中主要是使用[官方API](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.apple.com%2Freference%2Fobjectivec%2Fobjective_c_runtime%23%2F%2Fapple_ref%2Fdoc%2Fuid%2FTP40001418-CH1g-126286)，解决我们框架性的需求。  

高级编程语言想要成为可执行文件需要先编译为汇编语言再汇编为机器语言，机器语言也是计算机能够识别的唯一语言，但是OC并不能直接编译为汇编语言，而是要先转写为纯C语言再进行编译和汇编的操作，从OC到C语言的过渡就是由runtime来实现的。然而我们使用OC进行面向对象开发，而C语言更多的是面向过程开发，这就需要将面向对象的类转变为面向过程的结构体。  
### Runtime消息传递
一个对象方法的调用如```[obj walk]```，编译器会编译成消息发送```objc_msgSend(obj, walk)```
objec_msgSend的方法定义如下：

```C
OBJC_EXPORT id objc_msgSend(id self, SEL op, ...)
```
要理解消息传递是怎么实现的，先看看对象(object)，类(class)，方法(method)的结构体：

```C
//对象
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
//类
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;
//方法列表
struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;
    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
//方法
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
```
消息传递的实现过程（即Runtime时执行的流程）：  
1. 先判断 *obj* 是否为 *nil*，如果是的话则直接返回，否则进入下一步  
2. 通过 *obj* 的isa指针找到它的 *class*  
3. 从 *class* 的缓存 *objc_cache* 中寻找是否有 *walk* 这个方法指针，如果有执行此方法的 *IMP*，转发 *IMP* 的 *return* 值，如果没有则进入下一步  
4. 从 *class* 的方法列表 *methodLists* 里寻找是否有 *walk* 这个方法指针，如果有执行此方法的 *IMP*，转发 *IMP* 的 *return* 值，如果没有则进入下一步  
5. 从父类 *super_class* 的缓存 *objc_cache* 中寻找是否有 *walk* 这个方法指针，如果有执行此方法的 *IMP*，转发 *IMP* 的 *return* 值，如果没有则进入下一步  
6. 从父类的方法列表 *methodLists* 里寻找是否有 *walk* 这个方法指针，如果有执行此方法的 *IMP*，转发 *IMP* 的 *return* 值，如果没有且再往上还有父类，则回到上一步，直至没有父类即到达根类。如果在根类仍然没有找到 *walk* 方法，则进入下一步  
7. 查看是否有进行方法的动态解析（即是否实现了```resolveInstanceMethod:```或```resolveClassMethod:```，这两个的区别在于一个是实例方法的动态解析，一个是类方法的动态解析），如果有，重新进行类和父类的方法查询，如果没有，则进入下一步  
8. 查看是否有进行消息转发（即是否实现了```forwardingTargetForSelector:```或通过实现```methodSignatureForSelector:```进行方法注册，然后在```forwardInvocation:```方法中进行方法的实现或者转发），如果没有进行消息转发，则直接报错：*"...: unrecognized selector sent to instance ......"*  
![流程图](/assets/article_images/2019-08-02-Objective-C方法调用内部原理.png)

这个过程中的方法缓存是将之前调用过的方法放到 *objc_cache* 中，提高方法查询的效率。在找到 *walk* 方法后，把 *walk* 的method_name 作为 *key*，*method_imp* 作为 *value* 给存起来。当再次收到 *walk* 消息的时候，可以直接在 *cache* 里找到，避免去遍历*objc_method_list*。从前面的源代码可以看到 *objc_cache* 是存在 *objc_class* 结构体中的。  
### 消息传递中的相关结构体
#### 1. 类对象(objc_class)  
在Objective-C中的类(Class)实际上是一个指向 *objc_class* 结构体的指针

```C
typedef struct objc_class *Class;
```
*objc/runtime.h* 中 *objc_class* 结构体的定义如下：

```C
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```
*struct objc_class* 结构体里定义了指向父类的指针、类的名字、版本、实例大小、实例变量列表、方法列表、缓存、遵守的协议列表等。类对象就是一个结构体 *struct objc_class*，这个结构体存放的数据称为元数据( *metadata* )，该结构体的第一个成员变量也是 *isa* 指针，这就说明了 *Class* 本身其实也是一个对象，因此我们称之为类对象，类对象在编译期产生用于创建实例对象，是单例。  
#### 2. 实例(objc_object)

```C
/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
类对象中的元数据存储的都是如何创建一个实例的相关信息，那么类对象和类方法应该从哪里创建呢？
就是从 *isa* 指针指向的结构体创建，类对象的 *isa* 指针指向的我们称之为元类( *metaclass* )，元类中保存了创建类对象以及类方法所需的所有信息，因此整个结构应该如下图所示:  
![实例-类-元类之间的结构关系](/assets/article_images/2019-08-02-实例-类-元类之间的结构关系.png)
#### 3.元类(Meta Class)
通过上面的结构图，可以看出 实例（*struct objc_object* 结构体）中的isa指针指向类对象，类对象 的 *isa* 指针指向的是元类，*super_class* 指针指向的是父类的类对象。而元类的 *super_class* 指针指向了父类的元类，那元类的 *isa* 指针又指向了自己。 
 
元类(Meyta Class)是一个类对象的类。在上面我们提到，所有的类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。为了调用类方法，这个类的isa指针必须指向一个包含这些类方法的一个objc_class结构体。这就引出了meta-class的概念，元类中保存了创建类对象以及类方法所需的所有信息。任何NSObject继承体系下的meta-class都使用NSObject的meta-class作为自己的所属类，而基类的meta-class的isa指针是指向它自己。
#### 4.Method(objc_method)
```C
runtime.h
/// An opaque type that represents a method in a class definition.代表类定义中一个方法的不透明类型
typedef struct objc_method *Method;
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
};
```
*Method* 就是表示能够独立完成一个功能的一段代码，如：

```Objective-C
- (void)walk
{
	NSLog(@"walk");
}
```
*objc_method* 结构体的内容：  
*SEL method_name* 方法名  
*char \*method_types* 方法类型  
*IMP method_imp* 方法实现  
在结构体中出现了 *SEL* 和 *IMP*，说明 *SEL* 和 *IMP* 是 *Method*的属性。
#### 5.SEL(objc_selector)
```C
Objc.h
/// An opaque type that represents a method selector.代表一个方法的不透明类型
typedef struct objc_selector *SEL;
```
*objc_msgSend* 方法的第二个参数类型为 *SEL*，它是 *selector* 在 *Objective-C* 中的表示类型（*Swift* 中是 *Selector* 类）。*selector* 是方法选择器，可以理解为区分方法的 *ID*，而这个 *ID* 的数据结构是SEL:

```Objective-C
@property SEL selector;
```
其实 *selector* 是 *SEL* 的一个实例。

```
A method selector is a C string that has been registered (or “mapped“) with the Objective-C runtime. Selectors generated by the compiler are automatically mapped by the runtime when the class is loaded.
```
*selector* 就是个映射到方法的 *C* 字符串，你可以用 *Objective-C* 编译器命令 *@selector()* 或者 *Runtime* 系统的 *sel_registerName* 函数来获得一个 *SEL* 类型的方法选择器。  
*selector* 既然是一个 *string*，我觉得应该是类似 *className+method* 的组合，命名规则有两条：  
+ 同一个类，*selector* 不能重复  
+ 不同的类，*selector* 可以重复  

这也带来了一个弊端，我们在写 *C* 代码的时候，经常会用到函数重载，就是函数名相同，参数不同，但是这在 *Objective-C* 中是行不通的，因为 *selector* 只记了 *method* 的 *name*，没有参数，所以没法区分不同的 *method*。
#### 6.IMP
```C
/// A pointer to the function of a method implementation.  指向一个方法实现的指针
typedef id (*IMP)(id, SEL, ...); 
```
实际上就是一个指向最终实现程序的内存地址的指针。  
在 *iOS* 的 *Runtime* 中，*Method* 通过 *selector* 和 *IMP* 两个属性，实现了快速查询方法及实现，相对提高了性能，又保持了灵活性。
#### 7.类缓存(objc_cache)
当 *Objective-C* 运行时通过跟踪它的 *isa* 指针检查对象时，它可以找到一个实现许多方法的对象。然而，你可能只调用它们的一小部分，并且每次查找时都去遍历搜索 *methodLists*效率比较低。所以类实现一个缓存，每当你遍历搜索一次 *methodLists*，并找到相应的方法时，会将该方法放入缓存中。所以当 *objc_msgSend* 查找一个方法时，它首先搜索缓存。这是基于这样的理论：如果你在类上调用一个消息，你可能以后会再次调用该消息。  
为了加速消息分发， 系统会对方法和对应的地址进行缓存，就放在上述的 *objc_cache*，所以在实际运行中，大部分常用的方法都是会被缓存起来的，*Runtime* 系统实际上非常快，接近直接执行内存地址的程序速度。
#### 8.Category(objc_category)
Category是表示一个指向分类的结构体的指针，其定义如下：

```C
/*
name：是指 class_name 而不是 category_name。
cls：要扩展的类对象，编译期间是不会定义的，而是在Runtime阶段通过name对应到对应的类对象。
instanceMethods：category中所有给类添加的实例方法的列表。
classMethods：category中所有添加的类方法的列表。
protocols：category实现的所有协议的列表。
instanceProperties：表示Category里所有的properties，这就是我们可以通过objc_setAssociatedObject和objc_getAssociatedObject增加实例变量的原因，不过这个和一般的实例变量是不一样的。
*/
struct category_t { 
    const char *name; 
    classref_t cls; 
    struct method_list_t *instanceMethods; 
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
};
```
从上面的 *category_t* 的结构体中可以看出，分类中可以添加实例方法，类方法，甚至可以实现协议，添加属性，不可以添加成员变量。
### Runtime应用
*Runtime* 简直就是做大型框架的利器。它的应用场景非常多，下面就介绍一些常见的应用场景。  
+ 关联对象(Objective-C Associated Objects)给分类增加属性  
+ 方法魔法(Method Swizzling)方法添加和替换和KVO实现  
+ 消息转发(热更新)解决Bug(JSPatch)  
+ 实现NSCoding的自动归档和自动解档  
+ 实现字典和模型的自动转换(MJExtension)

#### 1.关联对象(Objective-C Associated Objects)给分类增加属性
我们都是知道分类是不能自定义属性和变量的。下面通过关联对象实现给分类添加属性。  
关联对象Runtime提供了下面几个接口：

```C
//关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
//获取关联的对象
id objc_getAssociatedObject(id object, const void *key)
//移除关联的对象
void objc_removeAssociatedObjects(id object)
```
参考解释

```C
id object：被关联的对象
const void *key：关联的key，要求唯一
id value：关联的对象
objc_AssociationPolicy policy：内存管理的策略
```
内存管理的策略

```C
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```
下面实现一个 *UIView* 的 *Category* 添加自定义属性 *defaultColor*

```Objective-C
#import "ViewController.h"
#import "objc/runtime.h"

@interface UIView (DefaultColor)

@property (nonatomic, strong) UIColor *defaultColor;

@end

@implementation UIView (DefaultColor)

@dynamic defaultColor;

static char kDefaultColorKey;

- (void)setDefaultColor:(UIColor *)defaultColor {
    objc_setAssociatedObject(self, &kDefaultColorKey, defaultColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)defaultColor {
    return objc_getAssociatedObject(self, &kDefaultColorKey);
}

@end

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    UIView *test = [UIView new];
    test.defaultColor = [UIColor blackColor];
    NSLog(@"%@", test.defaultColor);
}

@end
```
```
打印结果： 2018-04-01 15:41:44.977732+0800 ocram[2053:63739] UIExtendedGrayColorSpace 0 1
```
打印结果来看，我们成功在分类上添加了一个属性，实现了它的setter和getter方法。  
通过关联对象实现的属性的内存管理也是有ARC管理的，所以我们只需要给定适当的内存策略就行了，不需要操心对象的释放。  
我们看看内存测量对于的属性修饰

|内存策略|属性修饰|描述|
|-|-|-|
|OBJC_ASSOCIATION_ASSIGN|@property (assign) 或 @property (unsafe_unretained)|指定一个关联对象的弱引用。|
|OBJC_ASSOCIATION_RETAIN_NONATOMIC|@property (nonatomic, strong)|@property (nonatomic, strong)	指定一个关联对象的强引用，不能被原子化使用。|
|OBJC_ASSOCIATION_ASSIGN|@property (assign) 或 @property (unsafe_unretained)|指定一个关联对象的弱引用。|
|OBJC_ASSOCIATION_RETAIN_NONATOMIC|@property (nonatomic, strong)|@property (nonatomic, strong)	指定一个关联对象的强引用，不能被原子化使用。|
|OBJC_ASSOCIATION_COPY_NONATOMIC|@property (nonatomic, copy)|指定一个关联对象的copy引用，不能被原子化使用。|
|OBJC_ASSOCIATION_RETAIN|@property (atomic, strong)|指定一个关联对象的强引用，能被原子化使用。|
|OBJC_ASSOCIATION_COPY|@property (atomic, copy)|指定一个关联对象的copy引用，能被原子化使用。|
|-|-|-|

#### 2.方法魔法(Method Swizzling)方法添加和替换和KVO实现
##### • 方法添加
```C
//class_addMethod(Class  _Nullable __unsafe_unretained cls, SEL  _Nonnull name, IMP  _Nonnull imp, const char * _Nullable types)
class_addMethod([self class], sel, (IMP)fooMethod, "v@:");
```
+ cls 被添加方法的类
+ name 添加的方法的名称的SEL
+ imp 方法的实现。该函数必须至少要有两个参数：self，_cmd
+ 类型编码

##### • 方法替换
下面实现一个替换ViewController的viewDidLoad方法的例子。

```Objective-C
@implementation ViewController

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        SEL originalSelector = @selector(viewDidLoad);
        SEL swizzledSelector = @selector(jkviewDidLoad);
        
        Method originalMethod = class_getInstanceMethod(class,originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class,swizzledSelector);
        
        //judge the method named  swizzledMethod is already existed.
        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
        // if swizzledMethod is already existed.
        if (didAddMethod) {
            class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        }
        else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (void)jkviewDidLoad {
    NSLog(@"替换的方法");
    
    [self jkviewDidLoad];
}

- (void)viewDidLoad {
    NSLog(@"自带的方法");
    
    [super viewDidLoad];
}

@end
```	
*swizzling* 应该只在 *+load* 中完成。 在 *Objective-C* 的运行时中，每个类有两个方法都会自动调用。*+load* 是在一个类被初始装载时调用，*+initialize* 是在应用第一次调用该类的类方法或实例方法前调用的。两个方法都是可选的，并且只有在方法被实现的情况下才会被调用。

*swizzling* 应该只在 *dispatch_once* 中完成,由于 *swizzling* 改变了全局的状态，所以我们需要确保每个预防措施在运行时都是可用的。原子操作就是这样一个用于确保代码只会被执行一次的预防措施，就算是在不同的线程中也能确保代码只执行一次。*Grand Central Dispatch* 的 *dispatch_once* 满足了所需要的需求，并且应该被当做使用 *swizzling* 的初始化单例方法的标准。  
实现图解如下图。
![方法交换](/assets/article_images/2019-08-02-方法替换.png)
> 从图中可以看出，我们通过swizzling特性，将selectorC的方法实现IMPc与selectorN的方法实现IMPn交换了，当我们调用selectorC，也就是给对象发送selectorC消息时，所查找到的对应的方法实现就是IMPn而不是IMPc了。

#### 3.KVO实现
> 全称是Key-value observing，翻译成键值观察。提供了一种当其它对象属性被修改的时候能通知当前对象的机制。再MVC大行其道的Cocoa中，KVO机制很适合实现model和controller类之间的通讯。  

*KVO* 的实现依赖于 *Objective-C* 强大的 *Runtime*，当观察某对象 *A* 时，*KVO* 机制动态创建一个对象 *A* 当前类的子类，并为这个新的子类重写了被观察属性 *keyPath* 的 *setter* 方法。*setter* 方法随后负责通知观察对象属性的改变状况。

*Apple* 使用了 *isa-swizzling* 来实现 *KVO* 。当观察对象A时，*KVO* 机制动态创建一个新的名为：*NSKVONotifying_A* 的新类，该类继承自对象 *A* 的本类，且 *KVO* 为 *NSKVONotifying_A* 重写观察属性的 *setter* 方法，*setter* 方法会负责在调用原 *setter* 方法之前和之后，通知所有观察对象属性值的更改情况。
##### • NSKVONotifying_A 类剖析
```Objective-C
NSLog(@"self->isa:%@",self->isa);  
NSLog(@"self class:%@",[self class]); 
```
在建立KVO监听前，打印结果为：

```Objective-C
self->isa:NSKVONotifying_A
self class:A
```
在这个过程，被观察对象的 isa 指针从指向原来的 *A* 类，被 *KVO* 机制修改为指向系统新创建的子类 *NSKVONotifying_A* 类，来实现当前类属性值改变的监听；
所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 *KVO* 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为“*NSKVONotifying_A*”的类，就会发现系统运行到注册 *KVO* 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 *NSKVONotifying_A* 的中间类，并指向这个中间类了。
##### • 子类setter方法剖析
*KVO* 的键值观察通知依赖于 *NSObject* 的两个方法: *willChangeValueForKey:* 和 *didChangeValueForKey:* ，在存取数值的前后分别调用 2 个方法：
被观察属性发生改变之前，*willChangeValueForKey:* 被调用，通知系统该 *keyPath* 的属性值即将变更；当改变发生后， *didChangeValueForKey:* 被调用，通知系统该 *keyPath* 的属性值已经变更；之后， *observeValueForKey:ofObject:change:context:* 也会被调用。且重写观察属性的 *setter* 方法这种继承方式的注入是在运行时而不是编译时实现的。

KVO 为子类的观察者属性重写调用存取方法的工作原理在代码中相当于：

```Objective-C
- (void)setName:(NSString *)newName { 
      [self willChangeValueForKey:@"name"];    //KVO 在调用存取方法之前总调用 
      [super setValue:newName forKey:@"name"]; //调用父类的存取方法 
      [self didChangeValueForKey:@"name"];     //KVO 在调用存取方法之后总调用
}
```
#### 4.消息转发(热更新)解决Bug(JSPatch)
> JSPatch 是一个 iOS 动态更新框架，只需在项目中引入极小的引擎，就可以使用 JavaScript 调用任何 Objective-C 原生接口，获得脚本语言的优势：为项目动态添加模块，或替换项目原生代码动态修复 bug。

关于消息转发，前面已经讲到过了，消息转发分为三级，我们可以在每级实现替换功能，实现消息转发，从而不会造成崩溃。JSPatch不仅能够实现消息转发，还可以实现方法添加、替换能一系列功能。
#### 5.实现NSCoding的自动归档和自动解档
原理描述：用runtime提供的函数遍历Model自身所有属性，并对属性进行encode和decode操作。 核心方法：在Model的基类中重写方法：

```Objective-C
- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int outCount;
        Ivar * ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar ivar = ivars[i];
            NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            [self setValue:[aDecoder decodeObjectForKey:key] forKey:key];
        }
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
        [aCoder encodeObject:[self valueForKey:key] forKey:key];
    }
}
```
#### 6.实现字典和模型的自动转换(MJExtension)
原理描述：用runtime提供的函数遍历Model自身所有属性，如果属性在json中有对应的值，则将其赋值。 核心方法：在NSObject的分类中添加方法

```Objective-C
- (instancetype)initWithDict:(NSDictionary *)dict {

    if (self = [self init]) {
        //(1)获取类的属性及属性对应的类型
        NSMutableArray * keys = [NSMutableArray array];
        NSMutableArray * attributes = [NSMutableArray array];
        /*
         * 例子
         * name = value3 attribute = T@"NSString",C,N,V_value3
         * name = value4 attribute = T^i,N,V_value4
         */
        unsigned int outCount;
        objc_property_t * properties = class_copyPropertyList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            objc_property_t property = properties[i];
            //通过property_getName函数获得属性的名字
            NSString * propertyName = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            [keys addObject:propertyName];
            //通过property_getAttributes函数可以获得属性的名字和@encode编码
            NSString * propertyAttribute = [NSString stringWithCString:property_getAttributes(property) encoding:NSUTF8StringEncoding];
            [attributes addObject:propertyAttribute];
        }
        //立即释放properties指向的内存
        free(properties);

        //(2)根据类型给属性赋值
        for (NSString * key in keys) {
            if ([dict valueForKey:key] == nil) continue;
            [self setValue:[dict valueForKey:key] forKey:key];
        }
    }
    return self;

}
```
以上就是 *Runtime* 应用的一些场景，本文到此结束了。

参考&转载: [iOS Runtime详解](https://juejin.im/post/5ac0a6116fb9a028de44d717)