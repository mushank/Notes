---
title: "Objective-C中instancetype和id的区别"
date: 2015-11-04 10:45:00
tag: [VIM]
blog: true
layout: post

---

一、什么是instancetype
instancetype是clang 3.5开始，clang提供的一个关键字，表示某个方法返回的未知类型的Objective-C对象。我们都知道未知类型的的对象可以用id关键字表示，那为什么还会再有一个instancetype呢？
二、关联返回类型（related result types）
根据Cocoa的命名规则，满足下述规则的方法：
1、类方法中，以alloc或new开头
2、实例方法中，以autorelease，init，retain或self开头
会返回一个方法所在类类型的对象，这些方法就被称为是关联返回类型的方法。换句话说，这些方法的返回结果以方法所在的类为类型，说的有点绕口，请看下面的例子：

```
@interface NSObject  
+ (id)alloc;  
- (id)init;  
@end  
  
@interface NSArray : NSObject  
@end  
```

当我们使用如下方式初始化NSArray时：

```
NSArray *array = [[NSArray alloc] init];  
```

按照Cocoa的命名规则，语句[NSArray alloc] 的类型就是NSArray*因为alloc的返回类型属于关联返回类型。同样，[[NSArray alloc]init] 的返回结果也是NSArray*。
三、instancetype作用
1、作用
如果一个不是关联返回类型的方法，如下：

```
@interface NSArray  
+ (id)constructAnArray;  
@end  
```

当我们使用如下方式初始化NSArray时：

```
[NSArray constructAnArray];  
```

根据Cocoa的方法命名规范，得到的返回类型就和方法声明的返回类型一样，是id。
但是如果使用instancetype作为返回类型，如下：

```
@interface NSArray  
+ (instancetype)constructAnArray;  
@end  
```

当使用相同方式初始化NSArray时：

```
[NSArray constructAnArray];  
```


得到的返回类型和方法所在类的类型相同，是NSArray*!
总结一下，instancetype的作用，就是使那些非关联返回类型的方法返回所在类的类型！
2、好处
能够确定对象的类型，能够帮助编译器更好的为我们定位代码书写问题，比如：

```
[[[NSArray alloc] init] mediaPlaybackAllowsAirPlay]; //  "No visible @interface for `NSArray` declares the selector `mediaPlaybackAllowsAirPlay`"  
  
[[NSArray array] mediaPlaybackAllowsAirPlay]; // (No error)  
```

上例中第一行代码，由于[[NSArray alloc]init]的结果是NSArray*，这样编译器就能够根据返回的数据类型检测出NSArray是否实现mediaPlaybackAllowsAirPlay方法。有利于开发者在编译阶段发现错误。
第二行代码，由于array不属于关联返回类型方法，[NSArray array]返回的是id类型，编译器不知道id类型的对象是否实现了mediaPlaybackAllowsAirPlay方法，也就不能够替开发者及时发现错误。
四、instancetype和id的异同
1、相同点
都可以作为方法的返回类型
2、不同点
①instancetype可以返回和方法所在类相同类型的对象，id只能返回未知类型的对象；
②instancetype只能作为返回值，不能像id那样作为参数，比如下面的写法：

```
//err,expected a type  
- (void)setValue:(instancetype)value  
{  
    //do something  
} 
```

就是错的，应该写成：

```
- (void)setValue:(id)value  
{  
    //do something  
}  
```

原文地址：http://blog.csdn.net/wzzvictory/article/details/16994913

