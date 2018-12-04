
---
title: 面向对象设计的设计模式（二）：结构型模式（附 Demo 及 UML 类图）
tags: [iOS,Objectice-C,Object-Oriented,Design Pattern]
categories: Object-Oriented
---



![](http://jknight-blog.oss-cn-shanghai.aliyuncs.com/design-pattern-creation/odd_dp2_banner.png)

本篇是面向对象设计系列文章的第三篇，讲解的是设计模式中的结构型模式：



- 外观模式
- 适配器模式
- 桥接模式
- 代理模式
- 装饰者模式
- 享元模式

>该系列前面的两篇文章：
>- [面向对象设计的六大设计原则（附 Demo 及 UML 类图）](https://knightsj.github.io/2018/09/09/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E7%9A%84%E5%85%AD%E5%A4%A7%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99%EF%BC%88%E9%99%84%20Demo%20%E5%8F%8A%20UML%20%E7%B1%BB%E5%9B%BE%EF%BC%89/)
>- [面向对象设计的设计模式（一）：创建型设计模式（附 Demo 及 UML 类图）](https://knightsj.github.io/2018/10/21/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9A%E5%88%9B%E5%BB%BA%E5%9E%8B%E6%A8%A1%E5%BC%8F%EF%BC%88%E9%99%84%20Demo%20%E5%8F%8A%20UML%20%E7%B1%BB%E5%9B%BE%EF%BC%89/)


<!-- more -->


# 一. 外观模式



## 定义

> 外观模式(Facade Pattern)：外观模式定义了一个高层接口，为子系统中的一组接口提供一个统一的接口。外观模式又称为门面模式，它是一种结构型设计模式模式。



定义解读：通过这个高层接口，可以将客户端与子系统解耦：客户端可以不直接访问子系统，而是通过外观类间接地访问；同时也可以提高子系统的独立性和可移植性。



## 适用场景

- 子系统随着业务复杂度的提升而变得越来越复杂，客户端需要某些子系统共同协作来完成某个任务。
- 在多层结构的系统中，使用外观对象可以作为每层的入口来简化层间的调用。



## 成员与类图

### 成员

外观模式包括客户端共有三个成员：



- 客户端类（Client）:**客户端**是意图操作子系统的类，它与外观类直接接触；与外观类间接接触

- 外观类（Facade）：**外观类**知晓各个子系统的职责和接口，封装子系统的接口并提供给客户端
- 子系统类（SubSystem）：**子系统类**实现子系统的功能，对外观类一无所知



下面通过类图来看一下各个成员之间的关系：



### 模式类图

![外观模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755bffc790f116?w=1632&h=1032&f=png&s=25027)

> 上图中的``method1&2()``方法就是调用``SubSystem1``和``SubSystem2``的``method1()``和``method2()``方法。同样适用于``method2&3()``。



## 代码示例



### 场景概述



模拟一个智能家居系统。这个智能家居系统可以用一个中央遥控器操作其所接入的一些家具：台灯，音箱，空调等等。



在这里我们简单操纵几个设备：

- 空调
- CD Player
- DVD Player
- 音箱
- 投影仪





### 场景分析



有的时候，我们需要某个设备可以一次执行两个不同的操作；也可能会需要多个设备共同协作来执行一些任务。比如：



假设我们可以用遥控器直接开启热风，那么实际上就是两个步骤：

1. 开启空调
2. 空调切换为热风模式

我们把这两个步骤用一个操作包含起来，一步到位。像这样简化操作步骤的场景比较适合用外观模式。



同样的，我们想听歌的话，需要四个步骤：

1. 开启CD Player
2. 开启音箱
3. 连接CD Player和音箱
4. 播放CD Player

这些步骤我们也可以装在单独的一个接口里面。



类似的，如果我们想看DVD的话，步骤会更多，因为DVD需要同时输出声音和影像：

1. 开启DVD player
2. 开启音箱
3. 音响与DVD Player连接
4. 开启投影仪
5. 投影仪与DVD Player连接
6. 播放DVD Player

这些接口也可以装在一个单独的接口里。



最后，如果我们要出门，需要关掉所有家用电器，也不需要一个一个将他们关掉，也只需要一个关掉的总接口就好了，因为这个关掉的总接口里面可以包含所有家用电器的关闭接口。



因此，这些设备可以看做是该智能家居系统的子系统；而这个遥控器则扮演的是外观类的角色。



下面我们用代码来看一下如何实现这些设计。



### 代码实现

因为所有家用电器都有开启和关闭的操作，所以我们先创建一个家用电器的基类``HomeDevice``：

```objc
//================== HomeDevice.h ==================
//设备基类

@interface HomeDevice : NSObject

//连接电源
- (void)on;

//关闭电源
- (void)off;

@end
```

然后是继承它的所有家用电器类：

空调类``AirConditioner``:

```objc
//================== AirConditioner.h ==================

@interface AirConditioner : HomeDevice

//高温模式
- (void)startHighTemperatureMode;

//常温模式
- (void)startMiddleTemperatureMode;

//低温模式
- (void)startLowTemperatureMode;

@end
```

CD Player类：``CDPlayer``:

```objc
//================== CDPlayer.h ==================

@interface CDPlayer : HomeDevice

- (void)play;

@end
```

DVD Player类：``DVDPlayer``:

```objc
//================== DVDPlayer.h ==================

@interface DVDPlayer : HomeDevice

- (void)play;

@end
```

音箱类``VoiceBox``:

```objc
//================== VoiceBox.h ==================

@class CDPlayer;
@class DVDPlayer;

@interface VoiceBox : HomeDevice

//与CDPlayer连接
- (void)connetCDPlayer:(CDPlayer *)cdPlayer;

//与CDPlayer断开连接
- (void)disconnetCDPlayer:(CDPlayer *)cdPlayer;

//与DVD Player连接
- (void)connetDVDPlayer:(DVDPlayer *)dvdPlayer;

//与DVD Player断开连接
- (void)disconnetDVDPlayer:(DVDPlayer *)dvdPlayer;

@end
```

投影仪类``Projecter``：

```objc
//================== Projecter.h ==================

@interface Projecter : HomeDevice

//与DVD Player连接
- (void)connetDVDPlayer:(DVDPlayer *)dvdPlayer;

//与DVD Player断开连接
- (void)disconnetDVDPlayer:(DVDPlayer *)dvdPlayer;

@end
```

> 注意，音箱是可以连接CD Player和DVD Player的；而投影仪只能连接DVD Player

现在我们把所有的家用电器类和他们的接口都定义好了，下面我们看一下该实例的外观类``HomeDeviceManager``如何设计。

首先我们看一下客户端期望外观类实现的接口：

```objc
//================== HomeDeviceManager.h ==================

@interface HomeDeviceManager : NSObject

//===== 关于空调的接口 =====

//空调吹冷风
- (void)coolWind;

//空调吹热风
- (void)warmWind;


//===== 关于CD Player的接口 =====

//播放CD
- (void)playMusic;

//关掉音乐
- (void)offMusic;


//===== 关于DVD Player的接口 =====

//播放DVD
- (void)playMovie;

//关闭DVD
- (void)offMoive;


//===== 关于总开关的接口 =====

//打开全部家用电器
- (void)allDeviceOn;

//关闭所有家用电器
- (void)allDeviceOff;

@end
```

上面的接口分为了四大类，分别是：

- 关于空调的接口
- 关于CD Player的接口
- 关于DVD Player的接口
- 关于总开关的接口

> 为了便于读者理解，这四类的接口所封装的子系统接口的数量是逐渐增多的。

在看这些接口时如何实现的之前，我们先看一下外观类是如何保留这些子系统类的实例的。在该代码示例中，这些子系统类的实例在外观类的构造方法里被创建，而且作为外观类的成员变量被保存了下来。

```objc
//================== HomeDeviceManager.m ==================

@implementation HomeDeviceManager
{
    NSMutableArray *_registeredDevices;//所有注册(被管理的)的家用电器
    AirConditioner *_airconditioner;
    CDPlayer *_cdPlayer;
    DVDPlayer *_dvdPlayer;
    VoiceBox *_voiceBox;
    Projecter *_projecter;
    
}

- (instancetype)init{
    
    self = [super init];
    
    if (self) {
        
        _airconditioner = [[AirConditioner alloc] init];
        _cdPlayer = [[CDPlayer alloc] init];
        _dvdPlayer = [[DVDPlayer alloc] init];
        _voiceBox = [[VoiceBox alloc] init];
        _projecter = [[Projecter alloc] init];
        
        _registeredDevices = [NSMutableArray arrayWithArray:@[_airconditioner,
                                                              _cdPlayer,
                                                              _dvdPlayer,
                                                              _voiceBox,
                                                              _projecter]];
    }
    return self;
}
```

其中 ``_registeredDevices``这个成员变量是一个数组，它包含了所有和这个外观类实例关联的子系统实例。

子系统与外观类的关联实现方式不止一种，不作为本文研究重点，现在只需知道外观类保留了这些子系统的实例即可。按照顺序，我们首先看一下关于空调的接口的实现：

```objc
//================== HomeDeviceManager.m ==================

//空调吹冷风
- (void)coolWind{
    
    [_airconditioner on];
    [_airconditioner startLowTemperatureMode];
    
}

//空调吹热风
- (void)warmWind{
    
    [_airconditioner on];
    [_airconditioner startHighTemperatureMode];
}
```

吹冷风和吹热风的接口都包含了空调实例的两个接口，第一个都是开启空调，第二个则是对应的冷风和热风的接口。

我们接着看关于CD Player的接口的实现：

```objc
//================== HomeDeviceManager.m ==================

- (void)playMusic{
    
    //1. 开启CDPlayer开关
    [_cdPlayer on];
    
    //2. 开启音箱
    [_voiceBox on];
    
    //3. 音响与CDPlayer连接
    [_voiceBox connetCDPlayer:_cdPlayer];
    
    //4. 播放CDPlayer
    [_cdPlayer play];
}

//关掉音乐
- (void)offMusic{
    
   //1. 切掉与音箱的连接
    [_voiceBox disconnetCDPlayer:_cdPlayer];
    
    //2. 关掉音箱
    [_voiceBox off];
    
    //3. 关掉CDPlayer
    [_cdPlayer off];
}
```

在上面的场景分析中提到过，听音乐这个指令要分四个步骤：CD Player和音箱的开启，二者的连接，以及播放CD Player，这也比较符合实际生活中的场景。关掉音乐也是先断开连接再切断电源（虽然直接切断电源也可以）。

接下来我们看一下关于DVD Player的接口的实现：

```objc
//================== HomeDeviceManager.m ==================

- (void)playMovie{
    
    //1. 开启DVD player
    [_dvdPlayer on];
    
    //2. 开启音箱
    [_voiceBox on];
    
    //3. 音响与DVDPlayer连接
    [_voiceBox connetDVDPlayer:_dvdPlayer];
    
    //4. 开启投影仪
    [_projecter on];
    
    //5.投影仪与DVDPlayer连接
    [_projecter connetDVDPlayer:_dvdPlayer];
    
    //6. 播放DVDPlayer
    [_dvdPlayer play];
}


- (void)offMoive{

    //1. 切掉音箱与DVDPlayer连接
    [_voiceBox disconnetDVDPlayer:_dvdPlayer];
    
    //2. 关掉音箱
    [_voiceBox off];
    
    //3. 切掉投影仪与DVDPlayer连接
    [_projecter disconnetDVDPlayer:_dvdPlayer];
    
    //4. 关掉投影仪
    [_projecter off];
    
    //5. 关掉DVDPlayer
    [_dvdPlayer off];
}
```

因为DVD Player要同时连接音箱和投影仪，所以这两个接口封装的子系统接口相对于CD Player的更多一些。

最后我们看一下关于总开关的接口的实现：

```objc
//================== HomeDeviceManager.m ==================

//打开全部家用电器
- (void)allDeviceOn{
    
    [_registeredDevices enumerateObjectsUsingBlock:^(HomeDevice *device, NSUInteger idx, BOOL * _Nonnull stop) {
        [device on];
    }];
}


//关闭所有家用电器
- (void)allDeviceOff{
    
    [_registeredDevices enumerateObjectsUsingBlock:^(HomeDevice *device, NSUInteger idx, BOOL * _Nonnull stop) {
        [device off];
    }];
}
```

这两个接口是为了方便客户端开启和关闭所有设备的，有这两个接口的话，用户就不用一一开启或关闭多个设备了。

关于这两个接口的实现：

上文说过，该外观类通过一个数组成员变量``_registeredDevices``来保存所有可操作的设备。所以如果我们需要开启或关闭所有的设备就可以遍历这个数组并向每个元素调用``on``或``off``方法。因为这些元素都继承于``HomeDevice``，也就是都有``on``或``off``方法。

这样做的好处是，我们不需要单独列出所有设备来分别调用它们的接口；而且后面如果添加或者删除某些设备的话也不需要修改这两个接口的实现了。

下面我们看一下该demo多对应的类图。



### 代码对应的类图

![外观模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755bffc0b8f2cc?w=2232&h=1418&f=png&s=69709)

> 从上面的UML类图中可以看出，该示例的子系统之间的耦合还是比较多的；而外观类``HomeDeviceManager``的接口大大简化了``User``对这些子系统的使用成本。



## 优点

- 实现了客户端与子系统间的解耦：客户端无需知道子系统的接口，简化了客户端调用子系统的调用过程，使得子系统使用起来更加容易。同时便于子系统的扩展和维护。
- 符合迪米特法则（最少知道原则）：子系统只需要将需要外部调用的接口暴露给外观类即可，而且他的接口则可以隐藏起来。



## 缺点

- 违背了开闭原则：在不引入抽象外观类的情况下，增加新的子系统可能需要修改外观类或客户端的代码。





## Objective-C & Java的实践

- Objective-C:``SDWebImage``封装了负责图片下载的类和负责图片缓存的类，而外部仅向客户端暴露了简约的下载图片的接口。
- Java:``Spring-JDBC``中的``JdbcUtils``封装了``Connection``，``ResultSet``，``Statement``的方法提供给客户端



# 二. 适配器模式



## 定义

> 适配器模式(Adapter Pattern) ：将一个接口转换成客户希望的另一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。适配器模式的别名是包装器模式（Wrapper），是一种结构型设计模式。



定义解读：适配器模式又分为对象适配器和类适配器两种。

- 对象适配器：利用组合的方式将请求转发给被适配者。
- 类适配器：通过适配器类多重继承目标接口和被适配者，将目标方法的调用转接到调用被适配者的方法。



## 适用场景



- 想使用一个已经存在的类，但是这个类的接口不符合我们的要求，原因可能是和系统内的其他需要合作的类不兼容。
- 想创建一个功能上可以复用的类，这个类可能需要和未来某些未知接口的类一起工作。







## 成员与类图



### 成员

适配器模式有三个成员：

- 目标（Target）：**客户端**希望直接接触的类，给客户端提供了调用的接口
- 被适配者（Adaptee）：**被适配者**是已经存在的类，即需要被适配的类
- 适配器（Adapter）：**适配器**对Adaptee的接口和Target的接口进行适配



### 模式类图



如上文所说，适配器模式分为类适配器模式和对象适配器模式，因此这里同时提供这两种细分模式的 UML类图。





对象适配器模式：

![适配器模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c0001cc8c4e?w=1634&h=900&f=png&s=20894)



> 对象适配器中，被适配者的对象被适配器所持有。当适配器的``request``方法被调用时，在这个方法内部再调用被适配者对应的方法。



类适配器模式：
![类适配器模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c0149dc33ee?w=1652&h=884&f=png&s=17827)

> 类适配器中采用了多继承的方式：适配器同时继承了目标类和被适配者类，也就都持有了者二者的方法。

多继承在Objective-C中可以通过遵循多个协议来实现，在本模式的代码示例中只使用对象适配器来实现。



## 代码示例



### 场景概述

模拟一个替换缓存组件的场景：目前客户端已经依赖于旧的缓存组件的接口，而后来发现有一个新的缓组件的性能更好一些，需要将旧的缓存组件替换成新的缓存组件，但是新的缓存组件的接口与旧的缓存接口不一致，所以目前来看客户端是无法直接与新缓存组件一起工作的。



### 场景分析

由于客户端在很多地方依赖了旧缓存组件的接口，将这些地方的接口都换成新缓存组件的接口会比较麻烦，而且万一后面还要换回旧缓存组件或者再换成另外一个新的缓存组件的话就还要做重复的事情，这显然是不够优雅的。

因此该场景比较适合使用适配器模式：创建一个适配器，让原本与旧缓存接口的客户端可以与新缓存组件一起工作。



在这里，新的缓存组件就是``Adaptee``，旧的缓存组件（接口）就是``Target``，因为它是直接和客户端接触的。而我们需要创建一个适配器类``Adaptor``来让客户端与新缓存组件一起工作。下面用代码看一下上面的问题如何解决：



### 代码实现

首先我们创建旧缓存组件，并让客户端正常使用它。
先创建旧缓存组件的接口``OldCacheProtocol``：

> 对应Java的接口，Objective-C中叫做协议，也就是protocol。

```objc
//================== OldCacheProtocol.h ==================

@protocol OldCacheProtocol <NSObject>

- (void)old_saveCacheObject:(id)obj forKey:(NSString *)key;

- (id)old_getCacheObjectForKey:(NSString *)key;

@end
```

> 可以看到该接口包含了两个操作缓存的方法，方法前缀为``old``。

再简单创建一个缓存组件类``OldCache``，它实现了``OldCacheProtocol``接口：

```objc
//================== OldCache.h ==================

@interface OldCache : NSObject <OldCacheProtocol>

@end


    
//================== OldCache.m ==================
    
@implementation OldCache

- (void)old_saveCacheObject:(id)obj forKey:(NSString *)key{
    
    NSLog(@"saved cache by old cache object");
    
}

- (id)old_getCacheObjectForKey:(NSString *)key{
    
    NSString *obj = @"get cache by old cache object";
    NSLog(@"%@",obj);
    return obj;
}

@end
```

> 为了读者区分方便，将新旧两个缓存组件取名为``NewCache``和``OldCache``。实现代码也比较简单，因为不是本文介绍的重点，只需区分接口名称即可。

现在我们让客户端来使用这个旧缓存组件：

```objc
//================== client.m ==================

@interface ViewController ()

@property (nonatomic, strong) id<OldCacheProtocol>cache;

@end

@implementation ViewController


- (void)viewDidLoad {
    
    [super viewDidLoad];
 
    //使用旧缓存
    [self useOldCache];

    //使用缓存组件操作
    [self saveObject:@"cache" forKey:@"key"];
    
}

//实例化旧缓存并保存在``cache``属性里
- (void)useOldCache{

    self.cache = [[OldCache alloc] init];
}

//使用cache对象
- (void)saveObject:(id)object forKey:(NSString *)key{

    [self.cache old_saveCacheObject:object forKey:key];
}
```

> - 在这里的客户端就是``ViewController``，它持有一个遵从``OldCacheProtocol``协议的实例，也就是说它目前依赖于``OldCacheProtocol``的接口。
> - ``useOldCache``方法用来实例化旧缓存并保存在``cache``属性里。
> - ``saveObject:forKey:``方法是真正使用cache对象来保存缓存。

运行并打印一下结果输出是：``saved cache by old cache object``。现在看来客户端使用旧缓存是没有问题的。

而现在我们要加入新的缓存组件了：
首先定义新缓存组件的接口``NewCacheProtocol``：

```objc
//================== NewCacheProtocol.h ==================

@protocol NewCacheProtocol <NSObject>

- (void)new_saveCacheObject:(id)obj forKey:(NSString *)key;

- (id)new_getCacheObjectForKey:(NSString *)key;

@end
```

可以看到，``NewCacheProtocol``与``OldCacheProtocol``接口大致是相似的，但是名称还是不同，这里使用了不同的方法前缀做了区分。

接着看一下新缓存组件是如何实现这个接口的：

```objc
//================== NewCache.h ==================

@interface NewCache : NSObject <NewCacheProtocol>

@end


    
//================== NewCache.m ==================
@implementation NewCache

- (void)new_saveCacheObject:(id)obj forKey:(NSString *)key{
    
    NSLog(@"saved cache by new cache object");
}

- (id)new_getCacheObjectForKey:(NSString *)key{
    
    NSString *obj = @"saved cache by new cache object";
    NSLog(@"%@",obj);
    return obj;
}
@end
```

现在我们拿到了新的缓存组件，但是客户端类目前依赖的是旧的接口，因此适配器类应该上场了：

```objc
//================== Adaptor.h ==================

@interface Adaptor : NSObject <OldCacheProtocol>

- (instancetype)initWithNewCache:(NewCache *)newCache;

@end


    
//================== Adaptor.m ==================
    
@implementation Adaptor
{
    NewCache *_newCache;
}

- (instancetype)initWithNewCache:(NewCache *)newCache{
    
    self = [super init];
    if (self) {
        _newCache = newCache;
    }
    return self;
}

- (void)old_saveCacheObject:(id)obj forKey:(NSString *)key{
    
    //transfer responsibility to new cache object
    [_newCache new_saveCacheObject:obj forKey:key];
}

- (id)old_getCacheObjectForKey:(NSString *)key{
    
    //transfer responsibility to new cache object
    return [_newCache new_getCacheObjectForKey:key];
    
}
@end
```

> - 首先，适配器类也实现了旧缓存组件的接口；目的是让它也可以接收到客户端操作旧缓存组件的方法。
> - 然后，适配器的构造方法里面需要传入新组件类的实例；目的是在收到客户端操作旧缓存组件的命令后，将该命令转发给新缓存组件类，并调用其对应的方法。
> - 最后我们看一下适配器类是如何实现两个旧缓存组件的接口的：在``old_saveCacheObject:forKey:``方法中，让新缓存组件对象调用对应的``new_saveCacheObject:forKey:``方法；同样地，在``old_getCacheObjectForKey``方法中，让新缓存组件对象调用对应的``new_getCacheObjectForKey:``方法。

这样一来，适配器类就定义好了。
那么最后我们看一下在客户端里面是如何使用适配器的：

```objc
//================== client ==================

- (void)viewDidLoad {

    [super viewDidLoad];
 
    //使用新缓存组件
    [self useNewCache];
    
    [self saveObject:@"cache" forKey:@"key"];
}

- (void)useOldCache{
    
    self.cache = [[OldCache alloc] init];
}

//使用新缓存组件
- (void)useNewCache{
    
    self.cache = [[Adaptor alloc] initWithNewCache:[[NewCache alloc] init]];
}

//使用cache对象
- (void)saveObject:(id)object forKey:(NSString *)key{
    
    [self.cache old_saveCacheObject:object forKey:key];
}
```

> 我们可以看到，在客户端里面，只需要改一处就可以了：将我们定义好的适配器类保存在原来的``cache``属性中就可以了（``useNewCache``方法的实现）。而真正操作缓存的方法``saveObject:forKey``不需要有任何改动。

我们可以看到，使用适配器模式，客户端调用旧缓存组件接口的方法都不需要改变；只需稍作处理，就可以在新旧缓存组件中来回切换，也不需要原来客户端对缓存的操作。



而之所以可以做到这么灵活，其实也是因为在一开始客户端只是依赖了旧缓存组件类所实现的接口，而不是旧缓存组件类的类型。有心的读者可能注意到了，上面``viewController``的属性是``@property (nonatomic, strong) id<OldCacheProtocol>cache;``。正因为如此，我们新建的适配器实例才能直接用在这里，因为适配器类也是实现了``<OldCacheProtocol>``接口。相反，如果我们的``cache``属性是这么写的：``@property (nonatomic, strong) OldCache *cache;``，即客户端依赖了旧缓存组件的类型，那么我们的适配器类就无法这么容易地放在这里了。因此为了我们的程序在将来可以更好地修改和扩展，依赖接口是一个前提。

下面我们看一下该代码示例对应的类图：

### 代码对应的类图

![适配器模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c0149c74bc7?w=2358&h=1090&f=png&s=61305)



## 优点

- 符合开闭原则：使用适配器而不需要改变现有类，提高类的复用性。
- 目标类和适配器类解耦，提高程序扩展性。



## 缺点

- 增加了系统的复杂性



## Objective-C & Java的实践

- Objective-C：暂时未发现适配器模式的实践，有知道的同学可以留言
- Java：JDK中的``XMLAdapter``使用了适配器模式。



# 三. 桥接模式



## 定义

> 桥接模式(Simple Factory Pattern)：将抽象部分与它的实现部分分离,使它们都可以独立地变化。



定义解读：桥接模式的核心是两个抽象以组合的形式关联到一起，从而他们的实现就互不依赖了。



## 适用场景



如果一个系统存在两个独立变化的维度，而且这两个维度都需要进行扩展的时候比较适合使用桥接模式。



下面来看一下简单工厂模式的成员和类图。



## 成员与类图

### 成员

桥接模式一共只有三个成员：

- 抽象类（Abstraction）：**抽象类**维护一个实现部分的对象的引用，并声明调用实现部分的对象的接口。
- 扩展抽象类（RefinedAbstraction）：**扩展抽象类**定义跟实际业务相关的方法。
- 实现类接口（Implementor）：**实现类接口**定义实现部分的接口。
- 具体实现类（ConcreteImplementor）：**具体实现类**具体实现类是实现实现类接口的对象。



下面通过类图来看一下各个成员之间的关系：



### 模式类图

![桥接模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755bffef78877f?w=1361&h=529&f=png&s=18165)

> 从类图中可以看出``Abstraction``持有``Implementor``，但是二者的实现类互不依赖。这就是桥接模式的核心。



## 代码示例



### 场景概述



创建一些不同的形状，这些形状带有不同的颜色。



三种形状：

- 正方形
- 长方形
- 原型

三种颜色：

- 红色
- 绿色
- 蓝色



### 场景分析

根据上述需求，可能有的朋友会这么设计：

- 正方形
  - 红色正方形
  - 绿色正方形
  - 蓝色正方形
- 长方形
  - 红色长方形
  - 绿色长方形
  - 蓝色长方形
- 圆形
  - 红色圆形
  - 绿色圆形
  - 蓝色圆形

这样的设计确实可以实现上面的需求。但是设想一下，如果后来增加了一种颜色或者形状的话，是不是要多出来很多类？如果形状的种类数是``m``，颜色的种类数是``n``，以这种方式创建的总类数就是 ``m*n``，当m或n非常大的时候，它们相乘的结果就会变得很大。



我们观察一下这个场景：形状和颜色这二者的是没有关联性的，二者可以独立扩展和变化，这样的组合比较适合用桥接模式来做。



根据上面提到的桥接模式的成员：

- 抽象类就是图形的抽象类
- 扩展抽象类就是继承图形抽象类的子类：各种形状
- 实现类接口就是颜色接口
- 具体实现类就是继承颜色接口的类：各种颜色



下面我们用代码看一下该如何设计。



### 代码实现

首先我们创建形状的基类``Shape``：

```objc
//================== Shape.h ==================

@interface Shape : NSObject
{
    @protected Color *_color;
}

- (void)renderColor:(Color *)color;

- (void)show;

@end


    

//================== Shape.m ==================
    
@implementation Shape

- (void)renderColor:(Color *)color{
    
    _color = color;
}

- (void)show{
    NSLog(@"Show %@ with %@",[self class],[_color class]);
}

@end
```

> 由上面的代码可以看出：
>
> - 形状类``Shape``持有``Color``类的实例，二者是以组合的形式结合到一起的。而且``Shape``类定义了供外部传入``Color``实例的方法``renderColor:``：在这个方法里面接收从外部传入的``Color``实例并保存起来。
> - 另外一个公共接口``show``实际上就是打印这个图形的名称及其所搭配的颜色，便于我们后续验证。

接着我们创建三种不同的图形类，它们都继承于``Shape``类：

正方形类：

```objc
//================== Square.h ==================

@interface Square : Shape

@end


    
    
//================== Square.m ==================
    
@implementation Square

- (void)show{
    
    [super show];
}

@end
```

长方形类：

```objc
//================== Rectangle.h ==================

@interface Rectangle : Shape

@end

    
    
    
//================== Rectangle.m ==================
    
@implementation Rectangle

- (void)show{
    
    [super show];
}

@end
```

圆形类：

```objc
//================== Circle.h ==================

@interface Circle : Shape

@end
    

    
    
//================== Circle.m ==================  
    
@implementation Circle

- (void)show{
    
    [super show];
}

@end
```

还记得上面的``Shape``类持有的``Color``类么？它就是所有颜色类的父类：

```objc
//================== Color.h ==================   

@interface Color : NSObject

@end
    
    


//================== Color.m ================== 
    
@implementation Color

@end
```

接着我们创建继承这个``Color``类的三个颜色类：

红色类：

```objc
//================== RedColor.h ==================

@interface RedColor : Color

@end


    
    
//================== RedColor.m ==================  
    
@implementation RedColor

@end
```

绿色类：

```objc
//================== GreenColor.h ==================

@interface GreenColor : Color

@end


    
    
//================== GreenColor.m ==================
@implementation GreenColor

@end
```

蓝色类：

```objc
//================== BlueColor.h ==================

@interface BlueColor : Color

@end


    
 
//================== BlueColor.m ==================
    
@implementation BlueColor

@end
```

OK，到现在所有的形状类和颜色类的相关类已经创建好了，我们看一下客户端是如何使用它们来组合成不同的带有颜色的形状的:

```objc
//================== client ==================


//create 3 shape instances
Rectangle *rect = [[Rectangle alloc] init];
Circle *circle = [[Circle alloc] init];
Square *square = [[Square alloc] init];
    
//create 3 color instances
RedColor *red = [[RedColor alloc] init];
GreenColor *green = [[GreenColor alloc] init];
BlueColor *blue = [[BlueColor alloc] init];
    
//rect & red color
[rect renderColor:red];
[rect show];
    
//rect & green color
[rect renderColor:green];
[rect show];
    
    
//circle & blue color
[circle renderColor:blue];
[circle show];
    
//circle & green color
[circle renderColor:green];
[circle show];
    
    
    
//square & blue color
[square renderColor:blue];
[square show];
    
//square & red color
[square renderColor:red];
[square show];
```

> 上面的代码里，我们先声明了所有的形状类和颜色类的实例，然后自由搭配，形成不同的形状+颜色的组合。



下面我们通过打印的结果来看一下组合的效果：

```
Show Rectangle with RedColor
Show Rectangle with GreenColor
Show Circle with BlueColor
Show Circle with GreenColor
Show Square with BlueColor
Show Square with RedColor
```

从打印的接口可以看出组合的结果是没问题的。



跟上面没有使用桥接模式的设计相比，使用桥接模式需要的类的总和是  ``m + n``：当m或n的值很大的时候是远小于 ``m * n``（没有使用桥接，而是使用继承的方式）的。

而且如果后面还要增加形状和颜色的话，使用桥接模式就可以很方便地将原有的形状和颜色和新的形状和颜色进行搭配了，新的类和旧的类互不干扰。

下面我们看一下上面代码所对应的类图：

### 代码对应的类图

![桥接模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c00af7238c3?w=2110&h=906&f=png&s=30452)

> 从UML类图可以看出，该设计是由两个抽象层的类``Shape``和``Color``构建的，正因为依赖的双方都是抽象类（而不是具体的实现），而且二者是以组合的方式联系到一起的，所以扩展起来非常方便，互不干扰。这对于今后我们对代码的设计有比较好的借鉴意义。



## 优点

- 扩展性好，符合开闭原则：将抽象与实现分离，让二者可以独立变化



## 缺点

- 在设计之前，需要识别出两个独立变化的维度。



## Objective-C & Java的实践

- Objective-C：暂时未发现桥接模式的实践，有知道的同学可以留言
- Java：``Spring-JDBC``中的``DriveManager``通过``registerDriver``方法注册不同类型的驱动



# 四. 代理模式



## 定义

> 代理模式(Proxy Pattern) ：为某个对象提供一个代理，并由这个代理对象控制对原对象的访问。



定义解读：使用代理模式以后，客户端直接访问代理，代理在客户端和目标对象之间起到中介的作用。



## 适用场景

在某些情况下，一个客户不想或者不能直接引用一个对象，此时可以通过一个称之为“代理”的第三者来实现间接引用。



因为代理对象可以在客户端和目标对象之间起到中介的作用，因此可以通过代理对象去掉客户不能看到 的内容和服务或者添加客户需要的额外服务。



根据业务的不同，代理也可以有不同的类型：

- 远程代理：为位于不同地址或网络化中的对象提供本地代表。
- 虚拟代理：根据要求创建重型的对象。
- 保护代理：根据不同访问权限控制对原对象的访问。



下面来看一下代理模式的成员和类图。



## 成员与类图



### 成员

代理模式算上客户端一共有四个成员：

- 客户端（Client）：客户端意图访问真是主体接口
- 抽象主题（Subejct）：**抽象主题**定义客户端需要访问的接口
- 代理（Proxy）：**代理**继承于抽象主题，目的是为了它持有真实目标的实例的引用，客户端直接访问代理
- 真实主题（RealSubject）：**真实主题**即是被代理的对象，它也继承于抽象主题，它的实例被代理所持有，它的接口被包装在了代理的接口中，而且客户端无法直接访问真实主题对象。



> 其实我也不太清楚代理模式里面为什么会是Subject和RealSubject这个叫法。



下面通过类图来看一下各个成员之间的关系：



### 模式类图

![代理模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c00b1cb1b6f?w=1732&h=1054&f=png&s=19403)

> 从类图中可以看出，工厂类提供一个静态方法：通过传入的字符串来制造其所对应的产品。



## 代码示例



### 场景概述

在这里举一个买房者通过买房中介买房的例子。

现在一般我们买房子不直接接触房东，而是先接触中介，买房的相关合同和一些事宜可以先和中介进行沟通。



在本例中，我们在这里让买房者直接支付费用给中介，然后中介收取一部分的中介费， 再将剩余的钱交给房东。



### 场景分析



中介作为房东的代理，与买房者直接接触。而且中介还需要在真正交易前做其他的事情（收取中介费，帮买房者check房源的真实性等等），因此该场景比较适合使用代理模式。



根据上面的代理模式的成员：



- 客户端就是买房者

- 代理就是中介
- 真实主题就是房东
- 中介和房东都会实现收钱的方法，我们可以定义一个抽象主题类，它有一个公共接口是收钱的方法。



### 代码实现

首先我们定义一下房东和代理需要实现的接口``PaymentInterface``（在类图里面是继承某个共同对象，我个人比较习惯用接口来做）。

```objc
//================== PaymentInterface.h ==================

@protocol PaymentInterface <NSObject>

- (void)getPayment:(double)money;

@end
```

这个接口声明了中介和房东都需要实现的方法``getPayment:``

接着我们声明代理类``HouseProxy``:

```objc
//================== HouseProxy.h ==================

@interface HouseProxy : NSObject<PaymentInterface>

@end

    


//================== HouseProxy.m ==================
const float agentFeeRatio = 0.35;

@interface HouseProxy()

@property (nonatomic, copy) HouseOwner *houseOwner;

@end

@implementation HouseProxy

- (void)getPayment:(double)money{
    
    double agentFee = agentFeeRatio * money;
    NSLog(@"Proxy get payment : %.2lf",agentFee);
    
    [self.houseOwner getPayment:(money - agentFee)];
}

- (HouseOwner *)houseOwner{
    
    if (!_houseOwner) {
         _houseOwner = [[HouseOwner alloc] init];
    }
    return _houseOwner;
}

@end
```

在``HouseProxy``里面，持有了房东，也就是被代理者的实例。然后在的``getPayment:``方法里，调用了房东实例的``getPayment:``方法。而且我们可以看到，在调用房东实例的``getPayment:``方法，代理先拿到了中介费（中介费比率``agentFeeRatio``定义为0.35，即中介费的比例占35%）。

这里面除了房东实例的``getPayment:``方法之外的一些操作就是代理存在的意义：它可以在真正被代理对象做事情之前，之后做一些其他额外的事情。比如类似AOP编程一样，定义类似的``before***Method``或是``after**Method``方法等等。

最后我们看一下房东是如何实现``getPayment:``方法的：

```objc
//================== HouseOwner.h ==================

@interface HouseOwner : NSObject<PaymentInterface>

@end

    

    
//================== HouseOwner.m ==================
    
@implementation HouseOwner

- (void)getPayment:(double)money{
    
    NSLog(@"House owner get payment : %.2lf",money);
}

@end
```

房东类``HouseOwner``按照自己的方式实现了``getPayment:``方法。

很多时候被代理者（委托者）可以完全按照自己的方式去做事情，而把一些额外的事情交给代理来做，这样可以保持原有类的功能的纯粹性，符合开闭原则。

下面我们看一下客户端的使用以及打印出来的结果：

客户端代码：

```objc
//================== client.m ==================

HouseProxy *proxy = [[HouseProxy alloc] init];
[proxy getPayment:100];
```

> 上面的客户端支付给了中介100元。

下面我们看一下打印结果：

```
Proxy get payment : 35.00
House owner get payment : 65.00

```

和预想的一样，中介费收取了35%的中介费，剩下的交给了房东。

### 代码对应的类图

![代理模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c014f95933b?w=1718&h=964&f=png&s=25034)

> 从UML类图中我们可以看出，在这里没有使用抽象主题对象，而是用一个接口来分别让中介和房东实现。



## 优点

- 降低系统的耦合度：代理模式能够协调调用者和被调用者，在一定程度上降低了系 统的耦合度。
- 不同类型的代理可以对客户端对目标对象的访问进行不同的控制：
  - **远程代理**,使得客户端可以访问在远程机器上的对象，远程机器 可能具有更好的计算性能与处理速度，可以快速响应并处理客户端请求。
  - **虚拟代理**通过使用一个小对象来代表一个大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。
  - **保护代理**可以控制客户端对真实对象的使用权限。



## 缺点

- 由于在客户端和被代理对象之间增加了代理对象，因此可能会让客户端请求的速度变慢。



## Objective-C & Java的实践

- iOS SDK：NSProxy可以为持有的对象进行消息转发
- JDK：AOP下的``JDKDynamicAopProxy``是对JDK的动态代理进行了封装



# 五. 装饰者模式



## 定义

> 装饰模式(Decorator Pattern) ：不改变原有对象的前提下，动态地给一个对象增加一些额外的功能。



## 适用场景

- 动态地给一个对象增加职责（功能），这些职责（功能）也可以动态地被撤销。
- 当不能采用继承的方式对系统进行扩展或者采用继承不利于系统扩展和维护时。



## 成员与类图



### 成员

装饰者模式一共有四个成员：

1. 抽象构件（Component）：**抽象构件**定义一个对象（接口），可以动态地给这些对象添加职责。
2. 具体构件（Concrete Component）：**具体构件**是抽象构件的实例。
3. 装饰（Decorator）：**装饰类**也继承于抽象构件，它持有一个具体构件对象的实例，并实现一个与抽象构件接口一致的接口。
4. 具体装饰（Concrete Decorator）：**具体装饰**负责给具体构建对象实例添加上附加的责任。



### 模式类图



![装饰者模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c014f8afa39?w=1702&h=1046&f=png&s=35494)





## 代码示例

### 场景概述

模拟沙拉的制作：沙拉由沙拉底和酱汁两个部分组成，不同的沙拉底和酱汁搭配可以组成不同的沙拉。

| 沙拉底 | 价格 |
| ------ | ---- |
| 蔬菜   | 5    |
| 鸡肉   | 10   |
| 牛肉   | 16   |

| 酱汁   | 价格 |
| ------ | ---- |
| 醋汁   | 2    |
| 花生酱 | 4    |
| 蓝莓酱 | 6    |

> 注意：同一份沙拉底可以搭配多钟酱汁，而且酱汁的份数也可以不止一份。



### 场景分析

因为选择一个沙拉底之后，可以随意添加不同份数和种类的酱汁，也就是在原有的沙拉对象增加新的对象，所以比较适合用装饰者模式来设计：酱汁相当于装饰者，而沙拉底则是被装饰的构件。

下面我们用代码看一下如何实现该场景。

### 代码实现

首先我们定义 抽象构件，也就是沙拉类的基类``Salad``：

```objc
//================== Salad.h ==================

@interface Salad : NSObject

- (NSString *)getDescription;

- (double)price;

@end
```

> ``getDescription``和``price``方法用来描述当前沙拉的配置以及价格（因为随着装饰者的装饰，这两个数据会一直变化）。

下面我们再声明装饰者的基类``SauceDecorator``。按照装饰者设计模式类图，该类是继承于沙拉类的：

```objc
//================== SauceDecorator.h ==================

@interface SauceDecorator : Salad

@property (nonatomic, strong) Salad *salad;

- (instancetype)initWithSalad:(Salad *)salad;

@end

    

//================== SauceDecorator.m ==================
    
@implementation SauceDecorator

- (instancetype)initWithSalad:(Salad *)salad{
    
    self = [super init];
    
    if (self) {
        self.salad = salad;
    }
    return self;
}

@end
```

> 在装饰者的构造方法里面传入``Salad``类的实例，并将它保存下来，目的是为了在装饰它的时候用到。

现在抽象构件和装饰者的基类都创建好了，下面我们创建具体构件和具体装饰者。首先我们创建具体构件：

- 蔬菜沙拉
- 鸡肉沙拉
- 牛肉沙拉

蔬菜沙拉``VegetableSalad``：

```objc
//================== VegetableSalad.h ==================

@interface VegetableSalad : Salad

@end


 
//================== VegetableSalad.m ==================
    
@implementation VegetableSalad

- (NSString *)getDescription{
    return @"[Vegetable Salad]";
}

- (double)price{
    return 5.0;
}

@end
```

> 首先``getDescription``方法返回的是蔬菜沙拉底的描述；然后``price``方法返回的是它所对应的价格。

类似的，我们继续按照价格表来创建鸡肉沙拉底和牛肉沙拉底：

鸡肉沙拉底：

```objc
//================== ChickenSalad.h ==================

@interface ChickenSalad : Salad

@end


    
//================== ChickenSalad.m ==================
@implementation ChickenSalad

- (NSString *)getDescription{
    return @"[Chicken Salad]";
}

- (double)price{
    return 10.0;
}

@end
```

牛肉沙拉底：

```objc
//================== BeefSalad.h ==================

@interface BeefSalad : Salad

@end


    
//================== BeefSalad.m ==================
    
@implementation BeefSalad


- (NSString *)getDescription{
    return @"[Beef Salad]";
}

- (double)price{
    return 16.0;
}

@end
```

现在所有的被装饰者创建好了，下面我们按照酱汁的价格表来创建酱汁类（也就是具体装饰者）：

- 醋汁 
- 花生酱
- 蓝莓酱 

首先看一下醋汁``VinegarSauceDecorator``:

```objc
//================== VinegarSauceDecorator.h ==================

@interface VinegarSauceDecorator : SauceDecorator

@end

    

//================== VinegarSauceDecorator.m ==================    
    
@implementation VinegarSauceDecorator

- (NSString *)getDescription{
    return [NSString stringWithFormat:@"%@ + vinegar sauce",[self.salad getDescription]];
}

- (double)price{
    return [self.salad price] + 2.0;
}

@end
```

> 重写了``getDescription``方法，并添加了自己的装饰，即在原来的描述上增加了`` + vinegar sauce``字符串。之所以可以获取到原有的描述，是因为在构造方法里已经获取了被装饰者的对象（在装饰者基类中定义的方法）。同样地，价格也在原来的基础上增加了自己的价格。

现在我们知道了具体装饰者的设计，以此类推，我们看一下花生酱和蓝莓酱类如何定义：

花生酱``PeanutButterSauceDecorator``类：

```objc
//================== PeanutButterSauceDecorator.h ==================     

@interface PeanutButterSauceDecorator : SauceDecorator

@end


    
//================== PeanutButterSauceDecorator.m ==================     
@implementation PeanutButterSauceDecorator

- (NSString *)getDescription{
    return [NSString stringWithFormat:@"%@ + peanut butter sauce",[self.salad getDescription]];
}

- (double)price{
    return [self.salad price] + 4.0;
}

@end
```

蓝莓酱类``BlueBerrySauceDecorator``:

```objc
//================== BlueBerrySauceDecorator.h ==================     

@interface BlueBerrySauceDecorator : SauceDecorator

@end


 
//================== BlueBerrySauceDecorator.m ==================     
    
@implementation BlueBerrySauceDecorator
    
- (NSString *)getDescription{
    
    return [NSString stringWithFormat:@"%@ + blueberry sauce",[self.salad getDescription]];
}

- (double)price{
    
    return [self.salad price] + 6.0;
}

@end
```

OK，到现在所有的类已经定义好了，为了验证是否实现正确，下面用客户端尝试着搭配几种不同的沙拉吧：

1. 蔬菜加单份醋汁沙拉（7元）
2. 牛肉加双份花生酱沙拉（24元）
3. 鸡肉加单份花生酱再加单份蓝莓酱沙拉（20元）

首先我们看第一个搭配：

```objc
//================== client ==================     

//vegetable salad add vinegar sauce
Salad *vegetableSalad = [[VegetableSalad alloc] init];
NSLog(@"%@",vegetableSalad);

vegetableSalad = [[VinegarSauceDecorator alloc] initWithSalad:vegetableSalad];
NSLog(@"%@",vegetableSalad);
```

第一次打印输出：`` This salad is: [Vegetable Salad] and the price is: 5.00``
第二次打印输出：``This salad is: [Vegetable Salad] + vinegar sauce and the price is: 7.00``

上面代码中，我们首先创建了蔬菜底，然后再让醋汁装饰它（将蔬菜底的实例传入醋汁装饰者的构造方法中）。最后我们打印这个蔬菜底对象，描述和价格和装饰之前的确实发生了变化，说明我们的代码没有问题。

接着我们看第二个搭配：

```objc
//================== client ================== 

//beef salad add two peanut butter sauce:
Salad *beefSalad = [[BeefSalad alloc] init];
NSLog(@"%@",beefSalad);

beefSalad = [[PeanutButterSauceDecorator alloc] initWithSalad:beefSalad];
NSLog(@"%@",beefSalad);

beefSalad = [[PeanutButterSauceDecorator alloc] initWithSalad:beefSalad];
NSLog(@"%@",beefSalad);
```

第一次打印输出：``[Beef Salad] and the price is: 16.00``
第二次打印输出：``[Beef Salad] + peanut butter sauce and the price is: 20.00``
第三次打印输出：``[Beef Salad] + peanut butter sauce + peanut butter sauce and the price is: 24.00``

和上面的代码实现类似，都是先创建沙拉底（这次是牛肉底），然后再添加调料。由于是分两次装饰，所以要再写一次花生酱的装饰代码。对比每次打印的结果和上面的价格表可以看出输出是正确的。

这个例子是加了两次相同的酱汁，最后我们看第三个搭配，加入的是不同的两个酱汁：

```objc
//================== client ================== 

//chiken salad add peanut butter sauce and blueberry sauce
Salad *chikenSalad = [[ChickenSalad alloc] init];
NSLog(@"%@",chikenSalad);

chikenSalad = [[PeanutButterSauceDecorator alloc] initWithSalad:chikenSalad];
NSLog(@"%@",chikenSalad);

chikenSalad = [[BlueBerrySauceDecorator alloc] initWithSalad:chikenSalad];
NSLog(@"%@",chikenSalad);
```

第一次打印输出：``[Chicken Salad] and the price is: 10.00``
第二次打印输出：``[Chicken Salad] + peanut butter sauce and the price is: 14.00``
第三次打印输出：``[Chicken Salad] + peanut butter sauce + blueberry sauce and the price is: 20.00``

对比每次打印的结果和上面的价格表可以看出输出是正确的。

到这里，该场景就模拟结束了。可以试想一下，如果今后加了其他的沙拉底和酱汁的话，只需要分别继承``Salad``类和``SauceDecorator``类就可以了，现有的代码并不需要更改；而且经过不同组合可以搭配出更多种类的沙拉。

下面我们看一下该代码实现对应的类图。

### 代码对应的类图

![装饰者模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c014fa0e3cf?w=2586&h=1060&f=png&s=56023)



## 优点

- 比继承更加灵活：不同于在编译期起作用的继承；装饰者模式可以在运行时扩展一个对象的功能。另外也可以通过配置文件在运行时选择不同的装饰器，从而实现不同的行为。也可以通过不同的组合，可以实现不同效果。
- 符合“开闭原则”：装饰者和被装饰者可以独立变化。用户可以根据需要增加新的装饰类，在使用时再对其进行组合，原有代码无须改变。

## 缺点

- 装饰者模式需要创建一些具体装饰类，会增加系统的复杂度。



## Objective-C & Java的实践

- Objective-C中暂时未发现装饰者模式的实践，有知道的小伙伴可以留言
- JDK中：``BufferReader``继承了``Reader``，在``BufferReader``的构造器中传入了``Reader``，实现了装饰



# 六. 享元模式



## 定义

> 享元模式(Flyweight Pattern)：运用共享技术复用大量细粒度的对象,降低程序内存的占用,提高程序的性能。

定义解读：

- 享元模式的目的就是使用共享技术来实现大量细粒度对象的复用，提高性能。
- 享元对象能做到共享的关键是区分内部状态(Internal State)和外部状态(External State)。
  - **内部状态**是存储在享元对象内部并且不会随环境改变而改变的状态，因此内部状态可以共享。
  - **外部状态**是随环境改变而改变的、不可以共享的状态。享元对象的外部状态必须由客户端保存，并在享元对象被创建之后，在需要使用的时候再传入到享元对象内部。一个外部状态与另一个外部状态之间是相互独立的。



## 适用场景

- 系统有大量的相似对象，这些对象有一些外在状态。
- 应当在多次重复使用享元对象时才值得使用享元模式。使用享元模式需要维护一个存储享元对象的享元池，而这需要耗费资源，因此，



## 成员与类图



### 成员

享元模式一共有三个成员：

- 享元工厂（FlyweightFactory）: **享元工厂**提供一个用于存储享元对象的享元池，用户需要对象时，首先从享元池中获取，如果享元池中不存在，则创建一个新的享元对象返回给用户，并在享元池中保存该新增对象
- 抽象享元（Flyweight）：**抽象享元**定义了具体享元对象需要实现的接口。
- 具体享元（ConcreteFlyweight）: **具体享元**实现了抽象享元类定义的接口。



### 模式类图

![享元模式类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c015330858a?w=1674&h=834&f=png&s=21213)


## 代码示例



### 场景概述

这里我们使用[《Objective-C 编程之道：iOS设计模式解析》](https://book.douban.com/subject/6920082/)里的第21章使用的例子：在一个页面展示数百个大小，位置不同的花的图片，然而这些花的样式只有6种。

看一下截图：
![百花图](https://user-gold-cdn.xitu.io/2018/11/27/16755c016e55e306?w=207&h=448&f=png&s=221998)

### 场景分析

由于这里我们需要创建很多对象，而这些对象有可以共享的内部状态（6种图片内容）以及不同的外部状态（随机的，数百个位置坐标和图片大小），因此比较适合使用享元模式来做。

根据上面提到的享元模式的成员：

- 我们需要创建一个工厂类来根据花的类型来返回花对象（这个对象包括内部可以共享的图片以及外部状态位置和大小）：每次当新生成一种花的类型的对象的时候就把它保存起来，因为下次如果还需要这个类型的花内部图片对象的时候就可以直接用了。
- 抽象享元类就是Objective-C的原生``UIImageView``,它可以显示图片
- 具体享元类可以自己定义一个类继承于``UIImageView``，因为后续我们可以直接添加更多其他的属性。



下面我们看一下用代码如何实现：

### 代码实现

首先我们创建一个工厂，这个工厂可以根据所传入花的类型来返回花内部图片对象，在这里可以直接使用原生的``UIImage``对象，也就是图片对象。而且这个工厂持有一个保存图片对象的池子：

- 当该类型的花第一次被创建时，工厂会新建一个所对应的花内部图片对象，并将这个对象放入池子中保存起来。
- 当该类型的花内部图片对象在池子里已经有了，那么工厂则直接从池子里返回这个花内部图片对象。

下面我们看一下代码是如何实现的：

```objc
//================== FlowerFactory.h ================== 

typedef enum 
{
  kAnemone,
  kCosmos,
  kGerberas,
  kHollyhock,
  kJasmine,
  kZinnia,
  kTotalNumberOfFlowerTypes
    
} FlowerType;

@interface FlowerFactory : NSObject 

- (FlowerImageView *) flowerImageWithType:(FlowerType)type

@end




//================== FlowerFactory.m ================== 
    
@implementation FlowerFactory
{
    NSMutableDictionary *_flowersPool;
}

- (FlowerImageView *) flowerImageWithType:(FlowerType)type
{
    
  if (_flowersPool == nil){
      
     _flowersPool = [[NSMutableDictionary alloc] initWithCapacity:kTotalNumberOfFlowerTypes];
  }
  
  //尝试获取传入类型对应的花内部图片对象
  UIImage *flowerImage = [_flowersPool objectForKey:[NSNumber numberWithInt:type]];
  
  //如果没有对应类型的图片，则生成一个
  if (flowerImage == nil){
    
    NSLog(@"create new flower image with type:%u",type);
      
    switch (type){
            
      case kAnemone:
        flowerImage = [UIImage imageNamed:@"anemone.png"];
        break;
      case kCosmos:
        flowerImage = [UIImage imageNamed:@"cosmos.png"];
        break;
      case kGerberas:
        flowerImage = [UIImage imageNamed:@"gerberas.png"];
        break;
      case kHollyhock:
        flowerImage = [UIImage imageNamed:@"hollyhock.png"];
        break;
      case kJasmine:
        flowerImage = [UIImage imageNamed:@"jasmine.png"];
        break;
      case kZinnia:
        flowerImage = [UIImage imageNamed:@"zinnia.png"];
        break;
      default:
        flowerImage = nil;
        break;
    
    }
      
    [_flowersPool setObject:flowerImage forKey:[NSNumber numberWithInt:type]];
      
  }else{
      //如果有对应类型的图片，则直接使用
      NSLog(@"reuse flower image with type:%u",type);
  }
    
  //创建花对象，将上面拿到的花内部图片对象赋值并返回
  FlowerImageView *flowerImageView = [[FlowerImageView alloc] initWithImage:flowerImage];
    
  return flowerImageView;
}
```

> - 在这个工厂类里面定义了六中图片的类型
> - 该工厂类持有``_flowersPool``私有成员变量，保存新创建过的图片。
> - ``flowerImageWithType:``的实现：结合了``_flowersPool``：当``_flowersPool``没有对应的图片时，新创建图片并返回；否则直接从``_flowersPool``获取对应的图片并返回。

接着我们定义这些花对象``FlowerImageView``：

```objc
//================== FlowerImageView.h ================== 

@interface FlowerImageView : UIImageView 

@end


//================== FlowerImageView.m ================== 
    
@implementation FlowerImageView

@end
```

> 在这里面其实也可以直接使用``UIImageView``，之所以创建一个子类是为了后面可以更好地扩展这些花独有的一些属性。
>
> 注意一下花对象和花内部图片对象的区别：花对象``FlowerImageView``是包含花内部图片对象的``UIImage``。因为在Objective-C里面，``UIImage``是``FlowerImageView``所继承的``UIImageView``的一个属性，所以在这里``FlowerImageView``就直接包含了``UIImage``。



下面我们来看一下客户端如何使用``FlowerFactory``和``FlowerImageView``这两个类：

```objc
//================== client ================== 

//首先建造一个生产花内部图片对象的工厂
FlowerFactory *factory = [[FlowerFactory alloc] init];

for (int i = 0; i < 500; ++i)
{
    //随机传入一个花的类型，让工厂返回该类型对应花类型的花对象
    FlowerType flowerType = arc4random() % kTotalNumberOfFlowerTypes;
    FlowerImageView *flowerImageView = [factory flowerImageWithType:flowerType];

    // 创建花对象的外部属性值（随机的位置和大小）
    CGRect screenBounds = [[UIScreen mainScreen] bounds];
    CGFloat x = (arc4random() % (NSInteger)screenBounds.size.width);
    CGFloat y = (arc4random() % (NSInteger)screenBounds.size.height);
    NSInteger minSize = 10;
    NSInteger maxSize = 50;
    CGFloat size = (arc4random() % (maxSize - minSize + 1)) + minSize;

    //将位置和大小赋予花对象
    flowerImageView.frame = CGRectMake(x, y, size, size);

    //展示这个花对象
    [self.view addSubview:flowerImageView];
}
```

上面代码里面是生成了500朵位置和大小都是随机的花内部图片对象。这500朵花最主要的区别还是它们的位置和大小；而它们使用的花的图片对象只有6个，因此可以用专门的``Factory``来生成和管理这些少数的花内部图片对象，从工厂的打印我们可以看出来：

```
create new flower image with type:1
create new flower image with type:3
create new flower image with type:4
reuse flower image with type:3
create new flower image with type:5
create new flower image with type:2
create new flower image with type:0
reuse flower image with type:5
reuse flower image with type:5
reuse flower image with type:4
reuse flower image with type:1
reuse flower image with type:3
reuse flower image with type:4
reuse flower image with type:0

```

从上面的打印结果可以看出，在六种图片都创建好以后，再获取时就直接拿生成过的图片了，在一定程度上减少了内存的开销。

下面我们来看一下该代码示例对应的UML类图。

### 代码对应的类图

![享元模式代码示例类图](https://user-gold-cdn.xitu.io/2018/11/27/16755c016fb5f173?w=1081&h=435&f=png&s=36238)


> 这里需要注意的是
>
> - 工厂和花对象是组合关系：``FlowerFactroy``生成了多个``FlowerImageView``对象，也就是花的内部图片对象，二者的关系属于强关系，因为在该例子中二者如果分离而独立存在都将会失去意义，所以在UML类图中用了组合的关系（实心菱形）。
> - 抽象享元类是``UIImageView``，它的一个内部对象是``UIImage``（这两个都是Objective-C原生的关于图片的类）。
> - 客户端依赖的对象是工厂对象和花对象，而对花的内部图片对象``UIImage``可以一无所知，因为它是被``FlowerFactroy``创建并被``FlowerImageView``所持有的。（但是因为``UIImage``是``FlowerImageView``的一个外部可以引用的属性，所以在这里客户端还是可以访问到``UIImage``，这是Objective-C原生的实现。后面我们在用享元模式的时候可以不将内部属性暴露出来）



## 优点

- 使用享元模可以减少内存中对象的数量，使得相同对象或相似对象在内存中只保存一份，降低系统的使用内存，也可以提性能。
- 享元模式的外部状态相对独立，而且不会影响其内部状态，从而使得享元对象可以在不同的环境中被共享。



## 缺点

- 使用享元模式需要分离出内部状态和外部状态，这使得程序的逻辑复杂化。

- 对象在缓冲池中的复用需要考虑线程问题。


## Objective-C & Java的实践

- iOS SDK中的``UITableViewCell``的复用池就是使用享元模式的一个例子。
- Java：JDK中的``Integer``类的``valueOf``方法，如果传入的值的区间在``[IntegerCache.low,IntegerCache.high]``中的话，则直接从缓存里获取；否则就创建一个新的``Integer``。





------

到这里设计模式中的结构型模式就介绍完了，读者可以结合UML类图和demo的代码来理解每个设计模式的特点和相互之间的区别，希望读者可以有所收获。

另外，本篇博客的代码和类图都保存在我的GitHub库中：[knightsj:object-oriented-design](https://github.com/knightsj/object-oriented-design)中Chapter2的2.2小节。



下一篇是面向对象系列的第四篇，讲解的是面向对象设计模式中的行为型模式。



该系列前面的两篇文章：

>该系列前面的两篇文章：
>- [面向对象设计的六大设计原则（附 Demo 及 UML 类图）](https://knightsj.github.io/2018/09/09/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E7%9A%84%E5%85%AD%E5%A4%A7%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99%EF%BC%88%E9%99%84%20Demo%20%E5%8F%8A%20UML%20%E7%B1%BB%E5%9B%BE%EF%BC%89/)
>- [面向对象设计的设计模式（一）：创建型设计模式（附 Demo 及 UML 类图）](https://knightsj.github.io/2018/10/21/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9A%E5%88%9B%E5%BB%BA%E5%9E%8B%E6%A8%A1%E5%BC%8F%EF%BC%88%E9%99%84%20Demo%20%E5%8F%8A%20UML%20%E7%B1%BB%E5%9B%BE%EF%BC%89/)



# 参考书籍和教程

- [《设计模式 可复用面向对象软件的基础》](https://book.douban.com/subject/1052241/)
- [《Objective-C 编程之道：iOS设计模式解析》](https://book.douban.com/subject/6920082/)
- [《Head First 设计模式》](https://book.douban.com/subject/2243615/)
- [《大话设计模式》](https://book.douban.com/subject/2334288/)
- [慕课网实战课程：java设计模式精讲 Debug 方式+内存分析](https://coding.imooc.com/learn/list/270.html)


---


笔者在近期开通了个人公众号，主要分享编程，读书笔记，思考类的文章。

- **编程类**文章：包括笔者以前发布的精选技术文章，以及后续发布的技术文章（以原创为主），并且逐渐脱离 iOS 的内容，将侧重点会转移到**提高编程能力**的方向上。
- **读书笔记类**文章：分享**编程类**，**思考类**，**心理类**，**职场类**书籍的读书笔记。
- **思考类**文章：分享笔者平时在**技术上**，**生活上**的思考。

>因为公众号每天发布的消息数有限制，所以到目前为止还没有将所有过去的精选文章都发布在公众号上，后续会逐步发布的。

**而且因为各大博客平台的各种限制，后面还会在公众号上发布一些短小精干，以小见大的干货文章哦~**

扫下方的公众号二维码并点击关注，期待与您的共同成长~

![](https://user-gold-cdn.xitu.io/2018/7/17/164a4229c04303c0?w=378&h=392&f=jpeg&s=16274)

