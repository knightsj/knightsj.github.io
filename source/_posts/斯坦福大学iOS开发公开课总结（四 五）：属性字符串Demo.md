---
title: 斯坦福大学iOS开发公开课总结（四 五）：属性字符串Demo
tags: [iOS,Objective-C]
categories: iOS
---


本节课讲解了iOS框架里几个重要的知识点：动态绑定，控制器的生命周期，属性字符串等。最后演示了一个Demo用来讲解属性字符串的几个功能。


<!-- more -->

# 动态绑定
---
在OC的编译期，所有的指针都是id类型，只有在运行时，对象的类型才会被确定。

举个🌰 ：
在编译期，``NSString*`` 实际上与id相同，但是加上去的好处是让编译器知道你至少是**意图让该指针指向一个字符串**。所以如果尝试发送非字符串消息给该指针，它会发出警告，但是不会提示错误，仍可以通过编译。但是如果在运行时就会“露馅”，因为此时如果向该对象发送非字符串消息时，就会引起崩溃。


再举个具体的🌰：

```
NSString *hellow = @"hello";
Ship *hellowShip = (Ship *)hello;
[helloShip shoot];

```

>编译器会认为``hellowShip``是``ship``类型，因此向``hellowShip``发送``shoot``消息时，在编译器期是可以通过的。
>但是，``hellowShip``实际上指向的是字符串，所以会导致在运行时崩溃。

所以就引出了**类型保护机制**用来确定对象的类型：

# 类型保护机制
---
### 没有添加类型保护机制：

```
PlayingCard *otherCard = [otherCards firstObject];
[otherCard play];

```

>firstObject 方法返回的是id类型，这里需要保护机制确保取出的对象是``PlayingCard``的实例，以防止向其发送消息时导致程序崩溃。

### 添加了类型保护机制：

```
PlayingCard *otherCard = [otherCards firstObject];

id card = [otherCards firstObjct];
if ([card isKindOfClass:[PlayingCard class]])
{
   PlayingCard *otherCard = (PlaytingCard *)card;
   [otherCard play];

}

```

>我们可以看到``card``指针通过``isKindOfClass:``方法被确认了是``PlayingCard``类的实例，那么如果我们给``card``实例发送其消息时，就不会发生崩溃。反之，若``card``是其他类的实例，如果向其发送``card``类的消息就会非常危险！

# NSRange
---
NSRange是一个表示“范围”的结构体，包括起点和长度,主要用于字符串。

常用方法：

#### 字符串所有的字符：

```
NSString *title = @"好好学习天天向上";
NSMakeRange(0, [title length])

```

#### 判断某个字符串里包含某个字符：

```
NSString *greeting = @"hellow world";
Nsstring *hi = @"hi";
NSRange r = [greeting rangeOfString:hi];
if(r.location != NSNotFound)
{
    NSLog(@"Found");
}
```


# 控制器生命周期
---
在控制器(ViewController)的生命周期里，处于某个特定的时间点会执行某个特定的方法。通过在这些方法里之行某些特定的任务，可以正确地实现其应实现的功能。


### viewDidLoad

控制器的``viewDidLoad``方法在控制器的view为nil的时候被调用，在控制器的生命周期中只调用一次。

```

- （voidviewDidLoad
{
   [super viewDidLoad];    
   
   //可执行：
   //1. 控制器的初始化数据
   //2. 网络请求
   
   
   //不可执行：
   //1. 视图形状的初始化信息
}
```


### viewWillAppear:

控制器的``viewWillAppear:``在UIViewController对象的视图即将加入窗口时调用。只要该控制器的view即将要出现，都会调用，在控制器的生命周期中可以调用多次。
而且，如果该方法被调用，就说明视图**一定**会出现在屏幕上。


```
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    
    //可执行：
    //1. 更新view离开界面后可能会改变的数据。
    //2. view的几何变化。
        
}

```


### viewWillDisappear:

控制器的``viewWillDisappear:``在UIViewController的view即将不显示的时候调用，在控制器的生命周期中可以调用多次。

```
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    //可执行：
    //1. 记录滚动视图的偏移量(因为要记住滚动位置，便于下次查看)
    //2. 存储数据，便于再次显示该控制器时使用。
}

```


# 属性字符串Demo
---


## 设计需求

- 布局为TextView下方有四个颜色按钮，再下方有添加轮廓按钮和去除轮廓按钮。
- 选中TextView的文本后，点击色彩按钮，选中的文本的颜色变成点击的色彩按钮的背景色。
- 选中TextView的文本后，点击添加轮廓，选中的文本增加了轮廓，再点击色彩按钮，轮廓变成了相应的颜色。
- 文本有轮廓的状态下，点击去除轮廓按钮，轮廓消失。
- 在设置选项来改变系统字体，再回到本Demo界面，字体会做相应改变。


## 效果图

![属性字符串效果图](http://upload-images.jianshu.io/upload_images/859001-4660010abbea2854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 知识点详解

#### 属性字符串的设置

属性字符串分为不可变属性字符串``NSAttributedString``和``NSMutableAttributedString``。

设置属性字符串的一般步骤为：

1. 初始化可变属性字符串。
2. 向其添加属性字典和制定属性字典被应用的范围。

举个🌰：

```

//1. 由现有字符串初始化可变属性字符串
NSMutableAttributedString *title = [[NSMutableAttributedString alloc] initWithString:self.outLineButton.currentTitle];

//2. 添加属性字典和范围
[title setAttributes:@{NSStrokeWidthAttributeName : @3,
                      NSStrokeColorAttributeName  : self.outLineButton.tintColor}
                                             range: NSMakeRange(0, [title length])];

//3. 将属性字符串赋给按钮的属性字符串属性
[self.outLineButton setAttributedTitle:title forState:UIControlStateNormal];

```


```
//设定选中的字都被设置为和点击的按钮一样的背景颜色
[self.textView.textStorage  addAttribute:NSForegroundColorAttributeName value:sender.backgroundColor range:self.body.selectedRange];

```


#### 关于按钮的操作


```
//获取按钮的背景色
self.button.backgroundColor

```

```
//获取按钮当前的标题
self.button.currentTitle

```

```
//设定按钮当前的属性字符串标题
[self.button setAttributedTitle:title forState:UIControlStateNormal];

```


#### 属性字典里的key：

- ``NSForegroundColorAttributeName``:属性字符串字符的颜色
- ``NSStrokeColorAttributeName``:属性字符串字符轮廓的颜色
- ``NSStrokeWidthAttributeName``:属性字符串字符轮廓的宽度


#### 获取TextView被选中的范围

```
self.textView.selectedRange

```

#### 通知机制

为了实现本Demo最后一个需求，我们需要监听系统字体何时被改变了。所以需要注册一个能收听“系统改变”广播的频道：

注册通知：

```
 [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(prefredFontsChaged:) name:UIContentSizeCategoryDidChangeNotification object:nil];

```

这样一来，当系统字体发生变化时，注册该频道的对象会收到通知并执行自定义的方法。
当改变系统字体的大小后，该类会收到通知，并调用``prefredFontsChaged: ``方法，此时Demo上的字体也要做相应的改变：

```
- (void)prefredFontsChaged: (NSNotification *)notification
{
    //收到通知后，调用本地自定义的方法
    [self userPreferredFonts];
}

- (void)userPreferredFonts
{
    //使用被改变后的系统字体
    self.body.font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
    self.headLine.font = [UIFont preferredFontForTextStyle:UIFontTextStyleHeadline];
    
}

```

>这里，显然又是一个MVC的流程：系统字体(模型)被改变了，通过广播(通知)的机制来告诉控制器，然后控制器再调用更改View的方法。还记得在第一篇（详情请见：[斯坦福大学iOS开发公开课总结（一） ：iOS的MVC框架](http://www.jianshu.com/p/eb58ab21080a)）里强调的，从模型到控制器的通信是通过广播或KVO机制完成的么？

# 最后的话
---
如果哪位小伙伴想拿到此Demo的代码请不要客气，在评论里留言即可。
而且十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~

# 2016.7.12日更新：
---

笔者已经把目前为止整理的所有Demo(第二课到第十课)放入到了我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)仓库里。分为英文注释版和中文注释版(英文注释要少一点，嘿嘿)想要的小伙伴可以果断下载~ 如果有不知道怎么下载的小伙伴请联系我~




**-------------------------------------------------   2018年7月17日更新  -------------------------------------------------**


**注意注意！！！**

笔者在近期开通了个人公众号，主要分享编程，读书笔记，思考类的文章。

- **编程类**文章：包括笔者以前发布的精选技术文章，以及后续发布的技术文章（以原创为主），并且逐渐脱离 iOS 的内容，将侧重点会转移到**提高编程能力**的方向上。
- **读书笔记类**文章：分享**编程类**，**思考类**，**心理类**，**职场类**书籍的读书笔记。
- **思考类**文章：分享笔者平时在**技术上**，**生活上**的思考。

>因为公众号每天发布的消息数有限制，所以到目前为止还没有将所有过去的精选文章都发布在公众号上，后续会逐步发布的。

**而且因为各大博客平台的各种限制，后面还会在公众号上发布一些短小精干，以小见大的干货文章哦~**

扫下方的公众号二维码并点击关注，期待与您的共同成长~

![公众号：程序员维他命](http://upload-images.jianshu.io/upload_images/859001-5bddfacafb9e9079.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



