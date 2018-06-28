---
title: 斯坦福大学iOS开发公开课总结（十八）：国际化，自定义设置UI
tags: [iOS,Objective-C]
categories: iOS
---

本篇是斯坦福大学iOS7系列课程（CS193P）的最后一节课的总结，终于把18节课的内容都总结完了，而且这个文集也画上了句号，有点不舍的赶脚。。

好了，不煽情了，开始！
<!-- more -->

# 国际化 Internationalization

----------

如果我们想要将app推广到国际市场，那么就免不了将我们的app翻译成其他国家的语言以便于当地人去使用。

而翻译的内容主要集中于“非内容”部分：例如标题类文字，按钮上的文字，提示框的文字等等。


所谓“翻译”app的过程主要分为**国际化**和**本地化**两个步骤：

1. 国际化：是让app能够本地化的过程。
2. 本地化：将程序翻译另外一种语言。
     - 故事版字符串的本地化：改变故事版中出现的字符串。
     - 类文件字符串的本地化：改变文件中出现的字符串，包括字符串面量和非字符串面量。


>注意：
1. 好的UI设计：给用户呈现出的内容大多数都需要显示的（内容），而不是自己添加的（提示）。这样一来，也会减少我们本地化的工作量。
2. 所有的地方都要设置好自动布局，否则本地化将无从谈起，因为将文字翻译成其他语言后，长度很可能是不一样的。


下面来来具体看一下本地化的方法，分为**故事版中字符串**的本地化和**类文件中字符串**的本地化。


## 1. 故事版中的本地化


在故事版中本地化的过程是：

1. 在项目里添加其他语言。
2. 在被添加语言的故事版文件中找到相应的.string文件，加以更改。


具体过程如下图所示：

![故事版的本地化.gif](http://upload-images.jianshu.io/upload_images/859001-917657136741d815.gif?imageMogr2/auto-orient/strip)




## 2. 类文件中字符串的本地化

除了需要本地化故事版中的字符串，类文件中的字符串也需要本地化，因为故事版并不能显示所有需要翻译的字符串。

而且，类文件中的字符串分为两种形式：
1. 字符串面量。
2. 非字符串面量。

字符串面量:

``
NSString *myName = @“Jack”;
``

类似这样的字符串是比较好找的，只需要搜索``@``即可很容易找到。
但是，我们仍然会通过该方法搜索到不应该本地化的字符串：
也就是不出现在UI中的字符串面量。

例如：

- 文件扩展名
- segure的identifier
- stiringWithFormat:



#### 2.1字符串面量的本地化步骤：


1. 找到需要本地化的字符串面量。
2. 通过宏，将字符串面量添加到表中。
3. 创建表的.string文件。
4. 本地化.string文件


1.找到 ``@"Sorry, this device cannot add a photo."`` 字符串面量。


```
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    if (![[self class] canAddPhoto]) {
        [self fatalAlert:@"Sorry, this device cannot add a photo."]; //
    } else {
        [self.locationManager startUpdatingLocation];
    }
}
```


2.通过宏，将字符串面量添加到表中。


```
 - (void)viewDidAppear:(BOOL)animated
 {
    [super viewDidAppear: animated];
    
    if (![[self class] canAddPhoto]) {
        [self fatalAlert:ALERT_CANT_ADD_PHOTO]; // @"Sorry, this device cannot add a photo."
    } else {
        [self.locationManager startUpdatingLocation];
    }
 }
```


```
#define ALERT_CANT_ADD_PHOTO NSLocalizedStringFromTable(@"ALERT_CANT_ADD_PHOTO", @"AddPhotoViewController", @"Alert message delivered when there is something that prevents the user from adding a new photo to the database that the user can do nothing about.")
```



3.创建表的.string文件

我们已经在AddPhotoViewController.m文件里将需要本地化的字符串添加到了叫做AddPhotoViewController的表里，下面就需要用命令行工具找到该.m文件：

![找到.m文件所在目录](http://upload-images.jianshu.io/upload_images/859001-a043fa2da855615c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![输入genstrigns ^*m命令](http://upload-images.jianshu.io/upload_images/859001-b3adf539663b1269.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


4.本地化.string文件

在第三步中，我们需要手动创建``AddPhotoViewController``表的.string文件，然后在文件内部将对应的key进行翻译。因为过程略繁琐，所以以动图的形式呈献给大家：

![本地化string.gif](http://upload-images.jianshu.io/upload_images/859001-100dfbddbe7399bb.gif?imageMogr2/auto-orient/strip)


>失误了，右侧的字符串应该是没有“@”的，大家注意。因为到最后才发现的，不好改了 额。。 理解万岁。。

## 2.2 非字面量字符串的本地化：

在类文件里，有些显示出来的字符串并不都是通过字符串面量赋值的，比如下面这个例子：

``
self.title = newfrc.fetchRequest.entity.name;
``

在这里，title取的是模型里的字段，并没有用字面量语法来表示。
对于这种情况，我们需要用NSBundle的``localizedStringForKey:value:table:``方法来进行本地化。


```
self.title = [[NSBundle mainBundle] localizedStringForKey:newfrc.fetchRequest.entity.name
                         value:newfrc.fetchRequest.entity.name     
                         table:@"Entities"];                                                          
```

这样一来，我们就生成了对应名字叫**Entities**的表的映射。但是这张表对应的.string文件还没有生成，需要我们手动去生成，并设置对应的key和value。生成方法如下所示：



![手动生成.string文件](http://upload-images.jianshu.io/upload_images/859001-965d85e9b008d0c8.gif?imageMogr2/auto-orient/strip)



# 设置页的UI
---------

在苹果系统的设置里，会有我们装入的app的信息和设置。有时，我们需要将一些设置选项放在这里面供用户使用。

而这里的UI是通过通过Settings bundle来设定的。我们首先要新建一个Settings Bundle:



![创建Settings Bundle](http://upload-images.jianshu.io/upload_images/859001-d49a21d343306156.gif?imageMogr2/auto-orient/strip)

创建成功后，分别有一个slider，switch，和textfield来对应设置页里的UI。

在设置页里的样子是这样的：


![新建的Settings Bundle 后的设置页效果](http://upload-images.jianshu.io/upload_images/859001-6dc1b24f8d070871.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


那么如何使用呢？我们设置一下上一节课的Bouncer Demo的弹性系数：让用户通过调节设置里的slider来调整app里的弹性系数。


```
- (void)resetElasticity
{
    //连接代码与Setting Bundle
    NSNumber *elasticity = [[NSUserDefaults standardUserDefaults] valueForKey:@"Setting_Elasticity"];//连接setting bundle
    if (elasticity) {
        //如果有，就取当前设定的
        self.elastic.elasticity = [elasticity floatValue];
    } else {
        //如果没有，就设置为1
        self.elastic.elasticity = 1.0;
    }
}
```

在这里，通过``valueForKey``的键值对应了Setting Bundle plist 文件里的``identifier``，我们将plist文件里的``identifier``修改成了``Setting_Elasticity``而且更改了``Title``，而且将按钮和文本框删除掉，只保留了slider：

![Setting Bundle plist](http://upload-images.jianshu.io/upload_images/859001-f195429739d8262b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


所对应的设置页的UI：

![新的设置页UI](http://upload-images.jianshu.io/upload_images/859001-1d15dcab18d5df94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




现在我们可以通过identifier连接了代码和plist文件，获取到了键对应的值。

而且，我们还需考虑在程序运行过程中，用户跳转到了设置页面来设置弹性系数的情况。因此，我们需要监听用户是否更改了设置里的选项：


监听用户在设置中的行为：

```
    [[NSNotificationCenter defaultCenter] addObserverForName:NSUserDefaultsDidChangeNotification
                                                      object:nil
                                                       queue:nil
                                                  usingBlock:^(NSNotification *note) {
                                                      [self resetElasticity];
                                                  }];
```


# 最后的话
-----

当~当~当~当~！
笔者终于利用了2个月的部分业余时间总结了所有斯坦福iOS7的课程和相关Demo。通过以博客的形式总结，更加加深了对知识的理解和认识，也对基础知识进行了一次查缺补漏，或许也在一定程度上给其他看到这些博客的同仁们一些帮助吧~

最后附上这一系列笔者总结的所有Demo在GitHub上的地址：[Stanford_iOS_Lecture_DemoBundle](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)。

笔者下一阶段应该是总结下面两本书的内容：

![Effective Objective- C 2.0](http://upload-images.jianshu.io/upload_images/859001-0eeac0652f909c84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




![Swift 基础教程](http://upload-images.jianshu.io/upload_images/859001-c1046257878a01d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这两本书笔者都已经看了3分之一，因为要总结归纳，所以应该进度不是很快，不过还是会坚持写博客的！

加油~


