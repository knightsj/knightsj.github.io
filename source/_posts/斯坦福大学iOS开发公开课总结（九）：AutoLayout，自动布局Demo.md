---
title: 斯坦福大学iOS开发公开课总结（九）：AutoLayout，自动布局Demo
tags: [iOS,Objective-C]
categories: iOS
---


本节课介绍了iOS在故事版里构造AutoLayout(自动布局)的三种方法并通过沿用了第六课的[Demo](http://www.jianshu.com/p/8d5a4a8ac2be)具体演示了添加约束的过程。内容较少也比较简单，可惜的是没有讲解用纯代码构造自动布局。




**PS：严重多图预警！**

因为操作都是在故事版里进行的，所以只能通过截图来演示具体操作步骤。。。

<!-- more -->


# AutoLayout
------

## 在故事版里构造AutoLayout的三种方法：



1. 使用蓝色辅助线，并选择系统建议约束。
2. 点击底部的布局菜单，根据需求选择相应的约束。
3. 按住control按键拖动触发菜单，根据需求选择相应约束。



下面具体每种方法的做法：


## 1. 使用蓝色辅助线，并选择系统建议约束

我们现在要将“Thing 1”和“Thing 2”两个标签放在左上角和右下角。
![使用蓝色辅助线](http://upload-images.jianshu.io/upload_images/859001-1f209520ddccc48e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




>在拖动空间的时候，系统会在某些时刻给出建议的约束，用蓝色虚线表示，详情看左图。
>在约束显示出来的前提下放下控件，再选择系统建议的约束可以添加系统建议的约束，也就是之前虚线表示出来的约束，详情看右图。



## 2. 点击底部的布局菜单，根据需求选择相应的约束

我们现在要添加“Bad Thing”按钮，将其置于屏幕正中间。
![使用底部布局菜单](http://upload-images.jianshu.io/upload_images/859001-38cf0f8b7dddb55b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




>想让控件居中显示，可以根据底部的按钮弹出的菜单设置，具体看左图。

>添加约束后，生成了黄色虚线框，如中间的图所示。黄色虚线框为控件添加该约束后，控件应有的frame。这时，应该点击左上角的黄色小三角选择“update frame”，具体看右图。


最终效果图如下：


![效果图1](http://upload-images.jianshu.io/upload_images/859001-4ed297bd355f392e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




>点击黄色小三角显示的选项的意义：  

>1. update frame：通过修改frame 来适应约束。 
2. update constrains: 修改约束 适应这个控件的frame。
3. reset to suggested constrains:使用建议约束。







## 3. 按住control按键拖动触发菜单，根据需求选择相应约束。


我们现在要将“Bad Thing”和"Thing 2"垂直距离固定，右边对其。

![按住control键](http://upload-images.jianshu.io/upload_images/859001-192fcf72d0c3b71e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在拖动控件"Bad Thing"后，并不会改变其原有的约束(出现了黄色虚线框)，如左图。我们需要先删除其原有的约束。
然后点击“Bad Thing”按住``control``拖动到``Thing 2``,弹出菜单后，设置二者的垂直距离固定，右边对其，如右图所示。



最终效果图如下：

![效果图2](http://upload-images.jianshu.io/upload_images/859001-c292148931dab740.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>拖动也有三种方式：

>1. 从一个控件按住control按键到另一个控件，选择相应的排列方式。
2. 从一个控件拖拽到它的父视图：水平居中，垂直居中等。
3. 从一个控件拖拽到它自己：选择固定宽度等。









# Demo
------


首先我们拿到之前的属性字符串Demo，按照第一种设定约束的方法，结果不尽人意：


![宽度不等](http://upload-images.jianshu.io/upload_images/859001-143c8a529cb76416.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



显然，我们需要让四个彩色按钮宽度保持一致：
点击下方弹出菜单，选择“Equal Width”。

![设置等宽](http://upload-images.jianshu.io/upload_images/859001-4d27907e6dc68116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




现在确实是等宽了，但是控制台有报错信息，虽然运行木有问题。

什么问题呢？

因为我们在让四个彩色按钮宽度相等的同时**硬编码**了它们的宽度，这显然不同时适用于横屏和竖屏的情况，需要将它们的固定宽度删去：

![删除固定宽度](http://upload-images.jianshu.io/upload_images/859001-bf9616a1e7401713.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在第二个页面，我们把两个Label放到左下角：


![垂直固定](http://upload-images.jianshu.io/upload_images/859001-d8c09edac6a09686.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. 首先用下方的菜单，将靠左和靠下的距离固定，如左图。
2. 然后用拖动control键的方法设定第二个标签的左对齐和垂直距离，效果如右图。





在这里没有固定标签的宽度，这很好，因为如果数字是多位的，固定的宽度可能无法全部显示标签内的内容。

那么手动固定一下其中一个标签的宽度，通过拖动control键拖动到自己的方法点击“width”，使宽度固定：



![固定宽度.png](http://upload-images.jianshu.io/upload_images/859001-b9ab01e556babf45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




>固定宽度的标签无法完整显示了内容，因此这是一个危险的做法。


# 最后的话
----

如果哪位小伙伴想拿到本文Demo的代码请不要客气，在评论里留言即可。
而且十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~

# 2016.7.12日更新：
---

笔者已经把目前为止整理的所有Demo(第二课到第十课)放入到了我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)仓库里。分为英文注释版和中文注释版(英文注释要少一点，嘿嘿)想要的小伙伴可以果断下载~ 如果有不知道怎么下载的小伙伴请联系我~


