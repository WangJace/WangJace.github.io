#<center>《Effective Objective-C 2.0》读书笔记</center>
##1.2在类的头文件中尽量少引入其他头文件
1.除非确有必要，否则不要引入头文件。一般来说，硬在某个类的头文件中使用向前声明（即使用@class来申明一个类）来提及别的类，并在实现文件中引入那些类的头文件。这样做可以精良降低类之间的耦合度。

2.有事无法使用向前申明，比如要声明某个类遵循某协议。这种情况下，尽量吧“该类遵循的协议”的这条生命移至分类中。如果不行的话，酒吧协议单独放在一个头文件中，然后将其引入
##1.4多用类型常量，少用define预处理指令
1.不要用预处理指令定义常量。这样定义出来的常量不含类型信息，编译器只是会变异前据此执行查找与替换操作。基石有人重新定义了常量值，编译器也不会产生警告信息，这将导致应用程序中的常量值不一致。

2.在实现文件中使用`static const`来定义“只在编译单元内可见的常量”。由于此类常量不在全局符号表中，所以无需为其名称加前缀。

3.在头文件中使用`extern`来申明全局常量，并在相关实现文件中定义其值。这种常量要出现子啊全局符号表中，所以其命名应加以区分，通畅用与之相关的类名做前缀。
##1.5用枚举表示状态、选项、状态码
1.应该用枚举来表示状态机的状态、传递给方法的选项以及状态码等值，给这些值起个易懂的名字。

2.如果把传递给某个方法的选项表示为枚举类型，而多个选项又可同时使用，那么就将各选项值用二进制位表示，以便通过按位或操作将其组合起来。
例：
<pre><code>
typedef NS_ENUM(NSUInteger, Direction) {
	Up = 1 << 0,
	Down = 1 << 1,
	Left = 1 << 2,
	Right = 1 << 3,
}
</code></pre>
此时就可在返回中出现Up|Down等多个选项同时使用

3.用`NS_ENUM`与`NS_OPTIONS`宏来定义枚举类型，并指明其底层数据类型。这样做可以确保枚举是用开发者所选的底层数据类型实现出来的，而不会采用编译器所选的类型。

4.在处理枚举类型的switch语句中不要实现default分支。这样的话，加入新枚举之后，编译器就会提示开发者：switch语句并未处理所有枚举。
##2.7在对象内部尽量直接访问实例变量
1.在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，则应通过属性来写。

2.在初始化方法及dealloc方法中，总是应该直接通过实例变量来读写数据。

3.有时会使用惰性初始化技术配置某分数据，这种情况下，需要通过属性来读取数据。使用惰性初始化的原因：某些属性所指代的对象相当复杂，并且由于这些属性不常用，而且创建该属性的成本较高，此事可以在“获取方法”中对此属性执行惰性初始化。
例：
<pre><code>
-(EOCBrain*)brain {
	if (!brain) {
		_brain = [Brain new];
	}
}
</code></pre>
##2.8理解“对象等同性”这一概念
1.若想检测对象的等同性，对象必须是想`isEqual:`与`hash`方法。
例：
<pre><code>
-(BOOL)isEqual:(id)object {
	if (self == object) return YES;
	if ([self class] != [object class]) return NO;
	Person *otherPerson = (Person *)object;
	if (![_name isEqualToString:otherPerson.name])
		return NO;
	if (![_hometown isEqualToString:otherPerson.hometown])
		return NO;
	if (_age != otherPerson.age)
		return NO;
	return YES;
}

-(NSUInteger)hash {
	NSUInteger nameHash = [_name hash];
	NSUInteger hometownHash = [_hometown hash];
	NSUInteger ageHash = _age;
	return nameHash ^ hometownHash ^ ageHash;
}
</code></pre>
2.相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象未必相同。

3.不要盲目地逐个检测对象每个属性，而是应该根据具体需求来指定方案。

4.编写hash方法时，应该使用计算速度快而且哈希码碰撞几率低的算法。
##2.10在既有类中使用关联对象存放自定义数据
Objective-C中可以给某对象关联许多其他的对象，这些对象通过“键”来区分。存储对象值的时候，可以致命“存储策略”（storage polocy），用以维护相应的“内存管理语义”。存储策略有名为`objc_AssociationPolicy`的枚举所定义，表2-1列出了该枚举的取值，同事还列出了与之等效的@property属性：加入关联对象成为了属性，那么它就会具备对应的语义。

*表2-1 对象关联类型*
<table>
	<tr>
		<td style="align:center">关联类型</td>
		<td>等效的@property属性</td>
	</tr>
	<tr>
		<td>OBJC_ASSOCIATION_ASSIGN</td>
		<td>assign</td>
	</tr>
	<tr>
		<td>OBJC_ASSOCIATION_RETAIN_NONATOMIC</td>
		<td>nonatomic,retain</td>
	</tr>
	<tr>
		<td>OBJC_ASSOCIATION_COPY_NONATOMIC</td>
		<td>nonatomic,copy</td>
	</tr>
	<tr>
		<td>OBJC_ASSOCIATION_RETAIN</td>
		<td>retain</td>
	</tr>
	<tr>
		<td>OBJC_ASSOCIATION_COPY</td>
		<td>copy</td>
	</tr>
</table>
下列方法可以管理关联对象：

+ `void objc_setAssociatedObjec(id object, void *key, id value, objc_AssociationPolicy policy)`
此方法以给定的键和策略为某对象设置关联对象值
+ `id objc_getAssociatedObject(id object, void *key)`
此方法根据给定的键从某对象中获取相应的关联对象值
+ `void objc_removeAssociatedObjects(id object)`
此方法移除指定对象的全部关联对象

**关联对象用法举例**
<pre><code>
\#import \<objc/runtime.h>
static void *EOCMyAlertViewKey = "EOCMyAlertViewKey"

-(void)askUserAQuestion {
	UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Question" message:@"What do you want to do?" delegate:self cancelButtonTitle:@"Cancel" OtherButtonTitles:@"Continue",nil];
	void (^block)(NSInteger) = ^(NSInteger buttonIndex) {
		if (buttonIndex == 0) {
			[self doCancel];
		}else {
			[self doContinue];
		}
	};
	objc_setAssociatedObject(alert,EOCMyAlertViewKey,block,OBJC_ASSOCIATION_COPY);
	[alert show];
}

//UIAlertViewDelegate protocol method
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
	void (^block)(NSInteger) = objc_getAssociatedObject(alertView, EOCMyAlertViewKey);
	block(buttonIndex);
}
</pre></code>

**要点**

+ 可以通过“关联对象”机制来把两个对象连起来。
+ 定义关联对象时可指定内存管理语义，用以模仿定义属性时所采用的“拥有关系”与“非拥有关系”。
+ 只有在其他做法不可行时才应选用关联对象，因为这种做法通常会引入难以查找的bug。

##2.12理解消息转发机制
+ **动态方法解析**

	对象在收到无法解读的消息后，首先将调用其所属类的类方法：

	<code>+ (BOOL)resolveInstanceMethod:(SEL)selector</code>

	该方法的参数就是那个未知的选择子，其返回值为Boolean类型，表示这个类是否能新增一个实例方法用以处理此选择子。在继续往下执行转发机制之前，本类有机会新增一个处理次选择子的方法。

+ **备援接收者**

	当选择子无法被当前对象解读时，运行期系统会通过方法：

	<code>- (id)forwardingTargetForSelector:(SEL)selector</code>

	判断是否能将这条信息转给其他接收者来处理。请注意，开发者无法操作精油这一步所转发的消息，若是想在发送给备援接收者之前修改消息内容，那就得通过完整的消息转发机制来做了。

+ **完整的消息转发**

	如果没有可以接受未知选择子的对象，，那么唯一能做的就是启用完整的消息转发机制。首先创建NSInvocation对象，把与尚未处理的那条消息有关的全部细节都封于其中。此对象包含选择子、目标（target）及参数。在触发NSInvocation对象时。“消息派发系统”（message-dispatch system）将把消息指派给目标对象。
	
	此步骤会调用下列方法来转发消息：
	
	<code>- (void)forwardInvocation:(NSInvocation *)invocation</code>
	
	若发现某调用操作不应由本类处理，则需调用超类的同名方法，这样的话，继承体系中的每个类都有机会处理此调用请求，直至NSObject。如果最后调用了NSOvbject类的方法，那么该方法还会继而调用<code>doesNotRecognizeSelector:</code>以抛出异常，此异常表明选择子最终未能得到处理。
	
##3.1