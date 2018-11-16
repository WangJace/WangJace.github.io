---
layout: post
title:  "Swift与Objective-C的差异"
tags: [iOS笔记]
---
**1.Optional**    
Optional是Swift的一个新特性。它允许函数返回一个可选值，即在函数返回值不确定是否为nil的时候。在Objective-C是直接返回nil，但在Swift不能直接返回nil，可以选择返回一个可选类型。
{% highlight Swift %}
let a = "512"
let b = "Hello"
let c = Int(a)
let d = Int(b)
print(c) // Optional(512)
print(d) // nil
{% endhighlight %}
**2.流程控制**    
在Swift中，使用*if...else...*、*for*、*while*、*repeat*时，*{ }* 是不可以省略的。而在Objective-C中，如果流程控制中只有一个语句，则*{ }*是可以省略的。    
**3.类型推断**    
在Swift中，定义一个变量或者常量，可以不申明数据类型，编译器会根据数值推断数据类型。而在Objective-C中，定义一个变量或者常量必须同时申明数据类型。
{% highlight Swift %}
var str = "Some string"
// OR
var str2:String
str2 = "Other string"
{% endhighlight %}
{% highlight Objective-C %}
NSString str = @"There is no type inference in Objective-C :("
{% endhighlight %}
**4.元组**    
元组是Swift新增的又一个新特性，它可以存储一组数据，但不像数组，元组存储的数据可以是不同类型的。
{% highlight Swift %}
var t:(String, Int) = ("John", 33)
print(t.0, t.1)
 
var j:(name:String, Int) = ("Morgan", 52)
print(j.name, j.1)
{% endhighlight %}
当函数需要返回多个类型的数据时，就可以使用元组来实现。
{% highlight Swift %}
var arr = [23, 5, 7, 33, 9]
func findPosition(el:Int, arr:[Int]) -> (found:Bool, position:Int)? {
   if arr.isEmpty {
      return nil
   }
   for (i, value) in arr.enumerate() {
      if value == el {
         return (true, i)
      }
   }
   return (false, -1)
}
 
if let (isFound, elPosition) = findPosition(5, arr: arr) {
   print(isFound, elPosition)
   // true 1
}
{% endhighlight %}
**5.字符串操作**    
在Objective-C中，*NString*和*NSMutableString*分别表示不可变字符串和可变字符串，但Swift就不再需要如此区分啦，不可变字符串直接使用*let*申明，可变字符串则使用*var*申明即可。在字符串拼接操作上，Swift要比Objective-C简便多了。
{% highlight Swift %}
// Swift:
var str = "My string"
str += " and another string”
{% endhighlight %}
{% highlight Objective-C %}
// Objective-C:
NSString *myString = @"My string";
myString = [NSString stringWithFormat:@"%@ and another string", myString];
{% endhighlight %}
在字符串中拼接一些其他类型的数据：
{% highlight Objective-C %}
// Objective-C:
NSString *str = [NSString stringWithFormat:@"String: %@ | Signed 32-bit integer: %d | 64-bit floating-point number: %f", @"My String", myInt, myFloat];
{% endhighlight %}
{% highlight Swift %}
// Swift:
var str = "String: \(myString) | Signed 32-bit integer: \(myInt) | 64-bit floating-point number: \(myFloat)"
{% endhighlight %}
**6.Guard & Defer**    
先看一段Objective-C代码：
{% highlight Objective-C %}
enum TriangleAreaCalcError: ErrorType {
   case AngleNotSpecified
   case InvalidAngle
   case SideANotSpecified
   case SideBNotSpecified
}
func calcTriangleArea(a: Double ? , b : Double ? , alpha : Double ? ) throws - > Double {
   if let a = a {
      if let b = b {
         if let alpha = alpha {
            if alpha < 180 && alpha >= 0 {
               if alpha == 180 {
                  return 0
               }
               return 0.5 * a * b * sin(alpha * M_PI / 180.0)
            } else {
               throw TriangleAreaCalcError.InvalidAngle
            }
         } else {
            throw TriangleAreaCalcError.AngleNotSpecified
         }
      } else {
         throw TriangleAreaCalcError.SideBNotSpecified
      }
   } else {
      throw TriangleAreaCalcError.SideANotSpecified
   }
}
{% endhighlight %}
这段代码出现了多层*if...else...*嵌套，这种代码比较不好维护。但在Swift中，可以使用*guard*来解决这个问题。
{% highlight Swift %}
func calcTriangleArea(a: Double ? , b : Double ? , alpha : Double ? ) throws - > Double {
   guard let a = a else {
      throw TriangleAreaCalcError.SideANotSpecified
   }
   guard let b = b else {
      throw TriangleAreaCalcError.SideBNotSpecified
   }
   guard let alpha = alpha else {
      throw TriangleAreaCalcError.AngleNotSpecified
   }
   if alpha == 180 {
      return Double(0)
   }
   guard alpha < 180 && alpha >= 0 else {
      throw TriangleAreaCalcError.InvalidAngle
   }
   return 0.5 * a * b * sin(alpha * M_PI / 180.0)
}
{% endhighlight %}
Swift中还提供了*defer*，可以在函数代码执行结束，离开代码块直接执行*defer*的代码。
{% highlight Swift %}
func someImageFunc() - > UIImage ? {
   let dataSize: Int = ...
   let destData = UnsafeMutablePointer < UInt8 > .alloc(dataSize)
   // ...
   guard error else {
      destData.dealloc(dataSize) // #1
      return nil
   }
   guard error2 else {
      destData.dealloc(dataSize) // #2
      return nil
   }
   guard error3 else {
      destData.dealloc(dataSize) // #3
      return nil
   }
   destData.dealloc(dataSize) // #4
   // ...
}
{% endhighlight %}
上面的代码，*destData.dealloc(dataSize)*这句代码被重复写了4次。使用*defer*，可以对代码进行优化。
{% highlight Swift %}
func someImageFunc() - > UIImage ? {
   let dataSize: Int = ...
   let destData = UnsafeMutablePointer < UInt8 > .alloc(dataSize)
   // ...
   defer {
      destData.dealloc(dataSize)
   }
   guard error else {
      return nil
   }
   guard error2 else {
      return nil
   }
   guard error3 else {
      return nil
   }
   // ...
}
{% endhighlight %}
**7.函数式**    
Swift引进了一系列函数式编程的特征，例如用于集合遍历的*map*和*fliter*。而Objective-C本身并不支持函数式编程，如果想在Objective-C使用函数式编程，必须使用一些第三方的库，例如ReactiveCocoa。
{% highlight Swift %}
let a = [4, 8, 16]
   print(a.map{$0 / 2})
   // [2, 4, 8]
   let a:[(name:String, area:String)] = [("John", "iOS"), ("Sam", "Android"), ("Paul", "Web")]
   let b = a.map({"Developer \($0.name) (\($0.area))"})
   // ["Developer John (iOS)", "Developer Sam (Android)", "Developer Paul (Web)”]
   let a = [23, 5, 7, 12, 10]
   let b = a.filter{$0 > 10}
   print(b) // [23, 12]
   let sum = (20...30)
   .filter { $0 % 2 != 0 }
   .map { $0 * 2 }
   .reduce(0) { $0 + $1 }
   print(sum) // 250
{% endhighlight %}
**8.枚举**    
在Swift中，枚举更加强大，可以在枚举中定义方法，传值等。可以通过下面的例子一窥枚举在Swift中的使用方法。
{% highlight Swift %}
enum Location {
   case Address(city: String, street: String)
   case Coordinates(lat: Float, lon: Float)
   func printOut() {
      switch self {
         case let.Address(city, street):
            print("Address: " + street + ", " + city)
         case let.Coordinates(lat, lon):
            print("Coordiantes: (\(lat), \(lon))")
      }
   }
}
let loc1 = Location.Address(city: "Boston", street: "33 Court St")
let loc2 = Location.Coordinates(lat: 42.3586, lon: -71.0590)
loc1.printOut() // Address: 33 Court St, Boston
loc2.printOut() // Coordiantes: (42.3586, -71.059)
{% endhighlight %}
枚举还可以实现递归，因此我们使用枚举构建一个链表。
{% highlight Swift %}
enum List {
   case Empty
      indirect
   case Cell(value: Int, next: List)
}
let list0 = List.Cell(value: 1, next: List.Empty)
let list1 = List.Cell(value: 4, next: list0)
let list2 = List.Cell(value: 2, next: list1)
let list3 = List.Cell(value: 6, next: list2)
let headL = List.Cell(value: 3, next: list3)
func evaluateList(list: List) - > Int {
   switch list {
      case let.Cell(value, next):
         return value + evaluateList(next)
      case .Empty:
         return 0
   }
}
print(evaluateList(headL)) // 16
{% endhighlight %}
**9.函数**    
Swift的每一个函数都有一个类型，这个类型有函数的参数类型和返回值类型组成。这意味着函数可以作为变量，也可以作为参数传递给其他函数。
{% highlight Swift %}
func stringCharactersCount(s: String) - > Int {
   return s.characters.count
}
func stringToInt(s: String) - > Int {
   if let x = Int(s) {
      return x
   }
   return 0
}
func executeSuccessor(f: String - > Int, s: String) - > Int {
   return f(s).successor()
}
let f1 = stringCharactersCount
let f2 = stringToInt
executeSuccessor(f1, s: "5555") // 5
executeSuccessor(f2, s: "5555") // 5556
{% endhighlight %}
Swift还可以给函数参数定义默认值。
{% highlight Swift %}
func myFunction(someInt: Int = 5) {
   // If no arguments are passed, the value of someInt is 5
}
myFunction(6) // someInt is 6
myFunction() // someInt is 5
{% endhighlight %}
**10.Do 语句**    
Swift可以使用*do*来捕获异常。
{% highlight Swift %}
do {
   try expression
   statements
} catch pattern 1 {
   statements
} catch pattern 2 where condition {
   statements
}
{% endhighlight %}


未完，待续...


