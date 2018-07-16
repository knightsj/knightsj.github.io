---
title: 斯坦福大学iOS开发公开课总结（七）：自定义视图，手势识别，绘制纸牌Demo
tags: [iOS,Objective-C]
categories: iOS
---

本节课介绍了UIView的一些知识，自定义UIView的方法以及手势识别。最后应用本节所讲的大部分知识点向我们演示了一个绘制纸牌的Demo。


# UIView
-----

## 关于UIView，你需要知道的零散知识
- 视图是可以多层嵌套的。
- 每个视图可以有多个子视图，但是只能有一个父视图。
- 控制器的view属性指向自己的顶级视图。
- 令视图透明会加大系统的开销。
- 通过判断控制器view的``self.view.window``是否存在来判断控制器view是否被显示出来。

<!-- more -->

## UIView的一些属性和方法
```
@property CGFloat contentScaleFactor; //返回每个点所有的像素数 ：非retina为1，retina为2

- (UIView *)superView; //指向自己的父视图
- (NSArray *)subview; //自己的所有子视图的数组

- (void)addSubview: (Uiview *)aView;// 发送给目标父视图，让其把aView作为自己的子视图
- (void)removeFromSuperview;  //消息发送给要移除的vie
```

## View的初始化方法：
```
- (void)awakeFromNib {[self setup];}  //通过故事版创建的View的初始化
- (id)initWithFrame: (CGRect)aRect    //通过纯代码创建的View的初始化
{
     self = [super initWithFrame:aRect];
     [self setup];
     return self;
}

- (void)setup {....};
```

## Custom View 自定义视图

在iOS中，自定义是图的方法是创建一个UIView的子类并重写 ``- (void)drawInRect:(Rect)rect``方法。
> 注意：永远都不要自己调用这个方法，要交给系统负责！
> 可以调用以下的方法，告诉系统这个视图要被重绘：
```
- (void)setNeedsDisplay;
- (void)setNeedsDisplayInRect: (CGRect)aRect  
```

那么具体怎样重写 ``- (void)drawInRect:(Rect)rect``方法来绘图呢？
答：应用Core Graphics的相关知识。

## Core Graphics

Core Graphics是一套基于C的API框架，使用了Quartz作为绘图引擎，使用Core Graphics，可以创建直线、路径、渐变、文字与图像等内容，并可以做变形处理。

### Core Grephics的工作步骤：
1. 取得图形上下文。
2. 设置绘图路径(利用UIBezierPath)。
3. 设置颜色。
4. 用颜色填充路径 。

各位看官不用着急，具体方法在最后的Demo代码里给大家呈现。

# UIGestureRecoginizer ：手势识别抽象类
-----

**简单介绍**：``UIGestureRecoginizer``是一个抽象类，它的各种子类可以用于识别各种不同的手势：如捏合，滑动等等。通过识别各种不同的手势，实现各种交互操作。

## 使用步骤
1. 在视图中添加手势识别对象。
2. 提供手势发生时所需要调用的方法。

## 手势种类：

**1. UIPanGestureRecognizer ： 拖动手势**
```
- (void)setPannableView:(UIView*)pannableView
{
     _pannableView = pananbleView;
     UIPanGestureRecognizer *pangr = [UIPanGestureRecognizer alloc] initWithTarget:pannableView action: @selector(pan:)];
     [pannableView addGestureRecognnizer:panr];
} 
```

**2. UIPinchGestureReccognizer ：捏合手势**
```
@property CGFloat scale;   捏合手势距离
@property (readonly) CGFloat velocity; 每分钟变化的速度
```

**3. UIRotationGestureRecgnizer 旋转手势**
```
@property CGFloat rotation;   弧度
@property (readonly) CGFloat velocity; 每秒变化的速度
```

**4. UISwipeGestureRecgnizer ： 滑动手势**
```
@property UISwipeGestureRecognizerDirection direction 滑动方向
@property NSUInteger numberOfTouchesRequired; 几只手指来完成
```

**5. UITapGestureRecognizer ：点击手势**
```
@property NSUInteger numberOfTapsReqired；几次点击
@property NSUInteger numberOfTouchesRequired;     几只手指来完成 
```

>以上第4，5项手势是非连续手势；1，2，3属于连续手势。
>注意区分滑动手势和拖动手势。滑动手势是指短促，快速地滑动的手势，而拖动手势是相对较慢，路径较长的手势。

# 绘制纸牌Demo
-----

## Demo需求
- 绘制一张纸拍放到屏幕上，包括正面和背面。
- 滑动手势可以翻牌。
- 捏合手势可以伸缩纸牌正面的图案大小。

## Demo效果图


![左二图：翻牌 | 右二图：伸缩](http://upload-images.jianshu.io/upload_images/859001-aedaf67a6c46a092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 重要代码段

#### 1. 绘制纸牌正反面

```
- (void)drawRect:(CGRect)rect {

    //初始化一个圆角矩形
    UIBezierPath *roundRect = [UIBezierPath bezierPathWithRoundedRect:self.bounds cornerRadius:[self cornerRadius]];

    //裁剪，保证不会绘制四角
    [roundRect addClip];

    //填充白色
    [[UIColor whiteColor] setFill];
     UIRectFill(self.bounds);

    //轮廓
    [[UIColor blackColor] setStroke];
    [roundRect stroke];

    if (self.faceUp) {

        //1. 纸牌正面
        //1.1 纸牌正面中间的图
        UIImage *faceImage = [UIImage imageNamed:[NSString stringWithFormat:@"%@%@",[self rankAsString],self.suit]];

        if (faceImage) {

            CGRect imageRect = CGRectInset(self.bounds, self.bounds.size.width * (1.0 - self.faceCardScaleFactor) + 20, self.bounds.size.height * ( 1.0 - self.faceCardScaleFactor  ) + 20);
            [faceImage drawInRect:imageRect];
        }
      
        //1.2 纸牌正面四个角
        [self drawCorners];

    }else{

        //2. 纸牌背面
        [[UIImage imageNamed:@"cardBack"] drawInRect:self.bounds];
    }
}
```

#### 2. 绘制纸牌边角的花色和数字

```
- (void)drawCorners
{    
    //设定段落排列
    NSMutableParagraphStyle *paragraphStype = [[NSMutableParagraphStyle alloc] init];
    paragraphStype.alignment = NSTextAlignmentCenter;

   //设定字体
    UIFont *cornerFont = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
    cornerFont = [cornerFont fontWithSize:cornerFont.pointSize * [self cornerScaleFactor]]; 

    //角落文字
    NSAttributedString *cornerText = [[NSAttributedString alloc] initWithString:[NSString stringWithFormat:@"%@\n%@", [self rankAsString], self.suit] attributes:@{NSFontAttributeName:cornerFont,NSParagraphStyleAttributeName:paragraphStype}];

    //左上角
    //1. 获得图片的rect
    CGRect textBounds;
    textBounds.origin = CGPointMake([self cornerOffset], [self cornerOffset]);
    textBounds.size = [cornerText size];

     //2.绘制文字
    [cornerText drawInRect:textBounds];

    //右下角
    //1. 获取上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    // 2. 移动上下文
    CGContextTranslateCTM(context, self.bounds.size.width, self.bounds.size.height);
     //3. 翻转上下文（翻转180度）
    CGContextRotateCTM(context, M_PI);
     //4. 绘制
    [cornerText drawInRect:textBounds];
}
```
>我们可以看到，图片和文字的绘图方法都是可以通过``drawInRect:``方法来进行：通过传入需要绘制的``rect``，可以让系统根据原始的素材（图片，文字）来绘图。

#### 3. 添加手势：连线方式
```
/**
 *  滑动手势翻转牌
 *
 *  @param sender 滑动手势
 */

- (IBAction)swipe:(id)sender {
 
   //翻转牌面
    self.playCardView.faceUp  = !self.playCardView.faceUp;

}
```

#### 4. 添加手势：代码方式

```
//1. 添加捏合手势
 [self.playCardView addGestureRecognizer:[[UIPinchGestureRecognizer alloc] initWithTarget:self.playCardView
                                                                                    action:@selector(pinch:)]];

/**
 *  2. 捏合手势调用的方法
 *
 *  @param gesture 捏合手势
 */

- (void)pinch:(UIPinchGestureRecognizer *)gesture
{
    if (gesture.state == UIGestureRecognizerStateChanged || gesture.state == UIGestureRecognizerStateEnded) {
       //根据捏合的程度来伸缩图片
        self.faceCardScaleFactor *= gesture.scale;
        gesture.scale = 1.0;

    }
}
```

>在手势识别调用的方法里，我们需要对手势本身的状态加以判断以确保各种交互的实现都是正确的。

# 最后的话
---
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



