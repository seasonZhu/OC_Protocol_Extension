# OC_Protocol_Extension
这是一个困扰我很久的一个问题.起初是在一次面试上,和面试官讨论起Swift的特性的时候,我们聊到了Swift的Protocol
```
Swift的中的Protocol可以写其Extension方法.
如果遵守该Protocol的类或者结构体针对协议方法的不重写的话,那么就会调用Protocol中Extension的默认方法.
```
上面这段话用代码写非常容易理解
```
protocol Chef {
    func makeFood()
}

extension Chef {
    func makeFood() {
        print("make food")
    }
}

class A: Chef {
    func makeFood() {
        print("A food")
    }
}

class B: Chef {
    
}

let a = A()
let b = B()

a.makeFood()
b.makeFood()

```

打印的日志如下
```
A food
make food
```

**如果我想OC中的Protocol也能够有类似Swift的Protocol中Extension的特性,该如何实现呢?**

这个就是当时面试官提给我的问题.

这个问题是确实是问到我了,OC的中Protocol可以用@request和@optional在遵循的类中的去做强制实现和可选实现,但是要想要一个类去调用遵守协议的默认实现,这个该怎么办呢?

这个问题困扰了我很久,就在最近一次洗澡的时候,我突然顿悟了.

没办法,洗澡的时候想些莫名其妙的问题,我也很无奈.

我们先来创建一个协议,创建文件ClassNameConvertible.h
```
@protocol ClassNameConvertible <NSObject>

+ (NSString *)className;

- (NSString *)className;

@end
```
明眼人一眼就看出来这个协议想干嘛,无非就是获取类名的字符串嘛,我不是经常写OC了,所以总会忽略掉一些OC很重要的细节.
***
OC中什么可以遵守协议?

**对象!**

看看定一个OC协议的格式

`@protocol 协议名 <NSObject>
`
已经了然.

也就是说,遵守协议的必然都是对象,而对象的基类是什么?

**是NSObject**

那么如果NSObject去遵守定义的协议A,并实现协议的默认方法,其他的任何类都会遵守协议A,该一旦调用协议A的方法,其他类如果不重写协议A的方法,那边就会调用NSObject中协议A的默认方法.

如何让NSObject去遵守定义的协议A?

**可以创建一个NSObject的分类去遵守协议A**

这就是解决了上述的问题.

来上代码创建NSObject+ClassName的分类

.h文件
```
#import <Foundation/Foundation.h>

#import "ClassNameConvertible.h"

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (ClassName)<ClassNameConvertible>

@end

NS_ASSUME_NONNULL_END
```
.m文件
```
#import "NSObject+ClassName.h"

#import <objc/objc.h>
#import <objc/runtime.h>

@implementation NSObject (ClassName)

+ (NSString *)className {
    return NSStringFromClass(self);
}

- (NSString *)className {
    return [NSString stringWithUTF8String:class_getName([self class])];
}

@end
```
然后我们任意创建了一个SomeView类继承自UIView
.h和.m文件
```
NS_ASSUME_NONNULL_BEGIN

@interface SomeView : UIView

@end

NS_ASSUME_NONNULL_END
```
```
@implementation SomeView

@end
```
最后我们来一段测试代码
```
    SomeView *someView = [SomeView new];
    
    // 调用函数
    NSString *instaceName = [someView className];
    NSString *className = [SomeView className];
    
    NSLog(@"instaceName:%@", instaceName);
    NSLog(@"className:%@", className);
    
    // 是否遵循协议
    if ([someView conformsToProtocol:@protocol(ClassNameConvertible)]) {
        NSLog(@"someView 遵守ClassNameConvertible 协议");
    }
```
打印
```
instaceName:SomeView
className:SomeView
someView 遵守ClassNameConvertible 协议
```
**有人会说,你对NSObject的分类添加了一个ClassNameConvertible协议,实质上是扩展了整个系统的方法,这样代价也太大了吧.**

**我又没说一定要在NSObject的分类去遵守协议A,你可以在你定义的专用类的分类中去遵守协议A**

这样的话就即实现了Protocol的Extension的默认实现,也将影响力度控制在自己手里.并且你拥有了一个遵守协议的类.

一旦将这个思路扩展下去,其实OC中类泛型的思路也就更加明朗了.

