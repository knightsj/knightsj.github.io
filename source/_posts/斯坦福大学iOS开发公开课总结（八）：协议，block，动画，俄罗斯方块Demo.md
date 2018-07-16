---
title: 斯坦福大学iOS开发公开课总结（八）：协议，block，动画，俄罗斯方块Demo
tags: [iOS,Objective-C]
categories: iOS
---

本节课介绍了协议，block，动画的相关知识，最后结合了这些知识点展示了一个类似**俄罗斯方块**的小游戏Demo。
总体来说本节课的内容比较重要，稍微摆脱了UI层面的知识，对于初学者来说理解起来不是很容易，不过笔者会尽量详细地讲解给大家。

# 协议
-----
关于协议所介绍的知识点比较简单，而且实现起来相对容易，故不做详细介绍，各位可以参考文档或者相关博客即可。
在这里只强调一个知识点：
### ``id obj`` 和 ``id<MyProtocol>obj``的相同点和不同点:
**相同点**：都表示了某个对象。
**不同点**：
``id obj``表示``obj``是具体某一类的实例对象。
``id<MyProtocol>obj``只表示遵守了某协议的对象 。
>因为有的时候我们并不需要确保某个对象一定是某个类的实例对象，而只需要它遵循了某个协议，这个时候就需要用第二行的写法来确保这个对象确实遵循了<MyProtocol>。


<!-- more -->


# Block
-----
关于block的概念和语法在这里就不赘述了，因为有文档和很多牛人已经总结地很好了。
在这里只强调两点关于block的使用注意事项。

## 修改block内部变量的方案
如果我们要在block里将``found``值设为YES,就应该在block外部添加``__block``关键字。
```
    __block BOOL found = NO;
    //通过__block关键字，将found从栈中移动到堆中保证其可以被修改；block结束后，将该变量复制一份到堆中，再放回栈上

    [dict enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop){        

        if ([targetString isEqualToString:obj]) {            

            *stop = YES; //停止
            found = YES;
        }        
    }];
```

## 存储循环的解决方案
只要block存在，block内部消息中的每个对象都会被block的一个强指针指着。此时，如果这些对象里的某个或几个对象也有指向该block的指针，就会造成存储循环。

问题重现：

```

    //这个block有强指针指向self，而self也通过myBlocks数组有强指针指向block

    [self.myBlocks addObject:^{    

        [self doSomething];

    }];

```

解决方案：创建弱类型的局部变量

```

    __weak ViewController *weakSelf = self; //创建弱类型的局部变量

    [self.myBlocks addObject:^{    

        [weakSelf doSomething];

    }];

```

## Block的应用

block可以直接保存在变量中，属性中，字典和数组中。

具体使用环境：

- 多线程：用于主线程，子线程的回调。
- 枚举：数组，字典的枚举等。
- 通知：某件事情发生后，信息的传递。
- 错误时调用：“包住”错误发生后需要执行的代码。
- 成功时调用：“包住”任务成功后需要执行的代码。
- 动画
- 排序

# 通过View改变视图的属性来实现动画
-----
- 改变``frame``
- 改变``transform``
- 改变``alpha``

具体通过UIView的类方法来改变

```
+ (void)animateWithDuration:(NSTimeInterval)duration   //动画在这个屏幕上出现的时间
                                     delay:(NSTimeInterval)delay       //等待多长时间再执行
                                  options:(UIViewAnimationOptions)options 
                             animations:(void (^)(void))animations  //在此代码块中修改frame，transform 和 alpha
                             completion:(void (^ __nullable)(BOOL finished))completion;
```
options参数：
```
    UIViewAnimationOptionLayoutSubviews            = 1 <<  0,
    UIViewAnimationOptionAllowUserInteraction      = 1 <<  1, // turn on user interaction while animating
    UIViewAnimationOptionBeginFromCurrentState     = 1 <<  2, // start all views from current value, not initial value
    UIViewAnimationOptionRepeat                    = 1 <<  3, // repeat animation indefinitely
    UIViewAnimationOptionAutoreverse               = 1 <<  4, // if repeat, run animation back and forth
    UIViewAnimationOptionOverrideInheritedDuration = 1 <<  5, // ignore nested duration
    UIViewAnimationOptionOverrideInheritedCurve    = 1 <<  6, // ignore nested curve
    UIViewAnimationOptionAllowAnimatedContent      = 1 <<  7, // animate contents (applies to transitions only)
    UIViewAnimationOptionShowHideTransitionViews   = 1 <<  8, // flip to/from hidden state instead of adding/removing
    UIViewAnimationOptionOverrideInheritedOptions  = 1 <<  9, // do not inherit any options or animation type

```

# 通过给视图添加物理效果实现动画
------

添加物理效果主要需要三个元素：
1. DynamicAnimator
2. UIGravityBehavior
3. 遵守<UIDynamicItem>协议的item(大部分情况是UIView)

## DynamicAnimator：动力动画
```
UIDynamicAnimator *animator =[ [ UIDynamicAnimator alloc] initWithReferenceView:aView]; //aview是动画Views的顶级视图
```
动力动画的初始化需要给其添加要进行动画的顶级视图，详细内容后面再介绍。

## UIDynamicBehavior：动力行为

动力行为分为重力动力行为，碰撞行为等具体的行为。
这个类有很多子类：

### 1. UIGravityBehavior：重力行为
```
@property (readwrite, nonatomic) CGFloat angle;//重力方向
@property (readwrite, nonatomic) CGFloat magnitude; //重力加速度值
```

### 2. UICollisionBehavior：碰撞行为
```
@property (nonatomic, readwrite) UICollisionBehaviorMode collisionMode;//互相碰撞弹开还是只是从边界碰撞弹开

@property (nonatomic, readwrite) BOOL translatesReferenceBoundsIntoBoundary; //是否是有弹性的边界
```

### 3. UIAttachmentBehavior ：吸附行为

```
@property (readwrite, nonatomic) CGPoint anchorPoint; //设置锚点
- (instancetype)initWithItem:(id <UIDynamicItem>)item attachedToAnchor:(CGPoint)point;//将动力项吸附在锚点上
- (instancetype)initWithItem:(id <UIDynamicItem>)item1 attachedToItem:(id <UIDynamicItem>)item2;//吸附两个动力项

```

### 4. UISnapBehavior：速甩行为

```
- (instancetype)initWithItem:(id <UIDynamicItem>)item snapToPoint:(CGPoint)point NS_DESIGNATED_INITIALIZER;
```

### 5. UIPushBehavior：推动行为

```
@property (nonatomic, readonly) UIPushBehaviorMode mode;
@property (readwrite, nonatomic) CGFloat magnitude;//推力
@property (readwrite, nonatomic) CGVector pushDirection;//推动方向
```
### 6. UIDynamicItemBehavior：动力项行为

```
@property (readwrite, nonatomic) CGFloat elasticity; // Usually between 0 (inelastic) and 1 (collide elastically) 

@property (readwrite, nonatomic) CGFloat friction; // 0 being no friction between objects slide along each other

@property (readwrite, nonatomic) CGFloat density; // 1 by default

@property (readwrite, nonatomic) CGFloat resistance; // 0: no velocity damping

- (CGPoint)linearVelocityForItem:(id <UIDynamicItem>)item;//线速度
- (CGFloat)angularVelocityForItem:(id <UIDynamicItem>)item;//角速度

```

## 遵守<UIDynamicItem>协议的item(大部分情况是UIView)

只要是遵守了<UIDynamicItem>协议（动力项协议）的对象，都可以添加动力行为。
```
id<UIDynamicItem>item1 = ....;
id<UIDynamicItem>item2 = ....;
[gravity addItem:itme2];
```

动力项协议的属性：
```
@property (nonatomic, readwrite) CGPoint center;//动力项的中心

@property (nonatomic, readonly) CGRect bounds; //动力项的绘制区域，只读，通过变换，居中，移动进行修改

@property (nonatomic, readwrite) CGAffineTransform transform;//动力项的旋转或缩放比例
```

若想与animator的动画相抗争，需要调用animator的以下方法：

```
- (void)updateItemUsingCurrentState:(id <UIDynamicItem>)item;
```

# Demo
---------

## Demo需求

- 点击屏幕后，在顶部随机位置生成具有随机色的正方形，正方形显示后立即下落并停止。
- 方块排满的行会自动被炸飞，而且带动画。

## Demo效果图

![左：炸飞前 | 右：炸飞后](http://upload-images.jianshu.io/upload_images/859001-6116d4bb61ba202a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要代码段

因为每个方块的动作行为都是一致的，所以在这里自定义了一个``UIDynamicBehavior``类，给每个方块增加相同的动作行为。

**1. 自定义统一行为类：DropItBehavior**
```
- (instancetype)init
{
    self = [super init];    
   //重写初始化方法，同时增加重力和碰撞行为
    [self addChildBehavior:self.gravity];
    [self addChildBehavior:self.collider];
    return self;
}

//同时增加重力和碰撞行为
- (void)addItem:(id<UIDynamicItem>)item
{
    [self.gravity addItem:item];
    [self.collider addItem:item];
}

//同时移除重力和碰撞行为

- (void)removeItem:(id<UIDynamicItem>)item
{
    [self.gravity removeItem:item];
    [self.collider removeItem:item];
}

- (UIGravityBehavior *)gravity
{

    if (!_gravity) {

        _gravity = [[UIGravityBehavior alloc] init];
         //设置重力加速度
        _gravity.magnitude = 1.9;
    }
    return _gravity;
}

- (UICollisionBehavior *)collider
{
    if (!_collider) {
        _collider = [[UICollisionBehavior alloc] init];
        //触碰边缘弹性 
        _collider.translatesReferenceBoundsIntoBoundary = YES;
    }
    return _collider;
}
```

**2. 初始化animator**
```
- (UIDynamicAnimator *)animator
{
    if (!_animator) {
        //self.gameView 是动画实现的顶级视图，它的子视图是掉落的方块
        _animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.gameView];
    }
    return _animator;
}
```
**3. 给``UIDynamicAnimator``添加行为**
```

- (DropItBehavior *)dropitBehavior
{
    if (!_dropitBehavior) {
         _dropitBehavior = [[DropItBehavior alloc] init];
        [self.animator addBehavior:_dropitBehavior];
    }
    return _dropitBehavior;
}
```

**4. 生成随机方块并让其下落**

```
/**
 *  生成随机方块并下落
 */
- (void)drop
{
    //1. 随机位置

    CGRect frame;
    frame.origin = CGPointZero;
    frame.size = DROP_SIZE;
    int x = (arc4random()%(int)self.gameView.bounds.size.width)/DROP_SIZE.width;
    frame.origin.x = x * DROP_SIZE.width;
    UIView *dropView = [[UIView alloc] initWithFrame:frame];
  
    //2. 随机颜色
    dropView.backgroundColor = [self randomColor];
    [self.gameView addSubview:dropView];
    
    //3. 添加下落
    [self.dropitBehavior addItem:dropView];
}
```
>目前小方块下落碰到障碍物后会旋转，所以容易让这些小方块散落成堆。这样一来，就不能计算好整行的排列情况，所以我们应该让小方块们没有旋转的特性。

**5.取消旋转特性**

在公用的behavior类``DropItBehavior``里增加一个``UIDynamicItemBehavior``实例，取消其旋转特性。

```
- (UIDynamicItemBehavior *)animationOptions
{
    if (!_animationOptions) {

        _animationOptions = [[UIDynamicItemBehavior alloc] init];
        _animationOptions.allowsRotation = NO;        
    }
    return _animationOptions;
}
```
这样就能整齐排列小方块了：

![左：可旋转 | 右：不可旋转](http://upload-images.jianshu.io/upload_images/859001-a6bc68132ab0425a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**6. 动画炸掉排满的行**

最好在方块都静止了之后再判断是否有排满的行，这里需要遵守协议``<UIDynamicAnimatorDelegate>``

```

/**
 *  监听动力动画内部的所有动画停止后调用炸飞整行的方法
 *
 *  @param animator 动力动画
 */
- (void)dynamicAnimatorDidPause:(UIDynamicAnimator *)animator

{
    [self removeCompleteRows];
}

```

下面来看一下炸飞整行的方法：

```

/**
 *  炸飞整行的方法：包括查看是否存在整行的算法和炸飞整行的动画
 */
- (void)removeCompleteRows
{
    NSMutableArray *dropsToRemove = [[NSMutableArray alloc] init];
    
    //遍历每一行
    for (CGFloat y = self.gameView.bounds.size.height - DROP_SIZE.height/2;y > 0;y-= DROP_SIZE.height) {
        
        BOOL rowIsComplete = YES;
        NSMutableArray *dropsFound = [[NSMutableArray alloc] init];

        for (CGFloat x = DROP_SIZE.width/2; x < self.gameView.bounds.size.width - DROP_SIZE.width/2; x+=DROP_SIZE.width) {
            
            //移动(x,y)获取这个点所在的view
            UIView *hitView = [self.gameView hitTest:CGPointMake(x, y) withEvent:NULL];

            if ([hitView superview] == self.gameView) {
               
                //如果获取的view的父视图是gameView,就说明它是方块
                [dropsFound addObject:hitView];
                
            }else{

                //否则这个行肯定是不完整的
                rowIsComplete = NO;
                break;
            }
        }

        if (![dropsFound count]) break;
        if (rowIsComplete)[dropsToRemove addObjectsFromArray:dropsFound];
  
    }    

    //如果有排满的行，则炸掉它
    if ([dropsToRemove count]){
        for (UIView *drop in dropsToRemove){
            [self.dropitBehavior removeItem:drop];
        }
        [self animatedRemovingDrops:dropsToRemove];
    }
}

/**
 *  炸飞整行
 *
 *  @param dropsToRemove 需要炸飞的View的数组
 */

- (void)animatedRemovingDrops:(NSArray *)dropsToRemove
{
    [UIView animateWithDuration:0.5 animations:^{
        
        for (UIView *drop in dropsToRemove) {
           
            //设定炸飞后终点的位置
            int x = (arc4random()%(int)(self.gameView.bounds.size.width*5)) - (int)self.gameView.bounds.size.width*2;
            int y = self.gameView.bounds.size.height;
            drop.center = CGPointMake(x,-y);

        }

    } completion:^(BOOL finished) {

        [dropsToRemove makeObjectsPerformSelector:@selector(removeFromSuperview)];

    }];
}
```


# 思考一下
----
关于通过给view添加物理效果的方法添加动画，需要弄清楚``DynamicAnimator``,``UIDynamicBehavior``和遵守<UIDynamicItem>协议的item三者之间的关系。

通过对代码的分析以及讲师的讲解，笔者将这三者以比喻的方法将他们的关系梳理了一下：

- ``DynamicAnimator``:代表了一个游乐场。
- ``UIDynamicBehavior``：代表了游乐场里的娱乐设施。
- 遵守<UIDynamicItem>协议的item：代表了去游乐场玩儿的小孩。

我们从代码看一下如何映射他们的关系：

#### DynamicAnimator
``
UIDynamicAnimator *animator =[ [ UIDynamicAnimator alloc] initWithReferenceView:aView]; 
``
在这里，``aView``代表了一片空地，这句话的意思是我们把游乐场建在了这片空地上。

#### UIDynamicBehavior
``
 [self.animator addBehavior:_dropitBehavior];
``
在这里，代表了我们在这个游乐场里增加了某个娱乐设施。

#### 遵守<UIDynamicItem>协议的item
```
- (void)addItem:(id<UIDynamicItem>)item
{
    [self.gravity addItem:item];
    [self.collider addItem:item];
}
```
在这里，代表了我们让某个小孩来玩儿某个娱乐设施。

这样就理清了：我们要让一个小孩玩儿一个娱乐设施就应该:
1. 找一片空地建设游乐场。
2. 在游乐场引进娱乐设备。
3. 孩子来玩儿这个娱乐设备。

笔者在开始看到这三者的相关代码的时候略懵逼，不知道为什么会这么设计，但是用了“比喻法”之后，顿时豁然开朗了~


# 最后的话
-----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，在评论里留言即可。
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



