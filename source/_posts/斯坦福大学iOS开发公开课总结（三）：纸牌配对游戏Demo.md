---
title: 斯坦福大学iOS开发公开课总结（三）：纸牌配对游戏Demo
tags: [iOS,Objective-C]
categories: iOS
---


本节课知识点内容不多，主要是延续了上一节课翻单张纸牌的游戏(详情请见：[斯坦福大学iOS开发公开课总结（二） ：翻纸牌Demo](http://www.jianshu.com/p/332324bff10a)），将一张纸牌扩展到一个多张纸牌并进行配对和打分的小游戏。

本节课的内容虽然简单，但是十分重要，讲师强调了MVC的设计原则并实际运用到了代码中，本文就Demo的具体代码来讲解本节课提到的知识点。

<!-- more -->

# 设计需求
----

- 显示多张纸牌，点击任意一张牌可以翻牌。
- 两张牌都显示正面后可以进行配对：
  - 花色匹配得1分；数字匹配得4分，匹配后，两张牌切换为不可点击状态（置灰）。
  - 都不匹配扣2分。
  - 每次翻牌都减一分。
- 每次翻牌都要更新分数。

# 效果图：
---
![左：初始界面 ；右：游戏中界面](http://upload-images.jianshu.io/upload_images/859001-6371568b2e366281.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 重要代码段与知识点
---

## 模型类：CardMatchingGame

#### 1. 在公共接口设置只读属性

```

//CardMatchingGame.h
@property (nonatomic, readonly) NSInteger score;

//CardMatchingGame.m
@property (nonatomic, readwrite) NSInteger score;

```

>在.h文件中将分数属性设置为只读，并在.m文件中设定该属性为读写，以便在内部计算。

>原因：不希望其他类更改此属性，只能获取该属性。通俗一点地说：“你们就拿我给你算好的分数就好了，你们是不能更改它的！”

#### 2. 指定初始化器：Designated initializer

```

- (instancetype)initWithCardCount:(NSUInteger)count usingDeck:(Deck *)deck
{
    self = [super init];
    
    if (self) {
        
        for (int i = 0; i < count; i++) {
            
            Card *card = [deck drawRandomCard];
            
            if (card) {
               
                [self.cards addObject:card];
                
            }else{
                
                self = nil;
                break;
            }
        }
    }
    return self;
}

```

>有些时候，我们需要在类实例化的时候就要求对象持有某些数据,这就需要设计**指定初始化器**，因为原始的初始化方法``-(instancetype)init``方法是无法让实例对象持有非零数据的(初始化后，基本数据类型属性=0；对象属性=nil)。

>在这段代码里，该模型类通过数量``count``和一堆纸牌``deck``中拿到了自己持有的数组``self.cards``。

>举个🌰 ：想要从一个有52张牌的堆里抽取了12张牌来作为自己的一堆纸牌的话，就要设置Deck为具有52张牌的数组；而设置count为12即可。


#### 3. 设定常量

```
#define MISMATCH_PENALTY 2 //简单的替换，不具有数据类型
```

```
static const int MISMATCH_PENALTY = 2; //非简单替换，具有数据类型
```


## 控制器类：CardMathcingGameViewController
---
#### 1. 接收来自View的点击事件并更新UI

```

/**
 *  接收用户的点击事件
 *
 *  @param sender 点击的按钮对象
 */
- (IBAction)touchCardButton:(UIButton *)sender {

    //1. 找到界面中所点击的按钮index
    NSInteger cardIndex = [self.cardButtons indexOfObject:sender];
    
    //2. 找到模型中相同index的纸牌数据，并判断是否匹配，计算分数
    [self.game chooseCardAtIndex: cardIndex];
    
    //3. 更新UI
    [self updateUI];
}

/**
 *  更新UI
 */
- (void)updateUI
{
    //1. 更新view上所有牌面
    for (UIButton *cardButton in self.cardButtons) {
        
        //1.1 找到界面中的一张纸牌(按照枚举的顺序)
        NSInteger cardIndex = [self.cardButtons indexOfObject:cardButton];
        
        //1.2 找到模型中对应的纸牌数据
        Card *card  = [self.game cardAtIndex:cardIndex];
        
        //1.3 根据数据更新纸牌的UI和可点击性
        [cardButton setTitle:[self titleForCard:card] forState:UIControlStateNormal];
        [cardButton setBackgroundImage: [self backgroundImageForCard:card] forState:UIControlStateNormal];
         cardButton.enabled = !card.isMatched;        
    }
    
    //2. 更新分数
    self.scoreLabel.text = [NSString stringWithFormat:@"Score: %ld", (long)self.game.score];
}
```

>在这里，我们可以很容易看到MVC的工作流程：
1. 在View里发生了点击事件并通知给了Controller。
2. Controller告诉Model发生了点击。
3. Model根据点击事件更新自己，然后将更新后的自己告诉Controller。
4. Controller根据更新后的模型去更新UI。

这里笔者有一张自己画的图，略逗逼，掩面贴出，独乐乐不如众乐乐~


![MVC流程图.png](http://upload-images.jianshu.io/upload_images/859001-0c8b70fd62292aa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 零散知识点
---

#### 1. 在数组里传入其包含的对象返回其所在序号

```
NSInteger cardIndex = [self.cardButtons indexOfObject:cardButton];
```

#### 2. 在数组中找到第一个元素

```
//应该使用：
PlayingCard *otherCard = [otherCards firstObject];

//不应该使用：
PlayingCard *otherCard = otherCards[0];

//不应该使用：
PlayingCard *otherCard = [otherCards objectAtIndex:0];
```

>应该使用第一种情况。
>因为如果数组为空，那么如果使用第一种情况会返回nil,而向nil发送消息是不会造成崩溃的。
>但是如果使用后两种方法，一旦数组为空，就会立刻造成程序崩溃！而且同样适用与取数组的最后一个元素的情况。
>在数组中找到最后一个元素：
```
PlayingCard *otherCard = [otherCards lastObject];
```



#### 3. 在数组中是否包含某元素

```
if ([ [PlayingCard ValidSuits] containsObject:suit]) {
        _suit = suit;
}
```

>containsObject:是NSArray的方法，返回布尔值，用来判断是否包含某个元素。

#### 4. JPG格式图片的读取

```
- (UIImage *)backgroundImageForCard: (Card *)card
{
    //默认是png，如果是jpg需要加上.jpg的后缀
    return [UIImage imageNamed:card.isChosen? @"cardFront.jpg":@"CardBack.png"];
}
```

>如果使用``imageNamed:``方法，仅传入jpg格式的文件名是无法显示出图片的，应该讲后缀``.jpg``拼接后传入才可以哦~而相同情况下，若要显示``png``格式的图片的话是不需要另外加后缀的。

# 最后的话：
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



