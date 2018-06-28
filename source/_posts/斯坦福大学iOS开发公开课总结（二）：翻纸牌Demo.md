---
title: 斯坦福大学iOS开发公开课总结（二）：翻纸牌Demo
tags: [iOS,Objective-C]
categories: iOS
---


本节课简单介绍了控件的懒加载(lazy instantiation)，数组，字典，类方法的使用，在最后展示了翻转卡牌的小demo。



## 懒加载(lazy instantiation)
---
**懒加载**：在实例变量被使用之前的那一刻初始化。防止大量的实例变量(属性)在同一时间初始化(尤其是不用将创建对象的方法全部写在``viewDidLoad:``方法里)。


```
@interface ViewController ()

@property (nonatomic, strong) NSMutableArray *cards;

@end

@implementation ViewController

- (NSMutableArray *)cards
{
    //如果此实例变量为空，则初始化；否则，直接调用
    if (!_cards) {
        _cards = [[NSMutableArray alloc] init];
    }
    return _cards;
}

@end
```

<!-- more -->

## 数组的使用
---
#### 在可变数组中插入元素
在数组中插入元素是可变数组的方法，因为不可变数组在初始化以后就无法再更改。
尤其注意的是：在数组中插入元素时，插入的元素必须不能为空，否则会引起程序崩溃。需要对要插入的元素做是否为空的判断！

```
- (void)addCard: (Card *)card atTop:(BOOL)atTop
{
    if (atTop) {
        
        //插入到数组第一个位置
        [self.cards insertObject:card atIndex:0];
    
    }else{
        
        //添加到数组末尾
        [self.cards addObject:card];
    
    }
}

```

#### 在可变数组中引用和移除数组元素

在数组中，提取和移除元素的时候需要注意的是需要判断数组是否为空，如果**在数组为空的情况下引用或移除某个元素会引起程序的崩溃！** 所以要先进行目标数组元素数量判断。

```
- (Card *)drawRandomCard
{
    //如果数组为空数组，则直接放回nil，因为没有可以抽取的元素。
    Card *randomCard = nil;
    
    //判断数组元素个数是否不为0，如果为0，则返回nil
    if ([self.cards count]) {
        
        //生成0到[self.cards count]的随机数
        NSUInteger index = arc4random() % [self.cards count];
        
        //引用下标为index的元素
        randomCard = self.cards[index];
        
        //移除下标为index的元素
        [self.cards removeObjectAtIndex:index];
        
    }
    
    return randomCard;
}

```


## 类方法
---
**类方法**也叫工厂方法，类方法主要包括两种：

1. **类的初始化**：形成类的实例。
2. **工具方法**：不经过实例化获得某些数据。

**应用**：用类方法生成四种不同花色：

```
+ (NSArray *)ValidSuits
{
    return @[@"♥︎",@"♦︎",@"♣︎",@"♠︎"];
}

- (void)setSuit:(NSString *)suit
{
    if ([ [PlayingCard ValidSuits] containsObject:suit]) {
        _suit = suit;
    }
}

```


## instancetype
---
instancetype的使用目的是确保返回的对象同这条消息要发送到的对象一样。

怎么说？

常用在类的初始化方法的返回值中，因为如果类的初始化返回值是``id``,那么这个指针可以指向任何对象，所以有可能指向非此类的对象类型；

所以，``instancetype``作为初始化方法的返回值后，那么初始化的结果一定会同此类的类型一致。

举个🌰：


```
- (instancetype)init
{
    //检查父类是否初始化成功
    self = [super init];
    
    if (self) {
        //初始化代码
    }
    
    return self;
}

```





## 翻牌Demo
---
#### 设计需求：
在界面上显示一张扑克牌，点击后翻牌：如果是正面，点击后就显示背面；如果是背面，点击后显示正面。

#### 效果图：



![左：纸牌背面；右：纸牌正面](http://upload-images.jianshu.io/upload_images/859001-191d3037dd166754.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




#### 实现代码：

**1. 点击按钮翻牌**
```
- (IBAction)touchCardButton:(UIButton *)sender {
    
    //先判断牌面：如果按钮的title有字，则为正面
    if ([sender.currentTitle length]) {
       
        //1. 背面的UI效果:
        
        //1.1 设置背景图片为背面的图片
        [sender setBackgroundImage:[UIImage imageNamed:@"cardBack"] forState:UIControlStateNormal];
       
        //1.2 设置按钮title为空，因为要翻到背面
        [sender setTitle:@"" forState:UIControlStateNormal];
        
    }else{
        
        //2. 正面的UI效果:
      
        //2.1 设置背景图片为空
        [sender setBackgroundImage:[UIImage imageNamed:@""] forState:UIControlStateNormal];
        
        //2.2 设置背景颜色为白色
        [sender setBackgroundColor:[UIColor whiteColor]];
       
        //2.3 设置按钮title为牌的花色
        [sender setTitle:@"A♣︎" forState:UIControlStateNormal];
    }
    
    //翻牌次数记录。同时存在setter和getter方法。首先getter方法取到当前的翻拍次数，然后用setter方法让翻拍次数+1
    self.flipCount++;
}

```


**2. 更新按钮UI**

更新按钮的UI的代码放到了属性``flipCount``的setter方法里。


```
- (void)setFlipCount:(int)flipCount
{
    _flipCount = flipCount;
    
    //在setter方法完成后，设置标签的显示数字
    self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d", self.flipCount];
}

```

但是笔者个人认为这种处理方式并不好，因为**一个方法最好只做一件事情**，所以应该将更新按钮UI的代码单独提取出来作为另一个方法:


```
- (IBAction)touchCardButton:(UIButton *)sender {
    
    //先判断牌面：如果按钮的title有字，则为正面
    if ([sender.currentTitle length]) {
       
        //背面的UI效果
        [sender setBackgroundImage:[UIImage imageNamed:@"cardBack"] forState:UIControlStateNormal];
        [sender setTitle:@"" forState:UIControlStateNormal];
        
    }else{
        
        //正面的UI效果
        [sender setBackgroundImage:[UIImage imageNamed:@""] forState:UIControlStateNormal];
        [sender setBackgroundColor:[UIColor whiteColor]];
        [sender setTitle:@"A♣︎" forState:UIControlStateNormal];
    }
    
    //翻牌次数+1。
    self.flipCount++;
    
    //更新Label显示的数字
    [self updateFlipsLabel];
}

/**
 *  单独提取出更新Label显示的数字的方法
 */
- (void)updateFlipsLabel
{
     self.flipsLabel.text = [NSString stringWithFormat:@"Flips: %d", self.flipCount];
}

```


这样一来，程序的可读性更高了一点。而且我们还不用重写``flipsCount``的setter方法。


