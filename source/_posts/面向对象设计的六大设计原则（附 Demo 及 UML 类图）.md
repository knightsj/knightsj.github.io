---
title: 面向对象设计的六大设计原则（附 Demo 及 UML 类图）
tags: [iOS,Objectice-C,Object-Oriented,Design Principle]
categories: Object-Oriented
---


![](http://jknight-blog.oss-cn-shanghai.aliyuncs.com/design-principle/odd_dp_banner.png)



# 学习初衷与讲解方式

笔者想在iOS从业第三年结束之前系统学习一下关于设计模式方面的知识。而在学习设计模式之前，觉得更有必要先学习面向对象设计（OOD：Object Oriented Design）的几大设计原则，为后面设计模式的学习打下基础。


本篇分享的就是笔者近阶段学习和总结的面向对象设计的六个设计原则：


缩写 | 英文名称 | 中文名称
---| ----| -----
SRP | Single Responsibility Principle | 单一职责原则
OCP | Open Close Principle | 开闭原则
LSP | Liskov Substitution Principle | 里氏替换原则
LoD | Law of Demeter （ Least Knowledge Principle） | 迪米特法则（最少知道原则）
ISP | Interface Segregation Principle | 接口分离原则
DIP | Dependency Inversion Principle | 依赖倒置原则

<!-- more -->

>注意，通常所说的``SOLID``（上方表格缩写的首字母，从上到下）设计原则没有包含本篇介绍的迪米特法则，而只有其他五项。另外，本篇不包含合成/聚合复用原则(CARP)，因为笔者认为该原则没有其他六个原则典型，而且在实践中也不容易违背。有兴趣的同学可以自行查资料学习。

在下一章节笔者将分别讲解这些设计原则，讲解的方式是将概念与代码及其对应的UML 类图结合起来讲解的方式。

代码的语言使用的是笔者最熟悉的Objective-C语言。虽然是一个比较小众的语言，但是因为有 UML 类图的帮助，而且主流的面向对象语言关于类，接口（Objective-C里面是协议）的使用在形式上类似，所以笔者相信语言的小众不会对知识的理解产生太大的阻力。

另外，在每个设计模式的讲解里，笔者会首先描述一个应用场景（需求点），接着用**两种设计的代码来进行对比**讲解：先提供相对不好的设计的代码，再提供相对好的设计的代码。而且两种代码都会附上标准的 UML 类图来进行更形象地对比，帮助大家来理解。同时也可以帮助不了解 UML 类图的读者先简单熟悉一下 UML 类图的语法。

# 六大设计原则

本篇讲解六大设计原则的顺序大致按照难易程序排列。在这里最先讲解开闭原则，因为其在理解上比较简单，而且也是其他设计原则的基石。

>注意：
>1. 六个原则的讲解所用的例子之间并没有关联，所以阅读顺序可以按照读者的喜好来定。
>2. Java语言里的接口在Objective-C里面叫做协议。虽然Demo是用Objective-C写的，但是因为协议的叫法比较小众，故后面一律用接口代替协议这个说法。



## 原则一：开闭原则（Open Close Principle）


### 定义

> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

即：一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。

### 定义的解读

- 用抽象构建框架，用实现扩展细节。
- 不以改动原有类的方式来实现新需求，而是应该以实现事先抽象出来的接口（或具体类继承抽象类）的方式来实现。

### 优点

实践开闭原则的优点在于可以在不改动原有代码的前提下给程序扩展功能。增加了程序的可扩展性，同时也降低了程序的维护成本。



### 代码讲解

下面通过一个简单的关于在线课程的例子讲解一下开闭原则的实践。


#### 需求点

设计一个在线课程类：

由于教学资源有限，开始的时候只有类似于博客的，通过文字讲解的课程。
但是随着教学资源的增多，后来增加了视频课程，音频课程以及直播课程。

先来看一下不好的设计：

#### 不好的设计

最开始的文字课程类：

```objc
//================== Course.h ==================

@interface Course : NSObject

@property (nonatomic, copy) NSString *courseTitle;         //课程名称
@property (nonatomic, copy) NSString *courseIntroduction;  //课程介绍
@property (nonatomic, copy) NSString *teacherName;         //讲师姓名
@property (nonatomic, copy) NSString *content;             //课程内容

@end
```

``Course``类声明了最初的在线课程所需要包含的数据：

- 课程名称
- 课程介绍
- 讲师姓名
- 文字内容

接着按照上面所说的需求变更：增加了视频，音频，直播课程：



```objc
//================== Course.h ==================

@interface Course : NSObject

@property (nonatomic, copy) NSString *courseTitle;         //课程名称
@property (nonatomic, copy) NSString *courseIntroduction;  //课程介绍
@property (nonatomic, copy) NSString *teacherName;         //讲师姓名
@property (nonatomic, copy) NSString *content;             //文字内容


//新需求：视频课程
@property (nonatomic, copy) NSString *videoUrl;

//新需求：音频课程
@property (nonatomic, copy) NSString *audioUrl;

//新需求：直播课程
@property (nonatomic, copy) NSString *liveUrl;

@end
```

三种新增的课程都在原``Course``类中添加了对应的url。也就是每次添加一个新的类型的课程，都在原有``Course``类里面修改：新增这种课程需要的数据。

这就导致：我们从``Course``类实例化的视频课程对象会包含并不属于自己的数据：``audioUrl``和``liveUrl``：这样就造成了冗余，视频课程对象并不是纯粹的视频课程对象，它包含了音频地址，直播地址等成员。

很显然，这个设计不是一个好的设计，因为（对应上面两段叙述）：

1. 随着需求的增加，需要反复修改之前创建的类。
2. 给新增的类造成了不必要的冗余。

之所以会造成上述两个缺陷，是因为该设计**没有遵循对修改关闭，对扩展开放的开闭原则**，而是反其道而行之：开放修改，而且不给扩展提供便利。

难么怎么做可以遵循开闭原则呢？下面看一下遵循开闭原则的较好的设计：

#### 较好的设计


首先在``Course``类中仅仅保留所有课程都含有的数据：

```objc
//================== Course.h ==================

@interface Course : NSObject

@property (nonatomic, copy) NSString *courseTitle;         //课程名称
@property (nonatomic, copy) NSString *courseIntroduction;  //课程介绍
@property (nonatomic, copy) NSString *teacherName;         //讲师姓名
```


接着，针对文字课程，视频课程，音频课程，直播课程这三种新型的课程采用继承``Course``类的方式。而且继承后，添加自己独有的数据：


文字课程类：

```objc
//================== TextCourse.h ==================

@interface TextCourse : NSObject

@property (nonatomic, copy) NSString *content;             //文字内容

@end
```



视频课程类：

```objc
//================== VideoCourse.h ==================

@interface VideoCourse : Course

@property (nonatomic, copy) NSString *videoUrl;            //视频地址

@end
```

音频课程类：
```objc
//================== AudioCourse.h ==================

@interface AudioCourse : Course

@property (nonatomic, copy) NSString *audioUrl;            //音频地址

@end
```


直播课程类：
```objc
//================== LiveCourse.h ==================

@interface LiveCourse : Course

@property (nonatomic, copy) NSString *liveUrl;             //直播地址

@end
```

这样一来，上面的两个问题都得到了解决：

1. 随着课程类型的增加，不需要反复修改最初的父类（``Course``），只需要新建一个继承于它的子类并在子类中添加仅属于该子类的数据（或行为）即可。
2. 因为各种课程独有的数据（或行为）都被分散到了不同的课程子类里，所以每个子类的数据（或行为）没有任何冗余。

而且对于第二点：或许今后的视频课程可以有高清地址，视频加速功能。而这些功能只需要在``VideoCourse``类里添加即可，因为它们都是视频课程所独有的。同样地，直播课程后面还可以支持在线问答功能，也可以仅加在``LiveCourse``里面。

我们可以看到，正是由于最初程序设计合理，所以对后面需求的增加才会处理得很好。

下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践开闭原则：
![未实践开闭原则](http://oih3a9o4n.bkt.clouddn.com/OCP11.png)

实践了开闭原则：
![实践了开闭原则](http://oih3a9o4n.bkt.clouddn.com/OCP2.png)

>在实践了开闭原则的 UML 类图中，四个课程类继承了``Course``类并添加了自己独有的属性。（在 UML 类图中：实线空心三角箭头代表继承关系：由子类指向其父类）

### 如何实践

为了更好地实践开闭原则，在设计之初就要想清楚在该场景里哪些数据（或行为）是一定不变（或很难再改变）的，哪些是很容易变动的。将后者抽象成接口或抽象方法，以便于在将来通过创造具体的实现应对不同的需求。


## 原则二：单一职责原则（Single Responsibility Principle）


### 定义

>A class should have a single responsibility, where a responsibility is nothing but a reason to change.

即：一个类只允许有一个职责，即只有一个导致该类变更的原因。


### 定义的解读

- 类职责的变化往往就是导致类变化的原因：也就是说如果一个类具有多种职责，就会有多种导致这个类变化的原因，从而导致这个类的维护变得困难。

- 往往在软件开发中随着需求的不断增加，可能会给原来的类添加一些本来不属于它的一些职责，从而违反了单一职责原则。如果我们发现当前类的职责不仅仅有一个，就应该将本来不属于该类真正的职责分离出去。
- 不仅仅是类，函数（方法）也要遵循单一职责原则，即：一个函数（方法）只做一件事情。如果发现一个函数（方法）里面有不同的任务，则需要将不同的任务以另一个函数（方法）的形式分离出去。


### 优点

如果类与方法的职责划分得很清晰，不但可以提高代码的可读性，更实际性地更降低了程序出错的风险，因为清晰的代码会让bug无处藏身，也有利于bug的追踪，也就是降低了程序的维护成本。


### 代码讲解

单一职责原则的demo比较简单，通过对象（属性）的设计上讲解已经足够，不需要具体的客户端调用。我们先看一下需求点：

#### 需求点

初始需求：需要创造一个员工类，这个类有员工的一些基本信息。

新需求：增加两个方法：

- 判定员工在今年是否升职
- 计算员工的薪水

先来看一下不好的设计：

#### 不好的设计


```objc
//================== Employee.h ==================

@interface Employee : NSObject

//============ 初始需求 ============
@property (nonatomic, copy) NSString *name;       //员工姓名
@property (nonatomic, copy) NSString *address;    //员工住址
@property (nonatomic, copy) NSString *employeeID; //员工ID
 
 
 
//============ 新需求 ============
//计算薪水
- (double)calculateSalary;

//今年是否晋升
- (BOOL)willGetPromotionThisYear;

@end
```

由上面的代码可以看出：

- 在初始需求下，我们创建了``Employee``这个员工类，并声明了3个员工信息的属性：员工姓名，地址，员工ID。
- 在新需求下，两个方法直接加到了员工类里面。

新需求的做法看似没有问题，因为都是和员工有关的，但却违反了单一职责原则：**因为这两个方法并不是员工本身的职责**。

- ``calculateSalary``这个方法的职责是属于会计部门的：薪水的计算是会计部门负责。
- ``willPromotionThisYear``这个方法的职责是属于人事部门的：考核与晋升机制是人事部门负责。

而上面的设计将本来不属于员工自己的职责强加进了员工类里面，而这个类的设计初衷（原始职责）就是单纯地保留员工的一些信息而已。因此这么做就是给这个类引入了新的职责，故**此设计违反了单一职责原则**。

我们可以简单想象一下这么做的后果是什么：如果员工的晋升机制变了，或者税收政策等影响员工工资的因素变了，我们还需要修改当前这个类。

那么怎么做才能不违反单一职责原则呢？- 我们需要将这两个方法（责任）分离出去，让本应该处理这类任务的类来处理。

#### 较好的设计

我们保留员工类的基本信息：
```objc
//================== Employee.h ==================

@interface Employee : NSObject

//初始需求
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *address;
@property (nonatomic, copy) NSString *employeeID;
```

然后创建新的**会计部门**类：
```objc
//================== FinancialApartment.h ==================

#import "Employee.h"

//会计部门类
@interface FinancialApartment : NSObject

//计算薪水
- (double)calculateSalary:(Employee *)employee;

@end
```


和**人事部门**类：

```objc
//================== HRApartment.h ==================

#import "Employee.h"

//人事部门类
@interface HRApartment : NSObject

//今年是否晋升
- (BOOL)willGetPromotionThisYear:(Employee*)employee;

@end
```

通过创建了两个分别专门处理薪水和晋升的部门，会计部门和人事部门的类：``FinancialApartment`` 和 ``HRApartment``，把两个任务（责任）分离了出去，让本该处理这些职责的类来处理这些职责。

这样一来，不仅仅在此次新需求中满足了单一职责原则，以后如果还要增加人事部门和会计部门处理的任务，就可以直接在这两个类里面添加即可。


下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践单一职责原则：

![未实践单一职责原则](http://oih3a9o4n.bkt.clouddn.com/SRP11.png)


实践了单一职责原则：
![实践了单一职责原则](http://oih3a9o4n.bkt.clouddn.com/SRP2.png)


>可以看到，在实践了单一职责原则的 UML 类图中，不属于``Employee``的两个职责被分类了``FinancialApartment``类 和 ``HRApartment``类。（在 UML 类图中，虚线箭头表示依赖关系，常用在方法参数等，由依赖方指向被依赖方）

上面说过除了类要遵循单一职责设计原则之外，在函数（方法）的设计上也要遵循单一职责的设计原则。因函数（方法）的单一职责原则理解起来比较容易，故在这里就不提供Demo和UML 类图了。

可以简单举一个例子：

APP的默认导航栏的样式是这样的：

- 白色底
- 黑色标题
- 底部有阴影

那么创建默认导航栏的伪代码可能是这样子的：

```objc
//默认样式的导航栏
- (void)createDefaultNavigationBarWithTitle:(NSString *)title{
    
    //create white color background view
    
    //create black color title
    
    //create shadow bottom
}
```

现在我们可以用这个方法统一创建默认的导航栏了。
但是过不久又有新的需求来了，有的页面的导航栏需要做成透明的，因此需要一个透明样式的导航栏：

- 透明底
- 白色标题
- 底部无阴影

针对这个需求，我们可以新增一个方法：

```objc
//透明样式的导航栏
- (void)createTransParentNavigationBarWithTitle:(NSString *)title{
    
    //create transparent color background view
    
    //create white color title
}
```

看出问题来了么？在这两个方法里面，创造background view和 title color title的方法的差别仅仅是颜色不同而已，而其他部分的代码是重复的。
因此我们应该将这两个方法抽出来：

```objc
//根据传入的颜色参数设置导航栏的背景色
- (void)createBackgroundViewWithColor:(UIColor)color;

//根据传入的标题字符串和颜色参数设置标题
- (void)createTitlewWithColorWithTitle:(NSString *)title color:(UIColor)color;
```

而且上面的制造阴影的部分也可以作为方法抽出来：

```objc
- (void)createShadowBottom;
```

这样一来，原来的两个方法可以写成：


```objc
//默认样式的导航栏
- (void)createDefaultNavigationBarWithTitle:(NSString *)title{
    
    //设置白色背景
    [self createBackgroundViewWithColor:[UIColor whiteColor]];
    
    //设置黑色标题
    [self createTitlewWithColorWithTitle:title color:[UIColor blackColor]];
    
    //设置底部阴影
    [self createShadowBottom];
}


//透明样式的导航栏
- (void)createTransParentNavigationBarWithTitle:(NSString *)title{
    
    //设置透明背景
    [self createBackgroundViewWithColor:[UIColor clearColor]];
    
    //设置白色标题
    [self createTitlewWithColorWithTitle:title color:[UIColor whiteColor]];
}
```

而且我们也可以将里面的方法拿出来在外面调用也可以：

设置默认样式的导航栏：
```objc
//设置白色背景
[navigationBar createBackgroundViewWithColor:[UIColor whiteColor]];

//设置黑色标题
[navigationBar createTitlewWithColorWithTitle:title color:[UIColor blackColor]];

//设置阴影
[navigationBar createShadowBottom];
```

设置透明样式的导航栏:
```objc
//设置透明色背景
[navigationBar createBackgroundViewWithColor:[UIColor clearColor]];

//设置白色标题
[navigationBar createTitlewWithColorWithTitle:title color:[UIColor whiteColor]];
```

这样一来，无论写在一个大方法里面调用或是分别在外面调用，都能很清楚地看到导航栏的每个元素是如何生成的，因为每个职责都分配到了一个单独的方法里面。而且还有一个好处是，透明导航栏如果遇到浅色背景的话，使用白色字体不如使用黑色字体好，所以遇到这种情况我们可以在``createTitlewWithColorWithTitle:color:``方法里面传入黑色色值。
而且今后可能还会有更多的导航栏样式，那么我们只需要分别改变传入的色值即可，不需要有大量的重复代码了，修改起来也很方便。



### 如何实践

对于上面的员工类的例子，或许是因为我们先入为主，知道一个公司的合理组织架构，觉得这么设计理所当然。但是在实际开发中，我们很容易会将不同的责任揉在一起，这点还是需要开发者注意的。




## 原则三：依赖倒置原则（Dependency Inversion Principle）

### 定义

>- Depend upon Abstractions. Do not depend upon concretions.
>- Abstractions should not depend upon details. Details should depend upon abstractions
>- High-level modules should not depend on low-level modules. Both should depend on abstractions.

即：

- 依赖抽象，而不是依赖实现。
- 抽象不应该依赖细节；细节应该依赖抽象。
- 高层模块不能依赖低层模块，二者都应该依赖抽象。


### 定义解读


- 针对接口编程，而不是针对实现编程。
- 尽量不要从具体的类派生，而是以继承抽象类或实现接口来实现。
- 关于高层模块与低层模块的划分可以按照决策能力的高低进行划分。业务层自然就处于上层模块，逻辑层和数据层自然就归类为底层。 



### 优点


通过抽象来搭建框架，建立类和类的关联，以减少类间的耦合性。而且以抽象搭建的系统要比以具体实现搭建的系统更加稳定，扩展性更高，同时也便于维护。



### 代码讲解

下面通过一个模拟项目开发的例子来讲解依赖倒置原则。

### 需求点

实现下面这样的需求：

用代码模拟一个实际项目开发的场景：前端和后端开发人员开发同一个项目。

#### 不好的设计

首先生成两个类，分别对应前端和后端开发者：

前端开发者：
```objc
//================== FrondEndDeveloper.h ==================

@interface FrondEndDeveloper : NSObject

- (void)writeJavaScriptCode;

@end



//================== FrondEndDeveloper.m ==================

@implementation FrondEndDeveloper

- (void)writeJavaScriptCode{
    NSLog(@"Write JavaScript code");
}

@end
```

后端开发者：

```objc
//================== BackEndDeveloper.h ==================

@interface BackEndDeveloper : NSObject

- (void)writeJavaCode;

@end



//================== BackEndDeveloper.m ==================

@implementation BackEndDeveloper

- (void)writeJavaCode{
    NSLog(@"Write Java code");
}
@end
```

这两个开发者分别对外提供了自己开发的方法：``writeJavaScriptCode``和``writeJavaCode``。

接着创建一个``Project``类：

```objc
//================== Project.h ==================

@interface Project : NSObject

//构造方法，传入开发者的数组
- (instancetype)initWithDevelopers:(NSArray *)developers;

//开始开发
- (void)startDeveloping;

@end



//================== Project.m ==================

#import "Project.h"
#import "FrondEndDeveloper.h"
#import "BackEndDeveloper.h"

@implementation Project
{
    NSArray *_developers;
}


- (instancetype)initWithDevelopers:(NSArray *)developers{
    
    if (self = [super init]) {
        _developers = developers;
    }
    return self;
}



- (void)startDeveloping{
    
    [_developers enumerateObjectsUsingBlock:^(id  _Nonnull developer, NSUInteger idx, BOOL * _Nonnull stop) {
        
        if ([developer isKindOfClass:[FrondEndDeveloper class]]) {
            
            [developer writeJavaScriptCode];
            
        }else if ([developer isKindOfClass:[BackEndDeveloper class]]){
            
            [developer writeJavaCode];
            
        }else{
            //no such developer
        }
    }];
}

@end
```

在``Project``类中，我们首先通过一个构造器方法，将开发者的数组传入``project``的实例对象。然后在开始开发的方法``startDeveloping``里面，遍历数组并判断元素类型的方式让不同类型的开发者调用和自己对应的函数。


思考一下，这样的设计有什么问题？

**问题一：**

假如后台的开发语言改成了GO语言，那么上述代码需要改动两个地方：

- ``BackEndDeveloper``:需要向外提供一个``writeGolangCode``方法。
- ``Project``类的``startDeveloping``方法里面需要将``BackEndDeveloper``类的``writeJavaCode``改成``writeGolangCode``。



**问题二：**

假如后期老板要求做移动端的APP（需要iOS和安卓的开发者），那么上述代码仍然需要改动两个地方：

- 还需要给``Project``类的构造器方法里面传入``IOSDeveloper``和``AndroidDeveloper``两个类。而且按照现有的设计，还要分别向外部提供``writeSwiftCode``和``writeKotlinCode``。
- ``Project``类的``startDeveloping``方法里面需要再多两个``elseif``判断，专门判断``IOSDeveloper``和``AndroidDeveloper``这两个类。

>开发安卓的代码也可以用Java，但是为了和后台的开发代码区分一下，这里用了同样可以开发安卓的Kotlin语言。


很显然，在这两种假设的场景下，**高层模块（Project）都依赖了低层模块（BackEndDeveloper）的改动，因此上述设计不符合依赖倒置原则**。

那么该如何设计才可以符合依赖倒置原则呢？

答案是**将开发者写代码的方法抽象出来，让Project类不再依赖所有低层的开发者类的具体实现，而是依赖抽象。而且从下至上，所有底层的开发者类也都依赖这个抽象，通过实现这个抽象来做自己的任务**。

这个抽象可以用接口，也可以用抽象类的方式来做，在这里笔者用**使用接口**的方式进行讲解：

#### 较好的设计

首先，创建一个接口，接口里面有一个写代码的方法``writeCode``：



```objc
//================== DeveloperProtocol.h ==================

@protocol DeveloperProtocol <NSObject>

- (void)writeCode;

@end
```

然后，让前端程序员和后端程序员类实现这个接口（遵循这个协议）并按照自己的方式实现：


前端程序员类：
```objc
//================== FrondEndDeveloper.h ==================

@interface FrondEndDeveloper : NSObject<DeveloperProtocol>
@end



//================== FrondEndDeveloper.m ==================

@implementation FrondEndDeveloper

- (void)writeCode{
    NSLog(@"Write JavaScript code");
}
@end
```


后端程序员类：
```objc
//================== BackEndDeveloper.h ==================

@interface BackEndDeveloper : NSObject<DeveloperProtocol>
@end



//================== BackEndDeveloper.m ==================
@implementation BackEndDeveloper

- (void)writeCode{
    NSLog(@"Write Java code");
}
@end
```


最后我们看一下新设计后的``Project``类：

```objc
//================== Project.h ==================

#import "DeveloperProtocol.h"

@interface Project : NSObject

//只需传入遵循DeveloperProtocol的对象数组即可
- (instancetype)initWithDevelopers:(NSArray <id <DeveloperProtocol>>*)developers;

//开始开发
- (void)startDeveloping;

@end


//================== Project.m ==================

#import "FrondEndDeveloper.h"
#import "BackEndDeveloper.h"

@implementation Project
{
    NSArray <id <DeveloperProtocol>>* _developers;
}


- (instancetype)initWithDevelopers:(NSArray <id <DeveloperProtocol>>*)developers{
    
    if (self = [super init]) {
        _developers = developers;
    }
    return self;
    
}


- (void)startDeveloping{
    
    //每次循环，直接向对象发送writeCode方法即可，不需要判断
    [_developers enumerateObjectsUsingBlock:^(id<DeveloperProtocol>  _Nonnull developer, NSUInteger idx, BOOL * _Nonnull stop) {
        
        [developer writeCode];
    }];
    
}

@end
```

新的``Project``的构造方法只需传入遵循DeveloperProtocol协议的对象构成的数组即可。这样也比较符合现实中的需求：只需要会写代码就可以加入到项目中。

而新的``startDeveloping``方法里：每次循环，直接向当前对象发送writeCode方法即可，不需要对程序员的类型做判断。因为这个对象一定是遵循``DeveloperProtocol``接口的，而遵循该接口的对象一定会实现``writeCode``方法（就算不实现也不会引起重大错误）。

现在新的设计接受完了，我们通过上面假设的两个情况来和之前的设计做个对比：

**假设1：后台的开发语言改成了GO语言**

在这种情况下，只需更改``BackEndDeveloper``类里面对于``DeveloperProtocol``接口的``writeCode``方法的实现即可：

```objc
//================== BackEndDeveloper.m ==================
@implementation BackEndDeveloper

- (void)writeCode{

    //Old：
    //NSLog(@"Write Java code");
    
    //New:
    NSLog(@"Write Golang code");
}
@end
```

而在``Project``里面不需要修改任何代码，因为``Project``类只依赖了接口方法``WriteCode``，没有依赖其具体的实现。

我们接着看一下第二个假设：

**假设2：后期老板要求做移动端的APP（需要iOS和安卓的开发者）**

在这个新场景下，我们只需要将新创建的两个开发者类：``IOSDeveloper``和``AndroidDeveloper``分别实现``DeveloperProtocol``接口的``writeCode``方法即可。

同样，``Project``的接口和实现代码都不用修改：客户端只需要在``Project``的构建方法的数组参数里面添加这两个新类的实例即可，不需要在``startDeveloping``方法里面添加类型判断，原因同上。

我们可以看到，新设计很好地在高层类（``Project``）与低层类（各种``developer``类）中间加了一层抽象，解除了二者在旧设计中的耦合，使得在低层类中的改动没有影响到高层类。


同样是抽象，新设计同样也可以用抽象类的方式：创建一个``Developer``的抽象类并提供一个``writeCode``方法，让不同的开发者类继承与它并按照自己的方式实现``writeCode``方法。这样一来，在``Project``类的构造方法就是传入已``Developer``类型为元素的数组了。有兴趣的小伙伴可以自己实现一下~ 


下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践依赖倒置原则：

![未实践依赖倒置原则](http://oih3a9o4n.bkt.clouddn.com/DIP1.png)


实践了依赖倒置原则：
![实践了依赖倒置原则](http://oih3a9o4n.bkt.clouddn.com/DIP22.png)



>在实践了依赖倒置原则的 UML 类图中，我们可以看到``Project``仅仅依赖于新的接口；而且低层的``FrondEndDevelope``和``BackEndDevelope``类按照自己的方式实现了这个接口：通过接口解除了原有的依赖。（在 UML 类图中，虚线三角箭头表示接口实线，由实现方指向接口）

### 如何实践

今后在处理高低层模块（类）交互的情景时，尽量将二者的依赖通过抽象的方式解除掉，实现方式可以是通过接口也可以是抽象类的方式。


## 原则四：接口分离原则（Interface Segregation Principle）


### 定义


>Many client specific interfaces are better than one general purpose interface.

即：多个特定的客户端接口要好于一个通用性的总接口。


### 定义解读

- 客户端不应该依赖它不需要实现的接口。
- 不建立庞大臃肿的接口，应尽量细化接口，接口中的方法应该尽量少。

需要注意的是：接口的粒度也不能太小。如果过小，则会造成接口数量过多，使设计复杂化。


### 优点

避免同一个接口里面包含不同类职责的方法，接口责任划分更加明确，符合高内聚低耦合的思想。


### 代码讲解

下面通过一个餐厅服务的例子讲解一下接口分离原则。

#### 需求点

现在的餐厅除了提供传统的店内服务，多数也都支持网上下单，网上支付功能。写一些接口方法来涵盖餐厅的所有的下单及支付功能。

#### 不好的设计

```objc
//================== RestaurantProtocol.h ==================

@protocol RestaurantProtocol <NSObject>

- (void)placeOnlineOrder;         //下订单：online
- (void)placeTelephoneOrder;      //下订单：通过电话
- (void)placeWalkInCustomerOrder; //下订单：在店里

- (void)payOnline;                //支付订单：online
- (void)payInPerson;              //支付订单：在店里支付

@end
```

在这里声明了一个接口，它包含了下单和支付的几种方式：

- 下单：
  - online下单
  - 电话下单
  - 店里下单（店内服务）

- 支付
  - online支付（适用于online下单和电话下单的顾客）
  - 店里支付（店内服务）

>这里先不讨论电话下单的顾客是用online支付还是店内支付。

对应的，我们有三种下单方式的顾客：

1.online下单，online支付的顾客

```objc
//================== OnlineClient.h ==================

#import "RestaurantProtocol.h"

@interface OnlineClient : NSObject<RestaurantProtocol>
@end



//================== OnlineClient.m ==================

@implementation OnlineClient

- (void)placeOnlineOrder{
    NSLog(@"place on line order");
}

- (void)placeTelephoneOrder{
    //not necessarily
}

- (void)placeWalkInCustomerOrder{
    //not necessarily
}

- (void)payOnline{
    NSLog(@"pay on line");
}

- (void)payInPerson{
    //not necessarily
}
@end

```

2.电话下单，online支付的顾客

```objc
//================== TelephoneClient.h ==================

#import "RestaurantProtocol.h"

@interface TelephoneClient : NSObject<RestaurantProtocol>
@end



//================== TelephoneClient.m ==================

@implementation TelephoneClient

- (void)placeOnlineOrder{
    //not necessarily
}

- (void)placeTelephoneOrder{
    NSLog(@"place telephone order");
}

- (void)placeWalkInCustomerOrder{
    //not necessarily
}

- (void)payOnline{
    NSLog(@"pay on line");
}

- (void)payInPerson{
    //not necessarily
}

@end
```

3.在店里下单并支付的顾客：

```objc
//================== WalkinClient.h ==================

#import "RestaurantProtocol.h"

@interface WalkinClient : NSObject<RestaurantProtocol>
@end



//================== WalkinClient.m ==================

@implementation WalkinClient

- (void)placeOnlineOrder{
   //not necessarily
}

- (void)placeTelephoneOrder{
    //not necessarily
}

- (void)placeWalkInCustomerOrder{
    NSLog(@"place walk in customer order");
}

- (void)payOnline{
   //not necessarily
}

- (void)payInPerson{
    NSLog(@"pay in person");
}

@end
```

我们发现，并不是所有顾客都必须要实现``RestaurantProtocol``里面的所有方法。**由于接口方法的设计造成了冗余，因此该设计不符合接口隔离原则**。

>注意，Objective-C中的协议可以通过``@optional``关键字设置不需要必须实现的方法，该特性不与接口分离原则冲突：只要属于同一类责任的接口，都可以放入同一接口中。

那么如何做才符合接口隔离原则呢？我们来看一下较好的设计。

#### 较好的设计

要符合接口隔离原则，只需要将不同类型的接口分离出来即可。我们将原来的``RestaurantProtocol``接口拆分成两个接口：下单接口和支付接口。


下单接口：

```objc
//================== RestaurantPlaceOrderProtocol.h ==================

@protocol RestaurantPlaceOrderProtocol <NSObject>

- (void)placeOrder;

@end
```


支付接口：

```objc
//================== RestaurantPaymentProtocol.h ==================

@protocol RestaurantPaymentProtocol <NSObject>

- (void)payOrder;

@end
```

现在有了下单接口和支付接口，我们就可以让不同的客户来以自己的方式实现下单和支付操作了：

首先创建一个所有客户的父类，来遵循这个两个接口：

```objc
//================== Client.h ==================

#import "RestaurantPlaceOrderProtocol.h"
#import "RestaurantPaymentProtocol.h"

@interface Client : NSObject<RestaurantPlaceOrderProtocol,RestaurantPaymentProtocol>
@end
```

接着另online下单，电话下单，店内下单的顾客继承这个父类，分别实现这两个接口的方法：

1.online下单，online支付的顾客

```objc
//================== OnlineClient.h ==================

#import "Client.h"
@interface OnlineClient : Client
@end



//================== OnlineClient.m ==================

@implementation OnlineClient

- (void)placeOrder{
    NSLog(@"place on line order");
}

- (void)payOrder{
    NSLog(@"pay on line");
}

@end
```

2.电话下单，online支付的顾客

```objc
//================== TelephoneClient.h ==================
#import "Client.h"
@interface TelephoneClient : Client
@end



//================== TelephoneClient.m ==================
@implementation TelephoneClient

- (void)placeOrder{
    NSLog(@"place telephone order");
}

- (void)payOrder{
    NSLog(@"pay on line");
}

@end
```

3.在店里下单并支付顾客：

```objc
//================== WalkinClient.h ==================

#import "Client.h"
@interface WalkinClient : Client
@end



//================== WalkinClient.m ==================

@implementation WalkinClient

- (void)placeOrder{
    NSLog(@"place walk in customer order");
}

- (void)payOrder{
    NSLog(@"pay in person");
}

@end
```

因为我们把不同职责的接口拆开，使得接口的责任更加清晰，简洁明了。不同的客户端可以根据自己的需求遵循所需要的接口来以自己的方式实现。

而且今后如果还有和下单或者支付相关的方法，也可以分别加入到各自的接口中，避免了接口的臃肿，同时也提高了程序的内聚性。

下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践接口分离原则：
![未实践接口分离原则](http://oih3a9o4n.bkt.clouddn.com/ISP1.png)


实践了接口分离原则：
![实践了接口分离原则](http://oih3a9o4n.bkt.clouddn.com/ISP2.png)

>通过遵守接口分离原则，接口的设计变得更加简洁，而且各种客户类不需要实现自己不需要实现的接口。

### 如何实践

在设计接口时，尤其是在向现有的接口添加方法时，我们需要仔细斟酌这些方法是否是处理同一类任务的：如果是则可以放在一起；如果不是则需要做拆分。

做iOS开发的朋友对``UITableView``的``UITableViewDelegate``和``UITableViewDataSource``这两个协议应该会非常熟悉。这两个协议里的方法都是与``UITableView``相关的，但iOS SDK的设计者却把这些方法放在不同的两个协议中。原因就是这两个协议所包含的方法所处理的任务是不同的两种：

- ``UITableViewDelegate``：含有的方法是``UITableView``的实例告知其代理一些点击事件的方法，即**事件的传递**，方向是从``UITableView``的实例到其代理。
- ``UITableViewDataSource``：含有的方法是``UITableView``的代理传给``UITableView``一些必要数据供``UITableView``展示出来，即**数据的传递**，方向是从``UITableView``的代理到``UITableView``。

很显然，``UITableView``协议的设计者很好地实践了接口分离的原则，值得我们大家学习。


## 原则五：迪米特法则（Law of Demeter）


### 定义

>You only ask for objects which you directly need.

即：一个对象应该对尽可能少的对象有接触，也就是只接触那些真正需要接触的对象。

### 定义解读


- 迪米特法则也叫做最少知道原则（Least Know Principle）， 一个类应该只和它的成员变量，方法的输入，返回参数中的类作交流，而不应该引入其他的类（间接交流）。


### 优点

实践迪米特法则可以良好地降低类与类之间的耦合，减少类与类之间的关联程度，让类与类之间的协作更加直接。


### 代码讲解

下面通过一个简单的关于汽车的例子来讲解一下迪米特法则。

#### 需求点

设计一个汽车类，包含汽车的品牌名称，引擎等成员变量。提供一个方法返回引擎的品牌名称。


#### 不好的设计

Car类：

```objc

//================== Car.h ==================

@class GasEngine;

@interface Car : NSObject

//构造方法
- (instancetype)initWithEngine:(GasEngine *)engine;

//返回私有成员变量：引擎的实例
- (GasEngine *)usingEngine;

@end




//================== Car.m ==================

#import "Car.h"
#import "GasEngine.h"

@implementation Car
{
    GasEngine *_engine;
}

- (instancetype)initWithEngine:(GasEngine *)engine{
    
    self = [super init];
    
    if (self) {
        _engine = engine;
    }
    return self;
}

- (GasEngine *)usingEngine{
    
    return _engine;
}

@end
```

从上面可以看出，Car的构造方法需要传入一个引擎的实例对象。而且因为引擎的实例对象被赋到了Car对象的私有成员变量里面。所以Car类给外部提供了一个返回引擎对象的方法：``usingEngine``。

而这个引擎类``GasEngine``有一个品牌名称的成员变量``brandName``：

```objc
//================== GasEngine.h ==================
@interface GasEngine : NSObject

@property (nonatomic, copy) NSString *brandName;

@end

```

这样一来，客户端就可以拿到引擎的品牌名称了：

```objc
//================== Client.m ==================

#import "GasEngine.h"
#import "Car.h"

- (NSString *)findCarEngineBrandName:(Car *)car{

    GasEngine *engine = [car usingEngine];
    NSString *engineBrandName = engine.brandName;//获取到了引擎的品牌名称
    return engineBrandName;
}
```

上面的设计完成了需求，但是却违反了迪米特法则。原因是**在客户端的**``findCarEngineBrandName:``**中引入了和入参（Car）和返回值（NSString）无关的**``GasEngine``**对象**。增加了客户端与
``GasEngine``的耦合。而这个耦合显然是不必要更是可以避免的。

接下来我们看一下如何设计可以避免这种耦合：

#### 较好的设计

同样是Car这个类，我们去掉原有的返回引擎对象的方法，而是增加一个直接返回引擎品牌名称的方法：


```objc
//================== Car.h ==================

@class GasEngine;

@interface Car : NSObject

//构造方法
- (instancetype)initWithEngine:(GasEngine *)engine;

//直接返回引擎品牌名称
- (NSString *)usingEngineBrandName;

@end


//================== Car.m ==================

#import "Car.h"
#import "GasEngine.h"

@implementation Car
{
    GasEngine *_engine;
}

- (instancetype)initWithEngine:(GasEngine *)engine{
    
    self = [super init];
    
    if (self) {
        _engine = engine;
    }
    return self;
}


- (NSString *)usingEngineBrandName{
    return _engine.brand;
}

@end
```

因为直接``usingEngineBrandName``直接返回了引擎的品牌名称，所以在客户端里面就可以直接拿到这个值，而不需要间接地通过原来的``GasEngine``实例来获取。

我们看一下客户端操作的变化：


```objc
//================== Client.m ==================

#import "Car.h"

- (NSString *)findCarEngineBrandName:(Car *)car{
    
    NSString *engineBrandName = [car usingEngineBrandName]; //直接获取到了引擎的品牌名称
    return engineBrandName;
}
```

与之前的设计不同，在客户端里面，没有引入``GasEngine``类，而是直接通过``Car``实例获取到了需要的数据。

这样设计的好处是，如果这辆车的引擎换成了电动引擎(原来的``GasEngine``类换成了``ElectricEngine``类)，**客户端代码可以不做任何修改**！因为它没有引入任何引擎类，而是直接获取了引擎的品牌名称。

所以在这种情况下我们只需要修改Car类的``usingEngineBrandName``方法实现，将新引擎的品牌名称返回即可。

下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践迪米特法则：
![未实践迪米特法则](http://oih3a9o4n.bkt.clouddn.com/LOD111.png)



实践了迪米特法则：
![实践了迪米特法则](http://oih3a9o4n.bkt.clouddn.com/LOD2.png)

>很明显，在实践了迪米特法则的 UML 类图里面，没有了``Client``对``GasEngine``的依赖，耦合性降低。

### 如何实践

今后在做对象与对象之间交互的设计时，应该极力避免引出中间对象的情况（需要导入其他对象的类）：需要什么对象直接返回即可，降低类之间的耦合度。

## 原则六：里氏替换原则（Liskov Substitution Principle）


### 定义

>In a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e. an object of type T may be substituted with any object of a subtype S) without altering any of the desirable properties of the program (correctness, task performed, etc.)


即：所有引用基类的地方必须能透明地使用其子类的对象，也就是说子类对象可以替换其父类对象，而程序执行效果不变。


### 定义的解读

在继承体系中，子类中可以增加自己特有的方法，也可以实现父类的抽象方法，但是不能重写父类的非抽象方法，否则该继承关系就不是一个正确的继承关系。


### 优点

可以检验继承使用的正确性，约束继承在使用上的泛滥。

### 代码讲解

在这里用一个简单的长方形与正方形的例子讲解一下里氏替换原则。

#### 需求点

创建两个类：长方形和正方形，都可以设置宽高（边长），也可以输出面积大小。

#### 不好的设计

首先声明一个长方形类，然后让正方形类继承于长方形。

长方形类：

```objc
//================== Rectangle.h ==================

@interface Rectangle : NSObject
{
@protected double _width;
@protected double _height;
}

//设置宽高
- (void)setWidth:(double)width;
- (void)setHeight:(double)height;

//获取宽高
- (double)width;
- (double)height;

//获取面积
- (double)getArea;

@end



//================== Rectangle.m ==================

@implementation Rectangle

- (void)setWidth:(double)width{
    _width = width;
}

- (void)setHeight:(double)height{
    _height = height;
}

- (double)width{
    return _width;
}

- (double)height{
    return _height;
}


- (double)getArea{
    return _width * _height;
}

@end
```

正方形类：


```objc
//================== Square.h ==================

@interface Square : Rectangle
@end



//================== Square.m ==================

@implementation Square

- (void)setWidth:(double)width{
    
    _width = width;
    _height = width;
}

- (void)setHeight:(double)height{
    
    _width = height;
    _height = height;
}

@end
```

可以看到，正方形类继承了长方形类以后，为了保证边长永远是相等的，特意在两个set方法里面强制将宽和高都设置为传入的值，也就是重写了父类``Rectangle``的两个set方法。但是里氏替换原则里规定，子类不能重写父类的方法，所以上面的设计是违反该原则的。

而且里氏替换原则原则里面所属：子类对象能够替换父类对象，而程序执行效果不变。我们通过一个例子来看一下上面的设计是否符合：

在客户端类写一个方法：传入一个``Rectangle``类型并返回它的面积：

```objc
- (double)calculateAreaOfRect:(Rectangle *)rect{
    return rect.getArea;
}
```

我们先用``Rectangle``对象试一下：

```objc
Rectangle *rect = [[Rectangle alloc] init];
rect.width = 10;
rect.height = 20;
    
double rectArea = [self calculateAreaOfRect:rect];//output:200
```

长宽分别设置为10，20以后，结果输出200，没有问题。

现在我们使用``Rectange``的子类``Square``的对象替换原来的``Rectange``对象，看一下结果如何：

```objc
Square *square = [[Square alloc] init];
square.width = 10;
square.height = 20;
    
double squareArea = [self calculateAreaOfRect:square];//output:400
```

结果输出为400，结果不一致，再次说明了上述设计不符合里氏替换原则，因为子类的对象``square``替换父类的对象``rect``以后，程序执行的结果变了。

不符合里氏替换原则就说明该继承关系不是正确的继承关系，也就是说正方形类不能继承于长方形类，程序需要重新设计。

我们现在看一下比较好的设计。

#### 较好的设计


既然正方形不能继承于长方形，那么是否可以让二者都继承于其他的父类呢？答案是可以的。

既然要继承于其他的父类，它们这个父类肯定具备这两种形状共同的特点：有4个边。那么我们就定义一个四边形的类：``Quadrangle``。

```objc
//================== Quadrangle.h ==================

@interface Quadrangle : NSObject
{
@protected double _width;
@protected double _height;
}

- (void)setWidth:(double)width;
- (void)setHeight:(double)height;

- (double)width;
- (double)height;

- (double)getArea;
@end
```

接着，让``Rectangle``类和``Square``类继承于它：



``Rectangle``类：
```objc
//================== Rectangle.h ==================

#import "Quadrangle.h"

@interface Rectangle : Quadrangle

@end



//================== Rectangle.m ==================

@implementation Rectangle

- (void)setWidth:(double)width{
    _width = width;
}

- (void)setHeight:(double)height{
    _height = height;
}

- (double)width{
    return _width;
}

- (double)height{
    return _height;
}


- (double)getArea{
    return _width * _height;
}

@end
```


``Square``类：

```objc
//================== Square.h ==================

@interface Square : Quadrangle
{
    @protected double _sideLength;
}

-(void)setSideLength:(double)sideLength;

-(double)sideLength;

@end



//================== Square.m ==================

@implementation Square

-(void)setSideLength:(double)sideLength{    
    _sideLength = sideLength;
}

-(double)sideLength{
    return _sideLength;
}

- (void)setWidth:(double)width{
    _sideLength = width;
}

- (void)setHeight:(double)height{
    _sideLength = height;
}

- (double)width{
    return _sideLength;
}

- (double)height{
    return _sideLength;
}


- (double)getArea{
    return _sideLength * _sideLength;
}

@end
```

我们可以看到，``Rectange``和``Square``类都以自己的方式实现了父类``Quadrangle``的公共方法。而且由于``Square``的特殊性，它也声明了自己独有的成员变量``_sideLength``以及其对应的公共方法。

>注意，这里``Rectange``和``Square``并不是重写了其父类的公共方法，而是实现了其抽象方法。




下面来看一下这两个设计的UML 类图，可以更形象地看出两种设计上的区别：


#### UML 类图对比

未实践里氏替换原则：
![未实践里氏替换原则](http://oih3a9o4n.bkt.clouddn.com/LSP11.png)



实践了里氏替换原则：
![实践了里氏替换原则](http://oih3a9o4n.bkt.clouddn.com/LSP22.png)




 
### 如何实践

里氏替换原则是对继承关系的一种检验：检验是否真正符合继承关系，以避免继承的滥用。因此，在使用继承之前，需要反复思考和确认该继承关系是否正确，或者当前的继承体系是否还可以支持后续的需求变更，如果无法支持，则需要及时重构，采用更好的方式来设计程序。


# 最后的话


到这里关于六大设计原则的讲解已经结束了。本篇文章所展示的Demo和UML 类图都在笔者维护的一个专门的GitHub库中：[object-oriented-design](https://github.com/knightsj/object-oriented-design)。想看Demo和UML图的同学可以点击链接查看。欢迎fork，更欢迎给出更多语言的不同例子的PR~ 而且后续还会添加关于设计模式的 代码和 UML 类图。

关于这几个设计原则还有最后一点需要强调的是：
设计原则是设计模式的基石，但是很难在使实际开发中的某个设计中全部都满足这些设计原则。因此我们需要抓住具体设计场景的特殊性，有选择地遵循最合适的设计原则。




----


笔者在近期开通了个人公众号，主要分享编程，读书笔记，思考类的文章。

- **编程类**文章：包括笔者以前发布的精选技术文章，以及后续发布的技术文章（以原创为主），并且逐渐脱离 iOS 的内容，将侧重点会转移到**提高编程能力**的方向上。
- **读书笔记类**文章：分享**编程类**，**思考类**，**心理类**，**职场类**书籍的读书笔记。
- **思考类**文章：分享笔者平时在**技术上**，**生活上**的思考。

>因为公众号每天发布的消息数有限制，所以到目前为止还没有将所有过去的精选文章都发布在公众号上，后续会逐步发布的。

**而且因为各大博客平台的各种限制，后面还会在公众号上发布一些短小精干，以小见大的干货文章哦~**

扫下方的公众号二维码并点击关注，期待与您的共同成长~

![公众号：程序员维他命](http://upload-images.jianshu.io/upload_images/859001-5bddfacafb9e9079.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




