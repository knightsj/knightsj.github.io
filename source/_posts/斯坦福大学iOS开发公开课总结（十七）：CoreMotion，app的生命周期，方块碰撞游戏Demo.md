---
title: 斯坦福大学iOS开发公开课总结（十七）：CoreMotion，app的生命周期，方块碰撞游戏Demo
tags: [iOS,Objective-C]
categories: iOS
---

本节课讲解了Core Motion框架的知识，简单介绍了app的生命周期，最后用一个方块碰撞游戏来对本节课的知识作总结。

# Core Motion
--------

CoreMotion是一个专门处理设备“动作”的框架，其中包含了加速度计，陀螺仪和磁力针。加速计由三个坐标轴决定，用户最常见的操作设备的动作移动，晃动手机(摇一摇)，倾斜手机都可以被设备检测到，加速计可以检测到线性的变化。陀螺仪可以更好的检测到偏转的动作，可以根据用户的动作做出相应的动作；磁力针可以判断设备的方向。

CoreMotion的工作是基于``CMMotionManager``类来执行的。我们看一下该类的API：

## CMMotionManager

### 检测硬件设备：
```
@property(readonly, nonatomic, getter=isAccelerometerAvailable) BOOL accelerometerAvailable __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isGyroAvailable) BOOL gyroAvailable __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isMagnetometerAvailable) BOOL magnetometerAvailable NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isDeviceMotionAvailable) BOOL deviceMotionAvailable __TVOS_PROHIBITED;
```

<!-- more -->

### 开启相应的模块：

```
- (void)startAccelerometerUpdates __TVOS_PROHIBITED;
- (void)startGyroUpdates __TVOS_PROHIBITED;
- (void)startMagnetometerUpdates NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
- (void)startDeviceMotionUpdates __TVOS_PROHIBITED;
```

### 检测相应的模块是否正在收集数据：
```
@property(readonly, nonatomic, getter=isAccelerometerActive) BOOL accelerometerActive __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isGyroActive) BOOL gyroActive __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isMagnetometerActive) BOOL magnetometerActive NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
@property(readonly, nonatomic, getter=isDeviceMotionActive) BOOL deviceMotionActive __TVOS_PROHIBITED;
```

### 关闭相应的模块：
```
- (void)stopAccelerometerUpdates __TVOS_PROHIBITED;
- (void)stopGyroUpdates __TVOS_PROHIBITED;
- (void)stopMagnetometerUpdates NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
- (void)stopDeviceMotionUpdates __TVOS_PROHIBITED; 
```

### 使用block监听
```
- (void)startAccelerometerUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMAccelerometerHandler)handler __TVOS_PROHIBITED;
- (void)startGyroUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMGyroHandler)handler __TVOS_PROHIBITED;
- (void)startMagnetometerUpdatesToQueue:(NSOperationQueue *)queue withHandler:(CMMagnetometerHandler)handler NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
- (void)startDeviceMotionUpdatesUsingReferenceFrame:(CMAttitudeReferenceFrame)referenceFrame toQueue:(NSOperationQueue *)queue withHandler:(CMDeviceMotionHandler)handler NS_AVAILABLE(NA,5_0) __TVOS_PROHIBITED;
```
### CMMotionManager的工作步骤
1. 首先初始化``CMMotionManager``类。
2. 判断硬件设备的可使用性。
3. 调用API
  详细的使用方法请看Demo部分的讲解部分。

# app的生命周期
---------

程序在运行过程中，是由各种不同的状态的，在这些状态之间切换时可以执行一些代码用于满足一定的业务需求。

## 应用程序的几种状态：
- Not running:未运行  程序没启动
- Inactive:未激活, 程序在前台运行，不过没有接收到事件。在没有事件处理情况下程序通常停留在这个状态。
- Active:激活,  程序在前台运行而且接收到了事件。这是程序在前台运行的正常状态。
- Backgroud:后台, 程序在后台而且能执行代码。大多数程序进入这个状态后会在在这个状态上停留一会， 时间到之后会进入挂起状态(Suspended)。
- Suspended:挂起 ,程序在后台不能执行代码。系统会自动把程序变成这个状态而且不会发出通知。当挂起时，程序还是停留在内存中的，当系统内存低时，系统就把挂起的程序清除掉，为当前处于前台程序提供更多的内存。

## 应用程序的状态之间切换时调用的代理方法（AppDelegate）：
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions   //启动基本完成，程序准备开始运行时调用。
- (void)applicationWillResignActive:(UIApplication *)application          //当应用程序将要入非活动状态执行。在此期间，应用程序不接收消息或事件，比如来电话时。
- (void)applicationDidBecomeActive:(UIApplication *)application           //当应用程序入活动状态执行。
- (void)applicationDidEnterBackground:(UIApplication *)application        //当程序被推到后台的时候调用。
- (void)applicationWillEnterForeground:(UIApplication *)application       //当程序从后台将要重新回到前台时候调用。
- (void)applicationWillTerminate:(UIApplication *)application             // 当程序将要退出时调用，通常是用来保存数据和一些退出前的清理工作。
```

# Demo 
----

## Demo需求:
- 显示两个小方块，一黑一红，二者可以随着屏幕的转动而移动。
- 由红色方块碰撞黑色方块来得分。

## Demo效果图：

![左：游戏进行中 | 右：游戏暂停](http://upload-images.jianshu.io/upload_images/859001-baa2c6607140ad2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要知识点和代码段：

#### 1. 添加animator和重力行为和碰撞行为
```
- (UIDynamicAnimator *)animator
{
    if (!_animator) _animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
    return _animator;
}
- (UICollisionBehavior *)collider
{
    if (!_collider) {
        UICollisionBehavior *collider = [[UICollisionBehavior alloc] init];
        collider.translatesReferenceBoundsIntoBoundary = YES;
        [self.animator addBehavior:collider];
        self.collider = collider;
    }
    return _collider;
}
- (UIGravityBehavior *)gravity
{
    if (!_gravity) {
        UIGravityBehavior *gravity = [[UIGravityBehavior alloc] init];
        [self.animator addBehavior:gravity];
        self.gravity = gravity;
    }
    return _gravity;
}
```
>老规矩，还是要将animtor添加到view里面再向其添加各种动作行为。有关动画的讲解请参考笔者之前一篇总结：[斯坦福大学iOS开发公开课总结（八） ：协议，block，动画，俄罗斯方块Demo](http://www.jianshu.com/p/5f1f40f963ac)。

#### 2. 根据偏移量来设置方块的位置
```
- (UIView *)addBlockOffsetFromCenterBy:(UIOffset)offset
{
    CGPoint blockCenter = CGPointMake(CGRectGetMidX(self.view.bounds)+offset.horizontal,
                                      CGRectGetMidY(self.view.bounds)+offset.vertical);

    CGRect blockFrame = CGRectMake(blockCenter.x-blockSize.width/2,
                                   blockCenter.y-blockSize.height/2,
                                   blockSize.width,
                                   blockSize.height);

    UIView *block = [[UIView alloc] initWithFrame:blockFrame];
    [self.view addSubview:block];
    return block;
}
```
>在这里，使用了``CGRectGetMidX``函数来获得view的横向中心点。

#### 3. 初始化红色和黑色方块
```
        self.redBlock = [self addBlockOffsetFromCenterBy:UIOffsetMake(-100, 0)];
        self.redBlock.backgroundColor = [UIColor redColor];
        [self.collider addItem:self.redBlock];
        [self.gravity addItem:self.redBlock];
        
        self.blackBlock = [self addBlockOffsetFromCenterBy:UIOffsetMake(+100, 0)];
        self.blackBlock.backgroundColor = [UIColor blackColor];
        [self.collider addItem:self.blackBlock];

        //将开始的重力设为0：方块将不感受重力，只能通过人为手段施加
        self.gravity.gravityDirection = CGVectorMake(0, 0);
```
>在这里，红色方块具有碰撞和重力行为，但是黑色方块却只有碰撞行为。因为黑色是不受重力控制的，它的运动的触发只来自红色方块的碰撞。

#### 4. 在程序即将要挂起或者恢复前台活动状态时进行暂停游戏的操作：
```
    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationWillResignActiveNotification
                                                      object:nil
                                                       queue:nil
                                                  usingBlock:^(NSNotification *note) {
                                                      [self pauseGame];
                                                  }];

    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidBecomeActiveNotification
                                                     object:nil
                                                       queue:nil
                                                  usingBlock:^(NSNotification *note) {
                                                      if (self.view.window) [self resumeGame];
                                                  }];
```
>我们也可以在appdelegate里面来实现这些方法，不过个人认为将方法写在一起看起来比较直观

#### 5. CMMotionManager的初始化和使用
```
//初始化
- (CMMotionManager *)motionManager
{
    if (!_motionManager) {
        _motionManager = [[CMMotionManager alloc] init];
        _motionManager.accelerometerUpdateInterval = 0.1;
    }
    return _motionManager;
}

//将方法放在主线程的代码块中
 if (!self.motionManager.isAccelerometerActive) {

        [self.motionManager startAccelerometerUpdatesToQueue:[NSOperationQueue mainQueue]

                                                 withHandler:^(CMAccelerometerData *accelerometerData, NSError *error) {
    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿//获取加速器的方向
                                                     CGFloat x = accelerometerData.acceleration.x;
                                                     CGFloat y = accelerometerData.acceleration.y;
    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿
    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿    ﻿//根据设备的方向来给方块施加不同方向的重力
                                                     switch (self.interfaceOrientation) {
                                                         case UIInterfaceOrientationLandscapeRight:
                                                            self.gravity.gravityDirection = CGVectorMake(- y, - x); break;
  
                                                       case UIInterfaceOrientationLandscapeLeft:
                                                             self.gravity.gravityDirection = CGVectorMake(y, x); break;

                                                         case UIInterfaceOrientationPortrait:
                                                             self.gravity.gravityDirection = CGVectorMake(x, - y); break;

                                                         case UIInterfaceOrientationPortraitUpsideDown:
                                                            self.gravity.gravityDirection = CGVectorMake(- x, y); break;
                                                     }

                                                     [self updateScore];
                                                 }];
    }
```

# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，可以进入我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)下载哦~    这一系列到现在为止的所有Demo都在里面，分为英文注释版本和中文注释版本两种。

如果嫌麻烦的童鞋可以在留言留下邮箱，笔者会将Demo包发给你~

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~




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



