---
title: 面向对象设计的设计模式（一）：创建型模式（附 Demo 及 UML 类图）
tags: [iOS,Objectice-C,Object-Oriented,Design Pattern]
categories: Object-Oriented
---

![](http://jknight-blog.oss-cn-shanghai.aliyuncs.com/design-pattern-creation/odd_dp2_banner.png)

继上一篇的[面向对象设计的六大设计原则（附 Demo 及 UML 类图）](https://knightsj.github.io/2018/09/09/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E7%9A%84%E5%85%AD%E5%A4%A7%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99%EF%BC%88%E9%99%84%20Demo%20%E5%8F%8A%20UML%20%E7%B1%BB%E5%9B%BE%EF%BC%89/)，本篇是面向对象设计系列的第二个部分：面向对象设计的设计模式的第一篇文章。



最开始说一下什么是设计模式。关于设计模式的概念，有很多不同的版本，在这里说一下我个人比较赞同的一个说法：

> 设计模式用于在特定的条件下为一些重复出现的软件设计问题提供合理的、有效的解决方案。

去掉一些定语的修饰，这句话精简为：

> 设计模式为问题提供方案。



简单来看，设计模式其实就是针对某些问题的一些方案。在软件开发中，即使很多人在用不同的语言去开发不同的业务，但是很多时候这些人遇到的问题抽象出来都是相似的。一些卓越的开发者将一些常出现的问题和对应的解决方案汇总起来，总结出了这些设计模式。

因此掌握了这些设计模式，可以让我们更好地去解决开发过程中遇到的一些常见问题。而且对这些问题的解决方案的掌握程度越好，我们就越能够打破语言本身的限制去解决问题，也就是增强“软件开发的内功”。



介绍设计模式最著名的一本书莫属《设计模式 可复用面向对象软件的基础》这本书，书中共介绍了23个设计模式。而这些设计模式分为三大类：



- **创建型**设计模式：侧重于对象的创建。
- **结构型**设计模式：侧重于接口的设计和系统的结构。
- **行为型**设计模式：侧重于类或对象的行为。

<!-- more -->


而本篇作为该系列的第一篇，讲解的是设计模式中的6个创建型设计模式：

1. 简单工厂模式（Simple Factory Pattern）
2. 工厂方法模式（Factory Method Pattern）
3. 抽象工厂模式（Abstract Factory Pattern）
4. 单例模式（Singleton Pattern）
5. 生成器模式（Builder Pattern）
6. 原型模式（Prototype Pattern）

> 注意：简单工厂模式不是 GoF总结出来的23种设计模式之一，不存在于《设计模式 可复用面向对象软件的基础》这本书中。



在面向对象设计中，类与对象几乎是构成所有系统的基本元素，因此我认为学好了创建型模式才是学会设计系统的第一步：因为你应该知道如何去创建一些特定性质的对象，这才是设计好的系统的开始。

在讲解这6个设计模式之前先说一下该系列文章的讲解方式：

从更多维度来理解一件事物有助于更深刻地理解它，因此每个设计模式我都会从以下这几点来讲解：

- 定义
- 使用场景
- 成员与类图
- 代码示例
- 优点
- 缺点
- iOS SDK 和 JDK中的应用

> 最后一项：“iOS SDK 和 JDK中的应用”讲解的是该设计模式在Objective-C和java语言（JDK）中的应用。



首先我们看一下简单工厂模式：



# 一. 简单工厂模式



## 定义

> 简单工厂模式(Simple Factory Pattern)：专门定义一个类（工厂类）来负责创建其他类的实例。可以根据创建方法的参数来返回不同类的实例，被创建的实例通常都具有共同的父类。



简单工厂模式又称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。



## 适用场景

如果我们希望将一些为数不多的类似的对象的创建和他们的创建细节分离开，也不需要知道对象的具体类型，可以使用简单工厂模式。



举个形象点的例子：在前端开发中，常常会使用外观各式各样的按钮：比如有的按钮有圆角，有的按钮有阴影，有的按钮有边框，有的按钮无边框等等。但是因为同一种样式的按钮可以出现在项目的很多地方，所以如果在每个地方都把创建按钮的逻辑写一遍的话显然是会造成代码的重复（而且由于业务的原因有的按钮的创建逻辑能比较复杂，代码量大）。



那么为了避免重复代码的产生，我们可以将这些创建按钮的逻辑都放在一个“工厂”里面，让这个工厂来根据你的需求（传入的参数）来创建对应的按钮并返回给你。这样一来，同样类型的按钮在多个地方使用的时候，就可以只给这个工厂传入其对应的参数并拿到返回的按钮即可。



下面来看一下简单工厂模式的成员和类图。



## 成员与类图



### 成员

简单工厂模式的结构比较简单，一共只有三个成员：

- 工厂（Factory）：**工厂**负责实现创建所有产品实例的逻辑
- 抽象产品（Product）：**抽象产品**是工厂所创建的所有产品对象的父类，负责声明所有产品实例所共有的公共接口。
- 具体产品（Concrete Product）：**具体产品**是工厂所创建的所有产品对象类，它以自己的方式来实现其共同父类声明的接口。

下面通过类图来看一下各个成员之间的关系：



### 模式类图

![简单工厂模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_sfp_1.png)

> 从类图中可以看出，工厂类提供一个静态方法：通过传入的字符串来制造其所对应的产品。



## 代码示例



### 场景概述

举一个店铺售卖不同品牌手机的例子：店铺，即客户端类向手机工厂购进手机售卖。



### 场景分析

该场景可以使用简单工厂的角色来设计：

- 抽象产品：``Phone``，是所有具体产品类的父类，提供一个公共接口``packaging``表示手机的装箱并送到店铺。
- 具体产品：不同品牌的手机，iPhone手机类（``IPhone``），小米手机类（``MIPhone``），华为手机类（``HWPhone``）。
- 工厂：``PhoneFactory``根据不同的参数来创建不同的手机。
- 客户端类：店铺类``Store``负责售卖手机。



### 代码实现

抽象产品类``Phone``：

```objc
//================== Phone.h ==================
@interface Phone : NSObject

//package to store
- (void)packaging;

@end
```

具体产品类 ``IPhone``：

```objc
//================== IPhone.h ==================
@interface IPhone : Phone

@end


//================== IPhone.m ==================
@implementation IPhone

- (void)packaging{
    NSLog(@"IPhone has been packaged");
}

@end
```

具体产品类 ``MIPhone``：

```objc
//================== MIPhone.h ==================
@interface MIPhone : Phone

@end



//================== MIPhone.m ==================
@implementation MIPhone

- (void)packaging{
    NSLog(@"MIPhone has been packaged");
}

@end
```

具体产品类：``HWPhone``:

```objc
//================== HWPhone.h ==================
@interface HWPhone : Phone

@end



//================== HWPhone.m ==================
@implementation HWPhone

- (void)packaging{
    NSLog(@"HUAWEI Phone has been packaged");
}

@end
```

以上是抽象产品类以及它的三个子类：苹果手机类，小米手机类和华为手机类。
下面看一下工厂类 ``PhoneFactory``：

```objc
//================== PhoneFactory.h ==================
@interface PhoneFactory : NSObject

+ (Phone *)createPhoneWithTag:(NSString *)tag;

@end


//================== PhoneFactory.m ==================
#import "IPhone.h"
#import "MIPhone.h"
#import "HWPhone.h"

@implementation PhoneFactory

+ (Phone *)createPhoneWithTag:(NSString *)tag{
    
    if ([tag isEqualToString:@"i"]) {
        
        IPhone *iphone = [[IPhone alloc] init];
        return iphone;
        
    }else if ([tag isEqualToString:@"MI"]){
        
        MIPhone *miPhone = [[MIPhone alloc] init];
        return miPhone;
        
    }else if ([tag isEqualToString:@"HW"]){
        
        HWPhone *hwPhone = [[HWPhone alloc] init];
        return hwPhone;
        
    }else{
        
        return nil;
    }
}

@end
```

> 工厂类向外部（客户端）提供了一个创造手机的接口``createPhoneWithTag:``，根据传入参数的不同可以返回不同的具体产品类。因此**客户端只需要知道它所需要的产品所对应的参数即可获得对应的产品了**。

在本例中，我们声明了店铺类 ``Store``为客户端类：

```objc
//================== Store.h ==================
#import "Phone.h"

@interface Store : NSObject

- (void)sellPhone:(Phone *)phone;

@end


//================== Store.m ==================
@implementation Store

- (void)sellPhone:(Phone *)phone{
    NSLog(@"Store begins to sell phone:%@",[phone class]);
}

@end
```

> 客户端类声明了一个售卖手机的接口``sellPhone:``。表示它可以售卖作为参数所传入的手机。

最后我们用代码模拟一下这个实际场景：

```objc
//================== Using by client ==================


//1. A phone store  wants to sell iPhone
Store *phoneStore = [[Store alloc] init];
    
//2. create phone
Phone *iPhone = [PhoneFactory  createPhoneWithTag:@"i"];
    
//3. package phone to store
[iphone packaging];
    
//4. store sells phone after receving it
[phoneStore sellPhone:iphone];
```

上面代码的解读：

1. 最开始实例化一个商店，商店打算卖苹果手机
2. 商店委托工厂给他制作一台iPhone手机，传入对应的字段``i``。
3. 手机生产好以后打包送到商店
4. 商店售卖手机



在这里我们需要注意的是：商店从工厂拿到手机不需要了解手机制作的过程，只需要知道它要工厂做的是手机（只知道``Phone``类即可），和需要给工厂类传入它所需手机所对应的参数即可（这里的iPhone手机对应的参数就是``i``）。

下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![简单工厂模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_sfp_2.png)



## 优点

- 客户端只需要给工厂类传入一个正确的（约定好的）参数，就可以获取你所需要的对象，而不需要知道其创建细节，一定程度上减少系统的耦合。
- 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可，减少开发者的记忆成本。



## 缺点

- 如果业务上添加新产品的话，就需要修改工厂类原有的判断逻辑，这其实是违背了开闭原则的。
- 在产品类型较多时，有可能造成工厂逻辑过于复杂。所以简单工厂模式比较适合产品种类比较少而且增多的概率很低的情况。



## iOS SDK 和 JDK中的应用

- Objective-C中的类簇就是简单工厂设计模式的一个应用。如果给``NSNumber``的工厂方法传入不同类型的数据，则会返回不同数据所对应的``NSNumber``的子类。
- JDK中的``Calendar``类中的私有的``createCalendar(TimeZone zone, Locale aLocale)``方法通过不同的入参来返回不同类型的Calendar子类的实例。



# 二. 工厂方法模式



## 定义

> 工厂方法模式(Factory Method Pattern)又称为工厂模式，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，即通过不同的工厂子类来创建不同的产品对象。







## 适用场景

工厂方法模式的适用场景与简单工厂类似，都是创建数据和行为比较类似的对象。但是和简单工厂不同的是：在工厂方法模式中，因为创建对象的责任移交给了抽象工厂的子类，因此客户端需要知道其所需产品所对应的工厂子类，而不是简单工厂中的参数。



下面我们看一下工厂方法模式的成员和类图。



## 成员与类图



### 成员

工厂方法模式包含四个成员：

1. 抽象工厂（Abstract Factory）：**抽象工厂**负责声明具体工厂的创建产品的接口。
2. 具体工厂（Concrete Factory）：**具体工厂**负责创建产品。
3. 抽象产品（Abstract Product）：**抽象产品**是工厂所创建的所有产品对象的父类，负责声明所有产品实例所共有的公共接口。
4. 具体产品（Concrete Product）：**具体产品**是工厂所创建的所有产品对象类，它以自己的方式来实现其共同父类声明的接口。

下面通过类图来看一下各个成员之间的关系：



### 模式类图

![工厂方法模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_fmp_1.png)

从类图中我们可以看到：抽象工厂负责定义具体工厂必须实现的接口，而创建产品对象的任务则交给具体工厂，由特定的子工厂来创建其对应的产品。

这使得工厂方法模式可以允许系统在不修改原有工厂的情况下引进新产品：只需要创建新产品类和其所对应的工厂类即可。



## 代码示例



### 场景概述

同样也是模拟上面的简单工厂例子中的场景（手机商店卖手机），但是由于这次是由工厂方法模式来实现的，因此在代码设计上会有变化。



### 场景分析

与简单工厂模式不同的是：简单工厂模式里面只有一个工厂，而工厂方法模式里面有一个抽象工厂和继承于它的具体工厂。

因此同样的三个品牌的手机，我们可以通过三个不同的具体工厂：苹果手机工厂（``IPhoneFactory``），小米手机工厂   (``MIPhoneFactory``)，华为手机工厂（``HWPhoneFactory``）来生产。而这些具体工厂类都会继承于抽象手机工厂类：``PhoneFactory``，它来声明生产手机的接口。

下面我们用代码来具体来看一下工厂类（抽象工厂和具体工厂）的设计：



### 代码实现

首先我们声明一个抽象工厂类 ``PhoneFactory``：

```objc
//================== PhoneFactory.h ==================
#import "Phone.h"

@interface PhoneFactory : NSObject

+ (Phone *)createPhone;

@end


//================== PhoneFactory.m ==================
@implementation PhoneFactory

+ (Phone *)createPhone{
    //implemented by subclass
    return nil;
}

@end
```

> 抽象工厂类给具体工厂提供了生产手机的接口，因此不同的具体工厂可以按照自己的方式来生产手机。下面看一下具体工厂：

苹果手机工厂 ``IPhoneFactory``

```objc
//================== IPhoneFactory.h ==================
@interface IPhoneFactory : PhoneFactory
@end


//================== IPhoneFactory.m ==================
#import "IPhone.h"

@implementation IPhoneFactory

+ (Phone *)createPhone{
    
    IPhone *iphone = [[IPhone alloc] init];
    NSLog(@"iPhone has been created");
    return iphone;
}

@end
```

小米手机工厂 ``MIPhoneFactory``：

```objc
//================== MIPhoneFactory.h ==================
@interface MPhoneFactory : PhoneFactory

@end



//================== MIPhoneFactory.m ==================
#import "MiPhone.h"

@implementation MPhoneFactory

+ (Phone *)createPhone{
    
    MiPhone *miPhone = [[MiPhone alloc] init];
    NSLog(@"MIPhone has been created");
    return miPhone;
}

@end
```

华为手机工厂  ``HWPhoneFactory``:

```objc
//================== HWPhoneFactory.h ==================
@interface HWPhoneFactory : PhoneFactory

@end



//================== HWPhoneFactory.m ==================
#import "HWPhone.h"

@implementation HWPhoneFactory

+ (Phone *)createPhone{
    
    HWPhone *hwPhone = [[HWPhone alloc] init];
    NSLog(@"HWPhone has been created");
    return hwPhone;
}

@end
```

以上就是声明的抽象工厂类和具体工厂类。因为生产手机的责任分配给了各个具体工厂类，因此客户端只需要委托所需手机所对应的工厂就可以获得其生产的手机了。

> 因为抽象产品类``Phone``和三个具体产品类（``IPhone``，``MIPhone``，``HWPhone``）和简单工厂模式中介绍的例子中的一样，因此这里就不再重复介绍了。

下面我们用代码模拟一下该场景：

```objc
//================== Using by client ==================


//A phone store
Store *phoneStore = [[Store alloc] init];
    
//phoneStore wants to sell iphone
Phone *iphone = [IPhoneFactory  createPhone];
[iphone packaging];
[phoneStore sellPhone:iphone];
    
    
//phoneStore wants to sell MIPhone
Phone *miPhone = [MPhoneFactory createPhone];
[miPhone packaging];
[phoneStore sellPhone:miPhone];
    
//phoneStore wants to sell HWPhone
Phone *hwPhone = [HWPhoneFactory createPhone];
[hwPhone packaging];
[phoneStore sellPhone:hwPhone];
```

由上面的代码可以看出：客户端``phoneStore``只需委托iPhone，MIPhone，HWPhone对应的工厂即可获得对应的手机了。

而且以后如果增加其他牌子的手机，例如魅族手机，就可以声明一个魅族手机类和魅族手机的工厂类并实现``createPhone``这个方法即可，而不需要改动原有已经声明好的各个手机类和具体工厂类。

下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![工厂方法模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_fmp_2.png)



## 优点

- 用户只需要关心其所需产品对应的具体工厂是哪一个即可，不需要关心产品的创建细节，也不需要知道具体产品类的类名。
- 当系统中加入新产品时，不需要修改抽象工厂和抽象产品提供的接口，也无须修改客户端和其他的具体工厂和具体产品，而只要添加一个具体工厂和与其对应的具体产品就可以了，符合了开闭原则（这一点与简单工厂模式不同）。



## 缺点

- 当系统中加入新产品时，除了需要提供新的产品类之外，还要提供与其对应的具体工厂类。因此系统中类的个数将成对增加，增加了系统的复杂度。



## iOS SDK 和 JDK中的应用

- 暂未发现iOS SDK中使用工厂方法的例子，有知道的小伙伴欢迎留言。
- 在JDK中，``Collection``接口声明了``iterator()``方法，该方法返回结果的抽象类是``Iterator``。``ArrayList``就实现了这个接口；，而ArrayList对应的具体产品是``Itr``。



# 三. 抽象工厂模式



## 定义

> 抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。



## 适用场景

有时候我们需要一个工厂可以提供多个产品对象，而不是单一的产品对象。比如系统中有多于一个的产品族，而每次只使用其中某一产品族，属于同一个产品族的产品将在一起使用。



在这里说一下产品族和产品等级结构的概念：



- 产品族：同一工厂生产的不同产品
- 产品等级结构：同一类型产品的不同实现



用一张图来帮助理解:

![](http://oih3a9o4n.bkt.clouddn.com/dp_afp_img1.png)

在上图中：



- 纵向的，不同形状，相同色系的图形属于同一产品组的产品，而同一产品族的产品对应的是同一个工厂；
- 横向的，同一形状，不同色系的图形属于统一产品等级结构的产品，而统一产品等级结构的产品对应的是同一个工厂方法。



下面再举一个例子帮助大家理解：



我们将小米，华为，苹果公司比作抽象工厂方法里的工厂：这三个工厂都有自己生产的手机，平板和电脑。
那么小米手机，小米平板，小米电脑就属于小米这个工厂的产品族；同样适用于华为工厂和苹果工厂。
而小米手机，华为手机，苹果手机则属于同一产品等级结构：手机的产品等级结构；平板和电脑也是如此。



结合这个例子对上面的图做一个修改可以更形象地理解抽象工厂方法的设计：



![](http://oih3a9o4n.bkt.clouddn.com/dp_afp_img2.png)



> 上面的关于产品族和产品等级结构的说法参考了[慕课网实战课程：java设计模式精讲 Debug 方式+内存分析](http://coding.imooc.com/class/270.html)的6-1节。







## 成员与类图



### 成员

抽象工厂模式的成员和工厂方法模式的成员是一样的，只不过抽象工厂方法里的工厂是面向产品族的。

1. 抽象工厂（Abstract Factory）：**抽象工厂**负责声明具体工厂的创建产品族内的所有产品的接口。
2. 具体工厂（Concrete Factory）：**具体工厂**负责创建产品族内的产品。
3. 抽象产品（Abstract Product）：**抽象产品**是工厂所创建的所有产品对象的父类，负责声明所有产品实例所共有的公共接口。
4. 具体产品（Concrete Product）：**具体产品**是工厂所创建的所有产品对象类，它以自己的方式来实现其共同父类声明的接口。

下面通过类图来看一下各个成员之间的关系：



### 模式类图

![抽象工厂模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_afp_1.png)





- 抽象工厂模式与工厂方法模式最大的区别在于，工厂方法模式针对的是一个产品等级结构，而抽象工厂模式则需要面对多个产品等级结构
- 增加新的具体工厂和产品族很方便，无须修改已有系统，符合“开闭原则”。



## 代码示例



### 场景概述

由于抽象工厂方法里的工厂是面向产品族的，所以为了贴合抽象工厂方法的特点，我们将上面的场景做一下调整：在上面两个例子中，商店只卖手机。在这个例子中我们让商店也卖电脑：分别是苹果电脑，小米电脑，华为电脑。



### 场景分析

如果我们还是套用上面介绍过的工厂方法模式来实现该场景的话，则需要创建三个电脑产品对应的工厂：苹果电脑工厂，小米电脑工厂，华为电脑工厂。这就导致类的个数直线上升，以后如果还增加其他的产品，还需要添加其对应的工厂类，这显然是不够优雅的。

仔细看一下这六个产品的特点，我们可以把这它们划分在三个产品族里面：

1. 苹果产品族：苹果手机，苹果电脑
2. 小米产品族：小米手机，小米电脑
3. 华为产品族：华为手机，华为电脑

而抽象方法恰恰是面向产品族设计的，因此该场景适合使用的是抽象工厂方法。下面结合代码来看一下该如何设计。



### 代码实现

首先引入电脑的基类和各个品牌的电脑类：

电脑基类：

```objc
//================== Computer.h ==================
@interface Computer : NSObject

//package to store
- (void)packaging;

@end



//================== Computer.m ==================
@implementation Computer

- (void)packaging{
    //implemented by subclass
}

@end
```

苹果电脑类  ``MacBookComputer``：

```objc
//================== MacBookComputer.h ==================
@interface MacBookComputer : Computer

@end



//================== MacBookComputer.m ==================
@implementation MacBookComputer

- (void)packaging{
     NSLog(@"MacBookComputer has been packaged");
}

@end
```

小米电脑类 ``MIComputer``：

```objc
//================== MIComputer.h ==================
@interface MIComputer : Computer

@end



//================== MIComputer.m ==================
@implementation MIComputer

- (void)packaging{
    NSLog(@"MIComputer has been packaged");
}

@end
```

华为电脑类 ``MateBookComputer``：

```objc
//================== MateBookComputer.h ==================
@interface MateBookComputer : Computer

@end



//================== MateBookComputer.m ==================
@implementation MateBookComputer

- (void)packaging{
    NSLog(@"MateBookComputer has been packaged");
}

@end
```

引入电脑相关产品类以后，我们需要重新设计工厂类。因为抽象工厂方法模式的工厂是面向产品族的，所以抽象工厂方法模式里的工厂所创建的是同一产品族的产品。下面我们看一下抽象工厂方法模式的工厂该如何设计：

首先创建所有工厂都需要集成的抽象工厂，它声明了生产同一产品族的所有产品的接口：

```objc
//================== Factory.h ==================
#import "Phone.h"
#import "Computer.h"

@interface Factory : NSObject

+ (Phone *)createPhone;

+ (Computer *)createComputer;

@end



//================== Factory.m ==================
@implementation Factory

+ (Phone *)createPhone{
    
    //implemented by subclass
    return nil;
}

+ (Computer *)createComputer{
    
    //implemented by subclass
    return nil;
}

@end
```

接着，根据不同的产品族，我们创建不同的具体工厂：

首先是苹果产品族工厂 ``AppleFactory``：

```objc
//================== AppleFactory.h ==================
@interface AppleFactory : Factory

@end



//================== AppleFactory.m ==================
#import "IPhone.h"
#import "MacBookComputer.h"

@implementation AppleFactory

+ (Phone *)createPhone{
    
    IPhone *iPhone = [[IPhone alloc] init];
    NSLog(@"iPhone has been created");
    return iPhone;
}

+ (Computer *)createComputer{
    
    MacBookComputer *macbook = [[MacBookComputer alloc] init];
    NSLog(@"Macbook has been created");
    return macbook;
}

@end
```

接着是小米产品族工厂 ``MIFactory``：

```objc
//================== MIFactory.h ==================
@interface MIFactory : Factory

@end



//================== MIFactory.m ==================
#import "MIPhone.h"
#import "MIComputer.h"

@implementation MIFactory

+ (Phone *)createPhone{
    
    MIPhone *miPhone = [[MIPhone alloc] init];
    NSLog(@"MIPhone has been created");
    return miPhone;
}

+ (Computer *)createComputer{
    
    MIComputer *miComputer = [[MIComputer alloc] init];
    NSLog(@"MIComputer has been created");
    return miComputer;
}

@end
```

最后是华为产品族工厂 ``HWFactory``：

```objc
//================== HWFactory.h ==================
@interface HWFactory : Factory

@end



//================== HWFactory.m ==================
#import "HWPhone.h"
#import "MateBookComputer.h"

@implementation HWFactory

+ (Phone *)createPhone{
    
    HWPhone *hwPhone = [[HWPhone alloc] init];
    NSLog(@"HWPhone has been created");
    return hwPhone;
}

+ (Computer *)createComputer{
    
    MateBookComputer *hwComputer = [[MateBookComputer alloc] init];
    NSLog(@"HWComputer has been created");
    return hwComputer;
}

@end
```

以上就是工厂类的设计。这样设计好之后，客户端如果需要哪一产品族的某个产品的话，只需要找到对应产品族工厂后，调用生产该产品的接口即可。假如需要苹果电脑，只需要委托苹果工厂来制造苹果电脑即可；如果需要小米手机，只需要委托小米工厂制造小米手机即可。



下面用代码来模拟一下这个场景：

```objc
//================== Using by client ==================


Store *store = [[Store alloc] init];
    
//Store wants to sell MacBook
Computer *macBook = [AppleFactory createComputer];
[macBook packaging];
    
[store sellComputer:macBook];
    
    
//Store wants to sell MIPhone
Phone *miPhone = [MIFactory createPhone];
[miPhone packaging];
    
[store sellPhone:miPhone];
    
    
//Store wants to sell MateBook
Computer *mateBook = [HWFactory createComputer];
[mateBook packaging];
    
[store sellComputer:mateBook];
```

上面的代码就是模拟了商店售卖苹果电脑，小米手机，华为电脑的场景。而今后如果该商店引入了新品牌的产品，比如联想手机，联想电脑，那么我们只需要新增联想手机类，联想电脑类，联想工厂类即可。



下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![抽象工厂模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_afp_2.png)

> 由于三个工厂的产品总数过多，因此在这里只体现了苹果工厂和小米工厂的产品。



## 优点

- 具体产品在应用层代码隔离，不需要关心产品细节。只需要知道自己需要的产品是属于哪个工厂的即可
  当一个产品族中的多个对象被设计成一起工作时，它能够保证客户端始终只使用同一个产品族中的对象。这对一些需要根据当前环境来决定其行为的软件系统来说，是一种非常实用的设计模式。



## 缺点

- 规定了所有可能被创建的产品集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口。
- 新增产品等级比较困难
- 产品等级固定，而产品族不固定，扩展性强的场景。





## iOS SDK 和 JDK中的应用

- 暂未发现iOS SDK中使用抽象工厂方法的例子，有知道的小伙伴欢迎留言。
- JDK中有一个数据库连接的接口``Connection``。在这个接口里面有``createStatement()``和``prepareStatement(String sql) ``。这两个接口都是获取的统一产品族的对象，比如MySql和PostgreSQL产品族，具体返回的是哪个产品族对象，取决于所连接的数据库类型。



OK，到现在三个工厂模式已经讲完了。在继续讲解下面三个设计模式之前，先简单回顾一下上面讲解的三个工厂模式：

大体上看，简单工厂模式，工厂方法模式和抽象工厂模式的复杂程度是逐渐升高的。

- 简单工厂模式使用不同的入参来让同一个工厂生产出不同的产品。
- 工厂方法模式和抽象工厂模式都需要有特定的工厂类来生产对应的产品；而工厂方法模式里的工厂是面向同一产品等级的产品；而抽象工厂方法模式里的工厂是面向同一产品族的产品的。

在实际开发过程中，我们需要根据业务场景的复杂程度的不同来采用最适合的工厂模式。



# 四. 单例模式



## 定义

> 单例模式(Singleton Pattern)：单例模式确保某一个类只有一个实例，并提供一个访问它的全剧访问点。



## 适用场景

系统只需要一个实例对象，客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。比较典型的例子是音乐播放器，日志系统类等等。



## 成员与类图



### 成员

单例模式只有一个成员，就是单例类。因为只有一个成员，所以该设计模式的类图比较简单：



### 模式类图

![单例模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_sp_1.png)

> 一般来说单例类会给外部提供一个获取单例对象的方法，内部会用静态对象的方式保存这个对象。



## 代码示例



### 场景概述

在这里我们创建一个简单的打印日至或上报日至的日至管理单例。



### 场景分析

在创建单例时，除了要保证提供唯一实例对象以外，还需注意多线程的问题。下面用代码来看一下。



### 代码实现

创建单例类 ``LogManager``

```objc
//================== LogManager.h ==================
@interface LogManager : NSObject

+(instancetype)sharedInstance;

- (void)printLog:(NSString *)logMessage;

- (void)uploadLog:(NSString *)logMessage;

@end



//================== LogManager.m ==================
@implementation LogManager

static LogManager* _sharedInstance = nil;

+(instancetype)sharedInstance
{
    static dispatch_once_t onceToken ;
    dispatch_once(&onceToken, ^{
        _sharedInstance = [[super allocWithZone:NULL] init] ;
    }) ;
    return _sharedInstance ;
}

+(id)allocWithZone:(struct _NSZone *)zone
{
    return [LogManager sharedInstance] ;
}

-(id)copyWithZone:(struct _NSZone *)zone
{
    return [LogManager sharedInstance];
}

-(id)mutableCopyWithZone:(NSZone *)zone
{
    return [LogManager sharedInstance];
}

- (void)printLog:(NSString *)logMessage{
    //print logMessage
}

- (void)uploadLog:(NSString *)logMessage{
    //upload logMessage
}

@end
```

从上面的代码中可以看到：

- ``sharedInstance``方法是向外部提供的获取唯一的实例对象的方法，也是该类中的其他可以创建对象的方法的都调用的方法。在这个方法内部使用了``dispatch_once``函数来避免多线程访问导致创建多个实例的情况。
- 为了在``alloc init``出初始化方法可以返回同一个实例对象，在``allocWithZone:``方法里面仍然调用了``sharedInstance``方法。
- 而且为了在``copy``和``mutableCopy``方法也可以返回同一个实例对象，在``copyWithZone:``与``mutableCopyWithZone``也是调用了``sharedInstance``方法。

下面分别用这些接口来验证一下实例的唯一性：

```objc
//================== Using by client ==================

//alloc&init
LogManager *manager0 = [[LogManager alloc] init];

//sharedInstance
LogManager *manager1 = [LogManager sharedInstance];

//copy
LogManager *manager2 = [manager0 copy];
    
//mutableCopy
LogManager *manager3 = [manager1 mutableCopy];
    
NSLog(@"\nalloc&init:     %p\nsharedInstance: %p\ncopy:           %p\nmutableCopy:    %p",manager0,manager1,manager2,manager3);
```

我们看一下打印出来的四个指针所指向对象的地址：

```
alloc&init:     0x60000000f7e0
sharedInstance: 0x60000000f7e0
copy:           0x60000000f7e0
mutableCopy:    0x60000000f7e0
```

可以看出打印出来的地址都相同，说明都是同一对象，证明了实现方法的正确性。

下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![单例模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_sp_2.png)

## 优点

- 提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。
- 因为该类在系统内存中只存在一个对象，所以可以节约系统资源。



## 缺点

- 由于单例模式中没有抽象层，因此单例类很难进行扩展。
- 对于有垃圾回收系统的语言（Java，C#）来说，如果对象长时间不被利用，则可能会被回收。那么如果这个单例持有一些数据的话，在回收后重新实例化时就不复存在了。



## iOS SDK 和 JDK中的应用

- 在Objective-C语言中使用单例模式的类有``NSUserDefaults``（key-value持久化）和``UIApplication``类（代表应用程序，可以处理一些点击事件等）。
- 在JDK中使用的单例模式的类有``Runtime``类（代表应用程序的运行环境，使应用程序能够与其运行的环境相连接）；``Desktop``类（允许 Java 应用程序启动已在本机桌面上注册的关联应用程序）



# 五. 生成器模式



## 定义



> 生成器模式(Builder Pattern)：也叫创建者模式，它将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。



具体点说就是：有些对象的创建流程是一样的，但是因为自身特性的不同，所以在创建他们的时候需要将创建过程和特性的定制分离开来。



下面我们看一下该设计模式的适用场景。



## 适用场景



当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时比较适合使用生成器模式。



一些复杂的对象，它们拥有多个组成部分（如汽车，它包括车轮、方向盘、发送机等各种部件）。而对于大多数用户而言，无须知道这些部件的装配细节，也几乎不会使用单独某个部件，而是使用一辆完整的汽车。而且这些部分的创建顺序是固定的，或者是需要指定的。

在这种情况下可以通过建造者模式对其进行设计与描述，生成器模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。



## 成员与类图

### 成员

建造者模式包含4个成员：

1. 抽象建造者（Builder）：定义构造产品的几个公共方法。
2. 具体建造者（ConcreteBuilder）：根据不同的需求来实现抽象建造者定义的公共方法；每一个具体建造者都包含一个产品对象作为它的成员变量。
3. 指挥者（Director）：根据传入的具体建造者来返回其所对应的产品对象。
4. 产品角色（Product）：创建的产品。

下面通过类图来看一下各个成员之间的关系：

### 模式类图

![生成器模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_bp_11.png)



> 需要注意的是：
>
> - Builder类中的product成员变量的关键字为``protected``，目的是为了仅让它和它的子类可以访问该成员变量。
> - Director类中的``constructProductWithBuilder(Builder builder)``方法是通过传入不同的builder来构造产品的。而且它的``getProduct()``方法同时也封装了``Concrete Builder``类的``getProduct()``方法，目的是为了让客户端直接从``Director``拿到对应的产品（有些资料里面的``Director``类没有封装``Concrete Builder``类的``getProduct()``方法）。



## 代码示例



### 场景概述

模拟一个制造手机的场景：手机的组装需要几个固定的零件：CPU，RAM，屏幕，摄像头，而且需要CPU -> RAM ->屏幕 -> 摄像头的顺序来制造。



### 场景分析



我们使用建造者设计模式来实现这个场景：首先不同的手机要匹配不同的builder；然后在``Director``类里面来定义制造顺序。





### 代码实现

首先我们定义手机这个类，它有几个属性：

```objc
//================== Phone.h ==================
@interface Phone : NSObject

@property (nonatomic, copy) NSString *cpu;
@property (nonatomic, copy) NSString *capacity;
@property (nonatomic, copy) NSString *display;
@property (nonatomic, copy) NSString *camera;

@end
```

然后我们创建抽象builder类：

```objc
//================== Builder.h ==================
#import "Phone.h"

@interface Builder : NSObject
{
    @protected Phone *_phone;
}

- (void)createPhone;

- (void)buildCPU;
- (void)buildCapacity;
- (void)buildDisplay;
- (void)buildCamera;


- (Phone *)obtainPhone;

@end
```

> 抽象builder类声明了创建手机各个组件的接口，也提供了返回手机实例的对象。

接下来我们创建对应不同手机的具体生成者类：

IPhoneXR手机的builder：``IPhoneXRBuilder``：

```objc
//================== IPhoneXRBuilder.h ==================
@interface IPhoneXRBuilder : Builder

@end



//================== IPhoneXRBuilder.m ==================
@implementation IPhoneXRBuilder


- (void)createPhone{
    
    _phone = [[Phone alloc] init];
}


- (void)buildCPU{
    
    [_phone setCpu:@"A12"];
}

- (void)buildCapacity{

    [_phone setCapacity:@"256"];
}


- (void)buildDisplay{
    
    [_phone setDisplay:@"6.1"];
}

- (void)buildCamera{
    
    [_phone setCamera:@"12MP"];
}

- (Phone *)obtainPhone{
    return _phone;
}

@end
```

小米8手机的builder：``MI8Builder``：

```objc
//================== MI8Builder.h ==================
@interface MI8Builder : Builder

@end



//================== MI8Builder.m ==================
@implementation MI8Builder

- (void)createPhone{
    
    _phone = [[Phone alloc] init];
}


- (void)buildCPU{
    
    [_phone setCpu:@"Snapdragon 845"];
}

- (void)buildCapacity{
    
    [_phone setCapacity:@"128"];
}


- (void)buildDisplay{
    
    [_phone setDisplay:@"6.21"];
}

- (void)buildCamera{
    
    [_phone setCamera:@"12MP"];
}

- (Phone *)obtainPhone{
    return _phone;
}

@end
```

> 从上面两个具体builder的代码可以看出，这两个builder都按照其对应的手机配置来创建其对应的手机。

下面来看一下Director的用法：

```objc
//================== Director.h ==================
#import "Builder.h"

@interface Director : NSObject

- (void)constructPhoneWithBuilder:(Builder *)builder;

- (Phone *)obtainPhone;

@end


//================== Director.m ==================
implementation Director
{
    Builder *_builder;
}


- (void)constructPhoneWithBuilder:(Builder *)builder{
    
    _builder = builder;
    
    [_builder buildCPU];
    [_builder buildCapacity];
    [_builder buildDisplay];
    [_builder buildCamera];
    
}


- (Phone *)obtainPhone{
    
    return [_builder obtainPhone];
}


@end

```

> Director类提供了``construct:``方法，需要传入builder的实例。该方法里面按照既定的顺序来创建手机。

最后我们看一下客户端是如何使用具体的Builder和Director实例的：



```objc
//================== Using by client ==================


//Get iPhoneXR
//1. A director instance
Director *director = [[Director alloc] init];
    
//2. A builder instance
IPhoneXRBuilder *iphoneXRBuilder = [[IPhoneXRBuilder alloc] init];
    
//3. Construct phone by director
[director construct:iphoneXRBuilder];
    
//4. Get phone by builder
Phone *iPhoneXR = [iphoneXRBuilder obtainPhone];
NSLog(@"Get new phone iPhoneXR of data: %@",iPhoneXR);
    
    
//Get MI8
MI8Builder *mi8Builder = [[MI8Builder alloc] init];
[director construct:mi8Builder];
Phone *mi8 = [mi8Builder obtainPhone];
NSLog(@"Get new phone MI8      of data: %@",mi8);
```

> 从上面可以看出客户端获取具体产品的过程：
>
> 1. 首先需要实例化一个Director的实例。
> 2. 然后根据所需要的产品找出其对应的builder。
> 3. 将builder传入director实例的``construct:``方法。
> 4. 从builder的``obtainPhone``获取手机实例。

下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![生成器模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_bp_2.png)

## 优点

- 客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。
- 每一个具体建造者都相对独立，而与其他的具体建造者无关，因此可以很方便地替换具体建造者或增加新的具体建造者， 用户使用不同的具体建造者即可得到不同的产品对象 。
- 增加新的具体建造者无须修改原有类库的代码，指挥者类针对抽象建造者类编程，系统扩展方便，符合“开闭原则”。
- 可以更加精细地控制产品的创建过程 。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。



## 缺点

- 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。

- 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大。


## iOS SDK 和 JDK中的应用

- 暂未发现iOS SDK中使用抽象工厂方法的例子，有知道的小伙伴欢迎留言。
- JDK中的``StringBuilder``属于builder，它向外部提供``append(String)``方法来拼接字符串（也可以传入int等其他类型）；而``toString()``方法来返回字符串。



# 六. 原型模式



## 定义

> 原型模式（Prototype Pattern）: 使用原型实例指定待创建对象的类型，并且通过复制这个原型来创建新的对象。



## 适用场景



- 对象层级嵌套比较多，从零到一创建对象的过程比较繁琐时，可以直接通过复制的方式创建新的对象

- 当一个类的实例只能有几个不同状态组合中的一种时，我们可以利用已有的对象进行复制来获得



## 成员与类图

       

### 成员

原型模式主要包含如下两个角色：

1. 抽象原型类(Prototype)：**抽象原型类**声明克隆自身的接口。 
2. 具体原型类（ConcretePrototype）：**具体原型类**实现克隆的具体操作（克隆数据，状态等）。 



下面通过类图来看一下各个成员之间的关系：



### 模式类图

![原型模式类图](http://oih3a9o4n.bkt.clouddn.com/dp_pp_1111.png)



需要注意的是，这里面的``clone()``方法返回的是被复制出来的实例对象。



## 代码示例



### 场景概述

模拟一份校招的简历，简历里面有人名，性别，年龄以及学历相关的信息。这里面学历相关的信息又包含学校名称，专业，开始和截止年限的信息。



### 场景分析

这里的学历相关信息可以使用单独一个对象来做，因此整体的简历对象的结构可以是：

简历对象：

- 人名
- 性别
- 年龄
- 学历对象
  - 学校名称
  - 专业
  - 开始年份
  - 结束年份

而且因为对于同一学校同一届的同一专业的毕业生来说，学历对象中的信息是相同的，这时候如果需要大量生成这些毕业生的简历的话比较适合使用原型模式。



### 代码实现

首先定义学历对象：

```objc
//================== UniversityInfo.h ==================
@interface UniversityInfo : NSObject<NSCopying>

@property (nonatomic, copy) NSString *universityName;
@property (nonatomic, copy) NSString *startYear;
@property (nonatomic, copy) NSString *endYear;
@property (nonatomic, copy) NSString *major;

- (id)copyWithZone:(NSZone *)zone;

@end



//================== UniversityInfo.m ==================
@implementation UniversityInfo

- (id)copyWithZone:(NSZone *)zone
{
    UniversityInfo *infoCopy = [[[self class] allocWithZone:zone] init];
    
    [infoCopy setUniversityName:[_universityName mutableCopy]];
    [infoCopy setStartYear:[_startYear mutableCopy]];
    [infoCopy setEndYear:[_endYear mutableCopy]];
    [infoCopy setMajor:[_major mutableCopy]];
   
    return infoCopy;
}

@end
```

> 因为学历对象是支持复制的，因此需要遵从``<NSCopying>``协议并实现``copyWithZone:``方法。而且支持的是深复制，所以在复制NSString的过程中需要使用``mutableCopy``来实现。



接着我们看一下简历对象:



```objc
//================== Resume.h ==================
#import "UniversityInfo.h"

@interface Resume : NSObject<NSCopying>

@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *gender;
@property (nonatomic, copy) NSString *age;

@property (nonatomic, strong) UniversityInfo *universityInfo;

@end



//================== Resume.m ==================
@implementation Resume

- (id)copyWithZone:(NSZone *)zone
{
    Resume *resumeCopy = [[[self class] allocWithZone:zone] init];
    
    [resumeCopy setName:[_name mutableCopy]];
    [resumeCopy setGender:[_gender mutableCopy]];
    [resumeCopy setAge:[_age mutableCopy]];
    [resumeCopy setUniversityInfo:[_universityInfo copy]];
    
    return resumeCopy;
}

@end
```

> 同样地，简历对象也需要遵从``<NSCopying>``协议并实现``copyWithZone:``方法。



最后我们看一下复制的效果有没有达到我们的预期（被复制对象和复制对象的地址和它们所有的属性对象的地址都不相同）

```objc
//================== Using by client ==================


//resume for LiLei
Resume *resume = [[Resume alloc] init];
resume.name = @"LiLei";
resume.gender = @"male";
resume.age = @"24";
    
UniversityInfo *info = [[UniversityInfo alloc] init];
info.universityName = @"X";
info.startYear = @"2014";
info.endYear = @"2018";
info.major = @"CS";
    
resume.universityInfo = info;
    
    
//resume_copy for HanMeiMei
Resume *resume_copy = [resume copy];
    
NSLog(@"\n\n\n======== original resume ======== %@\n\n\n======== copy resume ======== %@",resume,resume_copy);
    
resume_copy.name = @"HanMeiMei";
resume_copy.gender = @"female";
resume_copy.universityInfo.major = @"TeleCommunication";
    
NSLog(@"\n\n\n======== original resume ======== %@\n\n\n======== revised copy resume ======== %@",resume,resume_copy);
```

> 上面的代码模拟了这样一个场景：李雷同学写了一份自己的简历，然后韩梅梅复制了一份并修改了姓名，性别和专业这三个和李雷不同的信息。



这里我们重写了``Resume``的``description``方法来看一下所有属性的值及其内存地址。最后来看一下resume对象和resume_copy对象打印的结果：

```
//================== Output log ==================

======== original resume ======== 
resume object address:0x604000247d10
name:LiLei | 0x10bc0c0b0
gender:male | 0x10bc0c0d0
age:24 | 0x10bc0c0f0
university name:X| 0x10bc0c110
university start year:2014 | 0x10bc0c130
university end year:2018 | 0x10bc0c150
university major:CS | 0x10bc0c170


======== copy resume ======== 
resume object address:0x604000247da0
name:LiLei | 0xa000069654c694c5
gender:male | 0xa000000656c616d4
age:24 | 0xa000000000034322
university name:X| 0xa000000000000581
university start year:2014 | 0xa000000343130324
university end year:2018 | 0xa000000383130324
university major:CS | 0xa000000000053432





======== original resume ======== 
resume object address:0x604000247d10
name:LiLei | 0x10bc0c0b0
gender:male | 0x10bc0c0d0
age:24 | 0x10bc0c0f0
university name:X| 0x10bc0c110
university start year:2014 | 0x10bc0c130
university end year:2018 | 0x10bc0c150
university major:CS | 0x10bc0c170


======== revised copy resume ======== 
resume object address:0x604000247da0
name:HanMeiMei | 0x10bc0c1b0
gender:female | 0x10bc0c1d0
age:24 | 0xa000000000034322
university name:X| 0xa000000000000581
university start year:2014 | 0xa000000343130324
university end year:2018 | 0xa000000383130324
university major:TeleCommunication | 0x10bc0c1f0

```

> - 上面两个是原resume和刚被复制后的 copy resume的信息，可以看出来无论是这两个对象的地址还是它们的值对应的地址都是不同的，说明成功地实现了深复制。
> - 下面两个是原resume和被修改后的 copy_resume的信息，可以看出来新的copy_resume的值发生了变化，而且值所对应的地址还是和原resume的不同。

注：还可以用序列化和反序列化的办法来实现深复制，因为与代码设计上不是很复杂，很多语言直接提供了接口，故这里不做介绍。



下面我们看一下该例子对应的 UML类图，可以更直观地看一下各个成员之间的关系：



### 代码对应的类图

![原型模式代码示例类图](http://oih3a9o4n.bkt.clouddn.com/dp_pp_21.png) 



> 在这里需要注意的是：
>
> - ``copy``方法是``NSObject``类提供的复制本对象的接口。``NSObject``类似于Java中的``Object``类，在Objective-C中几乎所有的对象都继承与它。而且这个``copy``方法也类似于``Object``类的``clone()``方法。
> - ``copyWithZone(NSZone zone)``方法是接口``NSCopying``提供的接口。而因为这个接口存在于实现文件而不是头文件，所以它不是对外公开的；即是说外部无法直接调用``copyWithZone(NSZone zone)``方法。``copyWithZone(NSZone zone)``方法是在上面所说的``copy``方法调用后再调用的，作用是将对象的所有数据都进行复制。因此使用者需要在``copyWithZone(NSZone zone)``方法里做工作，而不是``copy``方法，这一点和Java的``clone``方法不同。





## 优点

- 可以利用原型模式简化对象的创建过程，尤其是对一些创建过程繁琐，包含对象层级比较多的对象来说，使用原型模式可以节约系统资源，提高对象生成的效率。
- 可以很方便得通过改变值来生成新的对象：有些对象之间的差别可能只在于某些值的不同；用原型模式可以快速复制出新的对象并手动修改值即可。



## 缺点

- 对象包含的所有对象都需要配备一个克隆的方法，这就使得在对象层级比较多的情况下，代码量会很大，也更加复杂。



## iOS SDK 和 JDK中的应用

- Objective-C中可以使用``<NSCopying>`` 协议，配合`` - (id)copyWithZone:(NSZone *)zone ``方法; 或者``<NSMutableCopying>``协议，配合 ``copyWithZone:/mutableCopyWithZone:``方法
- Java中可以让一个类实现``Cloneable``接口并实现``clone()``方法来复制该类的实例。





------



到这里设计模式中的创建型模式就介绍完了，读者可以结合UML类图和demo的代码来理解每个设计模式的特点和相互之间的区别，希望读者可以有所收获。

另外，本篇博客的代码和类图都保存在我的GitHub库中：[knightsj:object-oriented-design](https://github.com/knightsj/object-oriented-design)中的Chapter2。



下一篇是面向对象系列的第三篇，讲解的是面向对象设计模式中的结构型模式。
该系列的第一篇讲解的是设计原则，有兴趣的读者可以移步：[面向对象设计的六大设计原则（附 Demo 及 UML 类图）](https://juejin.im/post/5b9526c1e51d450e69731dc2)



# 参考书籍和教程

- [《设计模式 可复用面向对象软件的基础》](https://book.douban.com/subject/1052241/)
- [《Head First 设计模式》](https://book.douban.com/subject/2243615/)
- [《大话设计模式》](https://book.douban.com/subject/2334288/)
- [慕课网实战课程：java设计模式精讲 Debug 方式+内存分析](http://coding.imooc.com/class/270.html)

