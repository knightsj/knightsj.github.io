---
title: 斯坦福大学iOS开发公开课总结（六）：多态，导航控制器和选项卡栏控制器Demo
tags: [iOS,Objective-C]
categories: iOS
---


本节课的课程地址：[控制器多态性、导航控制器、选项卡栏控制器](http://open.163.com/movie/2014/1/F/D/M9H7S9F1H_M9H80B3FD.html)

本节课通过延伸第四节课的纸牌配对Demo（详情请见：[斯坦福大学iOS开发公开课总结（三） ：纸牌配对游戏Demo](http://www.jianshu.com/p/2e9c8af048d8)）讲解了控制器的多态性，扩展了第五节课的属性字符串Demo(详情请见:[斯坦福大学iOS开发公开课总结（四） ：属性字符串Demo](http://www.jianshu.com/p/c4e277bbad71))讲解了如何使用导航控制器和选项卡栏控制器。

建议读者先了解以上两个博客的内容，因为这样有助于对本节课笔记的理解。

<!-- more -->

# 多态
----

第四节课做的纸牌配对游戏只有一个ViewController，它导入了``PlayingCardDek.h``类，说明该ViewController只适用于纸牌游戏。如果我们想换一类牌，那么显然目前使用的ViewController是不适用的。

具体看一下代码：

```
#import "ViewController.h"
#import "PlayingCardDeck.h"

@implementation ViewController

- (CardMatchingGame *)game
{
    if (!_game) {
        _game = [[CardMatchingGame alloc] initWithCardCount:[self.cardButtons count] usingDeck:[self createDeck]];
    }
    return _game;
}


- (Deck *)createDeck
{
    //实例化了 PlayingCardDeck 类
    return [[PlayingCardDeck alloc] init];
}

```

>为了将该ViewController作为通用的控制器，我们需要将它设置为抽象类。通过创造不同的继承它的字类来实现各种各样的纸牌配对游戏。

## 抽象类
**抽象类**就是不能被实例化的类，它具有某种普遍性，可以通过继承它来实现基于这种普遍性并带有其他特性的字类。

举个🌰：船是一个具有普遍性的抽象类。基于这个抽象来，如果我们添加了木桨，就可以造一个木桨船；如果我们给它添加了帆，就可以早一个帆船等等。。

因此，在这里，我们需要将纸牌游戏的牌堆抽象出来，如果这个牌堆里是扑克牌，那么这个游戏就是扑克牌配对游戏；如果这个牌堆里的牌是塔罗牌，那么这个牌就是卡洛牌配对游戏。

## 如何创建抽象类的具体子类？

#### 1. 将该类抽象化：

从上面的代码可以看到，控制器的实例化是基于``createDeck``方法的(这个方法将``扑克牌堆``的模型交给了控制器), 如果可以阻止模型的生成就阻止控制器的实例化。因此我们将``createDeck``的方法的返回值设为nil：

```
#import "ViewController.h"
#import "PlayingCardDeck.h"

@implementation ViewController

- (CardMatchingGame *)game
{
    if (!_game) {
        _game = [[CardMatchingGame alloc] initWithCardCount:[self.cardButtons count] usingDeck:[self createDeck]];
    }    
    return _game;
}

- (Deck *)createDeck
{
    return nil;
}

```
>这样一来，该类无法取得自己的模型实例，就无法正常工作，变得“抽象”。

#### 2. 将需要字类实现的方法放在抽象类的公共API中：

```
#import <UIKit/UIKit.h>
#import "Deck.h"

@interface ViewController : UIViewController

/**
 *  abstract metod, for subclasses
 *
 *  @return 各种类型不同的纸牌堆
 */
- (Deck *)createDeck;

@end

```

>不难想到，子类将该方法实现的过程就是实例化具有不同特性的具体类的过程！这也就实现了类的多态。

#### 3. 创造继承抽象的子类

新建一个继承于抽象类``ViewController``的字类``PlayingCardViewController``：

**PlayingCardViewController.h**

```
#import "ViewController.h"

@interface PlayingCardViewController : ViewController

@end

```


**PlayingCardViewController.m**

```
#import "PlayingCardViewController.h"
#import "PlayingCardDeck.h"

@interface PlayingCardViewController ()

@end

@implementation PlayingCardViewController

//只需要实现
- (Deck *)createDeck
{
    return [[PlayingCardDeck alloc] init];
}

@end

```

>这样一来，我们就获得了一个扑克牌配对游戏的ViewController。
>将来，如果我们还有别的继承与``Deck``的牌，就可以用相同的方法：通过创建继承该抽象类并实现``createDeck``的方法来完成。


# 多MVC的实现：通过导航控制器管理多个ViewController
----

**导航控制器**拥有一个**栈数据结构**，它可以将多个控制器压入自己的栈结构中来管理这些控制器。我们经常看到的界面**滑入滑出**的过程就是导航控制器的栈结构在**压入弹出**控制器的过程。而每个控制器管理一个MVC模型，导航控制器通过管理这些控制器实现了管理多MVC的目的。

**需要注意的是**：手机的屏幕每次只能显示一个MVC模型的View。在view的切换过程中，手机界面显示的是**当前处于导航控制器栈顶的控制器的视图！**当该视图被移除界面的时候，该MVC的数据就会被释放。因此，每次要显示一个新的MVC的时候，都会创建一个新的MVC。

## 控制器在导航控制器的栈结构中弹出
```
- (void)popViewController
{
    //self是当前的控制器，它的navigationController属性指向管理自己的导航控制器
   [self.navigationController popViewControllerAnimitaed:YES];

}
```


## 跳转到下一个控制器之前执行的方法：
```
- (void)prepareForSegue:(UIStroyboardSegue *)segue sender: (id)sender
{

 //segue的identifier属性用来区分不同的控制器
  if ([segue.identifier isEqualToString:@"DoSomething"])
  { 
      //segue的destinationViewController指向的是下一个要滑入界面的控制器
      if ([segue.destinationViewController isKindOfClass:[DoSomethingVC class]]){
      
       DoSomethingVC *doVC = (DoSomethingVC *)segue.desitinationViewController；
       doVC.infoString = self.infoString;
      }
      
    }

}
```


## 判断是否可以跳转的方法

```
- (BOOL)shouldPerformSegueWithIdentifier: (NSString *)identifier sender: (id)sender
{

  if([segue.identifier isEqualToString:@"DoAParticularThing"])
  {
     //此方法是用来确定跳转的可行性，因为有时如果缺少下一个界面的数据是不能跳转的
    return [self canDoAParticularThing]? YES:NO;
  }

}

```


# 多MVC Demo

## DEMO需求：

- 在第一个页面可以设置字符属性
- 点击第一个页面导航栏右侧的按钮跳转到第二个页面
- 在第二个页面统计第一个页面中添加色彩和边框的字符数量

## Demo效果图

![|导航控制器|  左：第一个页面；右：第二个页面](http://upload-images.jianshu.io/upload_images/859001-35ed7fcc54dc0c69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![|选项卡栏控制器| 左：第一个Tab；右：第二个Tab](http://upload-images.jianshu.io/upload_images/859001-5788b1865e19247c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要代码与知识点

#### 跳转页面传值

实现跳转页面传值一共有两个步骤：

1.  我们先通过在第二个页面的公共API中设置属性
2.  然后在第一个页面跳转到第二个页面之前将数据赋予第二个页面的这个公共属性实现传值。

**1. 在第二个页面设置公共属性**：

```
#import "ViewController.h"

@interface TextAnylizeViewController : UIViewController

@property (nonatomic, retain) NSAttributedString *textToAnalyze;

@end


```

**2. 在第一个页面跳转到第二个页面之前执行传值**

```
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    //1. 首先判断segue的identifier
    if ([segue.identifier isEqualToString:@"Analyze Text"]) {
         //2. 然后判断目标控制器的类型
        if ([segue.destinationViewController isKindOfClass:[TextAnylizeViewController class]]) {
            //3. 在1和2都确定的情况下，实例化第二个页面
            TextAnylizeViewController *analyzeVC = (TextAnylizeViewController*)segue.destinationViewController;
            //4. 将第一个页面的字符串赋予第二个页面，用于第二个页面的分析
            analyzeVC.textToAnalyze = self.body.textStorage;
        }
    }
}

```

#### 获取一段字符中，具有某种属性的字符串

```
- (NSAttributedString *)charactersWithAttribute: (NSString *)attributedName
{
    NSMutableAttributedString *characters = [[NSMutableAttributedString alloc] init];
    
    NSUInteger index = 0;
    
    while (index < [self.textToAnalyze length]) {
        
        NSRange range;
        
        //查找一段字符串中，具有某种相同属性的值
        id value = [self.textToAnalyze attribute:attributedName atIndex:index effectiveRange:&range];
       
        if (value) {
            //如果值存在，获取具有该相同属性的字符串
            [characters appendAttributedString:[self.textToAnalyze attributedSubstringFromRange:range]];
            
            //将index移动到具有该相同属性的字符串的下一位
            index = range.location + range.length;
        
        }else{
            index ++;
        }
    }
    
    return characters;
}


```

#### 搭建导航控制器(Navigation Controller)

![导航控制器](http://upload-images.jianshu.io/upload_images/859001-7a518741341460be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 搭建选项卡栏控制器(Tab Bar Controller)

![选项卡栏控制器](http://upload-images.jianshu.io/upload_images/859001-c477e12dc9779726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 最后的话
---
如果哪位小伙伴想拿到本文Demo的代码请不要客气，在评论里留言即可。
而且十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~

# 2016.7.12日更新：
---

笔者已经把目前为止整理的所有Demo(第二课到第十课)放入到了我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)仓库里。分为英文注释版和中文注释版(英文注释要少一点，嘿嘿)想要的小伙伴可以果断下载~ 如果有不知道怎么下载的小伙伴请联系我~


