---
title: MBProgressHUD源码解析
tags: [iOS,Objective-C,源码解析]
categories: iOS
---

听过好多次：“程序员要通过多读好的源码来提升自己”这样类似的话，而且又觉得自己有很多不会的，于是就马上启动了自己的**读好源码Project**。



从哪个框架开始呢？我想到了``SDWebImage``，但是大致看下来文件很多，代码也不少，不知道从何看起，于是作罢。所以茅塞顿开，还是从最最简单的框架开始吧～因为学习曲线要给自己设定得平缓一点才有利于稳步提升，小步快跑才是王道～



<!-- more -->



找着找着就找到了``MBProgressHUD``，这个框架只有两个文件，一个头文件和一个实现文件，很适合我现在的水平（对于一个没怎么读过源码的选手），于是就撸起了袖子开始了。

连查知识点带记笔记一共花了大概3个小时（虽然文件很少，但是里面好多东西都不知道[捂脸]）。整体说来，收获还是比较大的，除了一些零碎的语法之外，框架作者对于代码结构的设计和各种情况的考虑还是很出色的，很值得学习，而且我在下文也有介绍。

这篇总结主要分三个部分来介绍这个框架：
1. 核心Public API
2. 方法调用流程图
3. 方法内部实现

不多说了，开始吧～



## 1. 核心Public API

### 属性：

```objc
@property (assign, nonatomic) MBProgressHUDMode mode;//HUD的类型
@property (assign, nonatomic) MBProgressHUDAnimation animationType UI_APPEARANCE_SELECTOR;//动画类型
@property (assign, nonatomic) NSTimeInterval graceTime;//show函数触发到显示HUD的时间段
@property (assign, nonatomic) NSTimeInterval minShowTime;//HUD显示的最短时间
```

### 类方法：

```objc
/**
 * 在某个view上添加HUD并显示
 *
 * 注意：显示之前，先去掉在当前view上显示的HUD。这个做法很严谨，我们将这个方案抽象出来：如果一个模型是这样的：我们需要将A加入到B中，但是需求上B里面只允许只有一个A。那么每次将A添加到B之前，都要先判断当前的b里面是否有A，如果有，则移除。
 */
+ (instancetype)showHUDAddedTo:(UIView *)view animated:(BOOL)animated;
/**
 * 找到某个view上最上层的HUD并隐藏它。
 * 如果返回值是YES的话，就表明HUD被找到而且被移除了。
 */
+ (BOOL)hideHUDForView:(UIView *)view animated:(BOOL)animated;
/**
 * 在某个view上找到最上层的HUD并返回它。
 * 返回值可以是空，所以返回值的关键字为：nullable
 */
+ (nullable MBProgressHUD *)HUDForView:(UIView *)view;
```

### 对象方法：

```objc
/**
 * 一个HUD的便利构造函数，用某个view来初始化HUD：这个view的bounds就是HUD的bounds
 */
- (instancetype)initWithView:(UIView *)view;
/** 
 * 显示HUD，有无动画。
 */
- (void)showAnimated:(BOOL)animated;
/** 
 * 隐藏HUD，有无动画。
 */
- (void)hideAnimated:(BOOL)animated;
/** 
 * 在delay的时间过后隐藏HUD，有无动画。
 */
- (void)hideAnimated:(BOOL)animated afterDelay:(NSTimeInterval)delay;
```

看完了这些比较主要的API，我们看一下方法调用的流程图：

## 2. 方法调用流程图：

总体来说，这个第三方框架的接口还是比较整齐的，可以大致上分为两类：显示（show）和隐藏（hide）。而且无论是调用显示方法还是隐藏方法，最终都会走到私有方法``animateIn:withType: completion:``里（前提是附加动画效果）。可以看一下方法调用的流程图：

![方法调用流程图](http://upload-images.jianshu.io/upload_images/859001-fe3f0f393bcc3b9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看完方法调用的结构之后，我们来具体看一下方法内部是如何实现的：

## 3. 方法内部实现：

在讲解API之前，有必要先介绍一下HUD使用的三个Timer。

```objc
@property (nonatomic, weak) NSTimer *graceTimer; //执行一次：在show方法触发后到HUD真正显示之前,前提是设定了graceTime，默认为0
@property (nonatomic, weak) NSTimer *minShowTimer;//执行一次：在HUD显示后到HUD被隐藏之前
@property (nonatomic, weak) NSTimer *hideDelayTimer;//执行一次：在HUD被隐藏的方法触发后到真正隐藏之前
```

- graceTimer：用来推迟HUD的显示。如果设定了graceTime，那么HUD会在``show``方法触发后的graceTime时间后显示。它的意义是：如果任务完成所消耗的时间非常短并且短于graceTime，则HUD就不会出现了，避免HUD一闪而过的差体验。
- minShowTimer：如果设定了minShowTime，就会在``hide``方法触发后判断任务执行的时间是否短于minShowTime。因此即使任务在minShowTime之前完成了，HUD也不会立即消失，它会在走完minShowTime之后才消失，这应该也是避免HUD一闪而过的情况。
- hideDelayTimer：用来推迟HUD的隐藏。如果设定了delayTime，那么在触发``hide``方法后HUD也不会立即隐藏，它会在走完delayTime之后才隐藏。

这三者的关系可以由下面这张图来体现（并没有包含所有的情况）：


![三种timer](http://upload-images.jianshu.io/upload_images/859001-c9f49bfcec64dd0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面开始分别讲解``show``系列的方法和``hide``系列的方法。

### show系列方法

```objc
+ (instancetype)showHUDAddedTo:(UIView *)view animated:(BOOL)animated {
    MBProgressHUD *hud = [[self alloc] initWithView:view];// 接着调用 [self initWithFrame:view.bounds]：根据传进来的view的frame来设定自己的frame
    hud.removeFromSuperViewOnHide = YES;//removeFromSuperViewOnHide 应该是一个标记，表明HUD自己处于“应该被移除的状态”
    [view addSubview:hud];//在view上将自己的实例添加上去
    [hud showAnimated:animated];
    return hud;
}
//调用showAnimated：
- (void)showAnimated:(BOOL)animated {
    MBMainThreadAssert();
    [self.minShowTimer invalidate];//取消当前的minShowTimer
     self.useAnimation = animated;//设置animated状态
     self.finished = NO;//添加标记：表明当前任务仍在进行
    // 如果设定了graceTime，就要推迟HUD的显示
    if (self.graceTime > 0.0) {
        NSTimer *timer = [NSTimer timerWithTimeInterval:self.graceTime target:self selector:@selector(handleGraceTimer:) userInfo:nil repeats:NO];
        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
        self.graceTimer = timer;
    } 
    // ... otherwise show the HUD immediately
    else {
        [self showUsingAnimation:self.useAnimation];
    }
}
//self.graceTimer触发的方法
- (void)handleGraceTimer:(NSTimer *)theTimer {
    // Show the HUD only if the task is still running
    if (!self.hasFinished) {
        [self showUsingAnimation:self.useAnimation];
    }
}
//所有的show方法最终都会走到这个方法
- (void)showUsingAnimation:(BOOL)animated {
    // Cancel any previous animations : 移走所有的动画
    [self.bezelView.layer removeAllAnimations];
    [self.backgroundView.layer removeAllAnimations];
    // Cancel any scheduled hideDelayed: calls :取消delay的timer
    [self.hideDelayTimer invalidate];
    //记忆开始的时间
    self.showStarted = [NSDate date];
    self.alpha = 1.f;
    // Needed in case we hide and re-show with the same NSProgress object attached.
    [self setNSProgressDisplayLinkEnabled:YES];
    if (animated) {        
        [self animateIn:YES withType:self.animationType completion:NULL];   
    } else {
        //方法弃用警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        self.bezelView.alpha = self.opacity;
#pragma clang diagnostic pop
        self.backgroundView.alpha = 1.f;
    }
}
```

>我们可以看到，无论是类方法的show方法，还是对象方法的show方法，而且无论是触发了``graceTimer``还是没有触发，最后都会走到``showUsingAnimation:``方法来让HUD显示出来。

这里补充讲解一下NSProgress的监听方法：

```objc
- (void)setNSProgressDisplayLinkEnabled:(BOOL)enabled {
    // 这里使用 CADisplayLink 来刷新progress的变化。因为如果使用kvo机制来监听的话可能会非常消耗主线程（因为频率可能非常快）。
    if (enabled && self.progressObject) {
        // Only create if not already active.
        if (!self.progressObjectDisplayLink) {
            self.progressObjectDisplayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(updateProgressFromProgressObject)];
        }
    } else {
        //不刷新
        self.progressObjectDisplayLink = nil;
    }
}
```

>CADisplayLink是一个能让我们以和屏幕刷新率同步的频率将特定的内容画到屏幕上的定时器类。 CADisplayLink以特定模式注册到runloop后， 每当屏幕显示内容刷新结束的时候，runloop就会向 CADisplayLink指定的target发送一次指定的selector消息，  CADisplayLink类对应的selector就会被调用一次。
>参考文章：[Core Animation系列之CADisplayLink](http://www.tuicool.com/articles/meMVR3)

### hide系列方法

```objc
+ (BOOL)hideHUDForView:(UIView *)view animated:(BOOL)animated {
    MBProgressHUD *hud = [self HUDForView:view];//获取当前view的最前为止的HUD
    if (hud != nil) {
        hud.removeFromSuperViewOnHide = YES;
        [hud hideAnimated:animated];
        return YES;
    }
    return NO;
}
+ (MBProgressHUD *)HUDForView:(UIView *)view {   
    NSEnumerator *subviewsEnum = [view.subviews reverseObjectEnumerator]; //倒叙排序
    for (UIView *subview in subviewsEnum) {
        if ([subview isKindOfClass:self]) {
            return (MBProgressHUD *)subview;
        }
    }
    return nil;
}
- (void)hideAnimated:(BOOL)animated {
    MBMainThreadAssert();
    [self.graceTimer invalidate];
     self.useAnimation = animated;
     self.finished = YES;
     //如果设定了HUD最小显示时间，那就需要判断最小显示时间和已经经过的时间的大小
     if (self.minShowTime > 0.0 && self.showStarted) {
        NSTimeInterval interv = [[NSDate date] timeIntervalSinceDate:self.showStarted];
        
        //如果最小显示时间比较大，则暂时不触发HUD的隐藏，而是启动一个timer，再经过二者的时间差的时间之后再触发隐藏
        if (interv < self.minShowTime) {
            NSTimer *timer = [NSTimer timerWithTimeInterval:(self.minShowTime - interv) target:self selector:@selector(handleMinShowTimer:) userInfo:nil repeats:NO];
            [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
            self.minShowTimer = timer;
            return;
        } 
     }
    //如果最小显示时间比较小，则立即将HUD隐藏
    [self hideUsingAnimation:self.useAnimation];
}
//self.minShowTimer触发的方法
- (void)handleMinShowTimer:(NSTimer *)theTimer {
    [self hideUsingAnimation:self.useAnimation];
}
- (void)hideUsingAnimation:(BOOL)animated {
    if (animated && self.showStarted) {
        //隐藏时，将showStarted设为nil
        self.showStarted = nil;
        [self animateIn:NO withType:self.animationType completion:^(BOOL finished) {
            [self done];
        }];
    } else {
        self.showStarted = nil;
        self.bezelView.alpha = 0.f;
        self.backgroundView.alpha = 1.f;
        [self done];
    }
}
```

>我们可以看到，无论是类方法的``hide``方法，还是对象方法的``hide``方法，而且无论是触发还是没有触发``minShowTimer``,最终都会走到``hideUsingAnimation``这个方法里。

而无论是``show``方法，还是``hide``方法，在设定animated属性为YES的前提下，最终都会走到``animateIn: withType: completion:``方法：

```objc
- (void)animateIn:(BOOL)animatingIn withType:(MBProgressHUDAnimation)type completion:(void(^)(BOOL finished))completion {
    // Automatically determine the correct zoom animation type
    if (type == MBProgressHUDAnimationZoom) {
        type = animatingIn ? MBProgressHUDAnimationZoomIn : MBProgressHUDAnimationZoomOut;
    }
    //()内代表x和y方向缩放倍数
    CGAffineTransform small = CGAffineTransformMakeScale(0.5f, 0.5f);
    CGAffineTransform large = CGAffineTransformMakeScale(1.5f, 1.5f);
    // 设定初始状态
    UIView *bezelView = self.bezelView;
    if (animatingIn && bezelView.alpha == 0.f && type == MBProgressHUDAnimationZoomIn) {
        bezelView.transform = small;
    } else if (animatingIn && bezelView.alpha == 0.f && type == MBProgressHUDAnimationZoomOut) {
        bezelView.transform = large;
    }
    // 创建动画任务
    dispatch_block_t animations = ^{
        if (animatingIn) {
            bezelView.transform = CGAffineTransformIdentity;//重置
        } else if (!animatingIn && type == MBProgressHUDAnimationZoomIn) {
            bezelView.transform = large;
        } else if (!animatingIn && type == MBProgressHUDAnimationZoomOut) {
            bezelView.transform = small;
        }
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        bezelView.alpha = animatingIn ? self.opacity : 0.f;
#pragma clang diagnostic pop
       //如果animatingIn是true，就是show方法，否则是hide方法
        self.backgroundView.alpha = animatingIn ? 1.f : 0.f;
    };
    // Spring animations are nicer, but only available on iOS 7+
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 70000 || TARGET_OS_TV
    if (kCFCoreFoundationVersionNumber >= kCFCoreFoundationVersionNumber_iOS_7_0) {
        //执行动画 >= iOS7
        [UIView animateWithDuration:0.3 delay:0. usingSpringWithDamping:1.f initialSpringVelocity:0.f options:UIViewAnimationOptionBeginFromCurrentState animations:animations completion:completion];
        return;
    }
#endif
    [UIView animateWithDuration:0.3 delay:0. options:UIViewAnimationOptionBeginFromCurrentState animations:animations completion:completion];
}
```

-----

除了一些细节上的语法之外，我觉得该框架有几个地方值得我们借鉴：
1. 暴露出来的API最终都会走到同一个私有方法里。
2. 将真正显示的时间的前后加上缓冲的时间（graceTimer 和 hideDelayTimer），可以提高可定制性和稳定性。
3. 如果有两个方法是矛盾的，并且可以同时调用，就需要在全局设置一个属性来判断当前的状态（removeFromSuperViewOnHide属性，finished属性）
4. 使用CADisplayLink来刷新更新频率可能很高的view。
5. 使用NSAssert来捕获各种异常。

就这样大致写完了，没有怎么读过第三方框架的源码，所以第一次可能显得稍许不足。有不好的地方还希望多多指点哈～


