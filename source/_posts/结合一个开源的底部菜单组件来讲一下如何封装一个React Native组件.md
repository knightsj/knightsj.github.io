---
title: 结合一个开源的底部菜单组件来讲一下如何封装一个React Native组件
tags: [React Native]
categories: Production
---



## 简介

前几天写了一个React Native组件：一个可定制性比较高的底部弹出菜单（ActionSheet）。该组件符合React Native的特性：同时支持iOS和Android双平台，一份相同的代码会在两个平台上展示几乎完全相同的样式。


先看一下效果(上排为iOS模拟器，下排为Android模拟器)：

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_show.png)



上图展示的是该组件的默认样式。由于该组件具有较高的定制性，所以只需要通过设置一些属性就可以得到更多不同的样式。



开源项目地址：[GitHub:react-naive-highly-customizable-action-sheet](https://github.com/knightsj/react-naive-highly-customizable-action-sheet)

<!-- more -->

## 定制性介绍


在该组件里：最顶部的标题，中间的选择项，最底部的取消项都是可有可无的，而且每一部分的字体，颜色，高度，距离，分割线颜色，圆角等也都是可以定制的。


先来看几个默认的样式：


### 默认的样式：

默认的样式是指使用者在不设置样式相关属性，只设置数据（文字）相关属性时展现的样式。该样式是微信，微博里使用的样式，也是我个人非常喜欢的样式。

![](https://github.com/knightsj/blog-image-storage/blob/master/react-native/action-sheet/as_1.gif?raw=true)

### 类似iOS原生 ActionSheet的样式

用户可以通过设置某些属性可以实现iOS默认的ActionSheet的样式：

![](https://github.com/knightsj/blog-image-storage/blob/master/react-native/action-sheet/as_2.gif?raw=true)



除此之外，用户还可以通过设置某些属性来实现各种其他的样式：

![](https://github.com/knightsj/blog-image-storage/blob/master/react-native/action-sheet/as_3.gif?raw=true)





下面结合使用方法来看一下如何通过代码来定制这些样式：



## 使用方法



### 安装：

``npm install react-naive-highly-customizable-action-sheet``



引用组件：

``import ActionSheet from 'react-naive-highly-customizable-action-sheet'``



然后给该组件传入标题，选项文字数组，回调方法数组等实现一个ActionSheet的组件。

下面结合一下代码和demo截图讲解一下：



一个默认样式的例子:

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_default.png)

该样式的实现代码：

```jsx
<ActionSheet
   mainTitle="There are three ways to contact. Please choose one to contact."
   itemTitles = {["By phone","By message","By email"]}
   selectionCallbacks = {[this.clickedByPhone,this.clickedByMessage,this.clickedByEmail]}
   mainTitleTextAlign = 'center'
   ref={(actionsheet)=>{this.actionsheet = actionsheet}}
/>
  
//弹出底部菜单
showActionSheet(){
	this.actionsheet.show();  
}

//回调函数
clickedByPhone(){
   alert('By Phone');
}

//回调函数
clickedByMessage(){
    alert('By Message');
}

//回调函数
clickedByEmail(){
    alert('By Email');
}
```



在这里，

- ``mainTitle``：是最上方的标题。
- ``itemTitles``：选项文字的数组。
- ``selectionCallbacks``：点击选项后的回调函数数组。

需要注意的是，选项文字的数组和回调函数数组里的元素应该是一一对应的。不过即使回调函数数组里的元素个数少于选项文字数组里的元素个数也不会引起崩溃。



一个iOS ActionSheet样式的例子：

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_ios.png)

该样式的实现代码：

```jsx
 <ActionSheet
    mainTitle="There are three ways to contact. Please choose one to contact."
    itemTitles = {["By phone","By message","By email"]}
    selectionCallbacks = {[this.clickedByPhone,this.clickedByMessage,this.clickedByEmail]}
    mainTitleTextAlign = 'center'
    contentBackgroundColor = '#EFF0F1'
    bottomSpace = {10}
    cancelVerticalSpace = {10}
    borderRadius = {5}
    sideSpace = {6}
    itemTitleColor = '#006FFF'
    cancelTitleColor = '#006FFF'
    ref={(actionsheet)=>{this.actionsheet = actionsheet}}
/>


//弹出底部菜单
showActionSheet(){
	this.actionsheet.show();  
}

//回调函数
clickedByPhone(){
   alert('By Phone');
}

//回调函数
clickedByMessage(){
    alert('By Message');
}

//回调函数
clickedByEmail(){
    alert('By Email');
}
```



更多其他的样式设定可以参考[demo](https://github.com/knightsj/react-naive-highly-customizable-action-sheet)里的``Example``。

大致介绍完这个组件的功能和使用方法，下面来看一下该组件是如何封装的。



## React Native组件的封装



### 封装些什么



对于GUI编程里视图组件来说，无外乎是以下三个内容：

1. 数据
2. 样式
3. 交互



而对于视图组件的封装，我个人的理解是：封装接收数据的形式，数据与样式之间的转化规则以及交互的逻辑。而这些都是从数据的接收开始的。没有数据的接收就没有UI的展示，更谈不上交互了。



所以在最开始从React Native视图组件的数据接收来说起是比较妥当的。



### 数据接收

在iOS开发中，给view提供数据的方式是通过设置属性或者实现数据源方法来做的。但是在React Native开发中，通常**只能**通过设置属性来传入该组件为了实现某个样式所需要的一些数据。比如在上面的两个例子里，标题，以及选项文字都是通过设置特定的属性来传入的。

而且，为了保证设置属性的类型正确，最好对属性做一个类型检查：

```javascript
import React, {Component, PropTypes} from 'react';

static propTypes = {
 
    mainTitle:PropTypes.string.isRequired,//类型为字符串，且必须传入
    mainTitleFont:PropTypes.number,//类型为数字
    mainTitleColor:PropTypes.string,//类型为字符串
    mainTitleTextAlign:PropTypes.oneOf(['center', 'left']),//二者选其一
    hideCancel:PropTypes.bool,//类型为布尔值

    ...
}
```



注意一下第一行的``mainTitle``属性，在上面将它设置为必须传入的属性。所以如果在这种情况下没有传入该属性，就会出现警告。

上面的只是我举的例子，在我封装的这个组件里没有任何属性是必须传入的。因为要提高定制性，所以所有属性都是可传可不传。



现在我们知道了如何将数据传入到组件里。但是这仅仅是第一步。因为组件所需要的数据可能不仅仅包括用户传入的这些数据，还包括一些通过用户传入的这些数据计算后得到的另一些数据，比如弹窗的总高度。不难理解，弹窗的总高度取决于标题的高度，选项的高度和选项的个数，以及取消项的高度总和。而这个数据显然是通过传入的标题，选项等数据后经过计算得到的。

而且，对于一些可以不一定需要用户传入的数据，可能组件自己也许要提供一下对应属性的默认值。



综上所述，对于数据处理部分，可以分为两类的处理：

1. 计算额外的数据。
2. 提供对应属性的默认值。



分别举两个在该组件中的代码（之间省略了部分内容）讲解一下。



### 数据处理



#### 1. 额外需要计算的数据

```jsx
componentWillMount(){
    
     ...
    //Calculate Title Height
    if (!this.props.mainTitle){
        this.real_titleHeight = 0
    }else {
        this.real_titleHeight = this.state.mainTitleHeight;
    }

    //Calculate Items height
    if (!this.props.itemTitles){
        this.real_itemsPartHeight = 0;
    }else {
        this.real_itemsPartHeight = (this.state.itemHeight + this.state.itemVerticalSpace) * this.props.itemTitles.length;
    }

    //Calculate Cancel part height
    if (this.props.hideCancel){
        this.real_cancelPartHeight = 0;
    }else {
        this.real_cancelPartHeight = this.state.cancelVerticalSpace + this.state.cancelHeight;
    }

    // total content height
    this.totalHeight = this.real_titleHeight +  this.real_itemsPartHeight + this.real_cancelPartHeight + this.state.bottomSpace;
     ...

}
```



在这里，``this.real_titleHeight``,``this.real_itemsPartHeight``,``this.real_cancelPartHeigh``,``this.totalHeight``都是在拿到属性以后，需要额外计算的数据。我把这些工作放在了``componentWillMount()``方法里面。





#### 2. 提供对应属性的默认值



如果用户没有传入标题文字的颜色，则提供一个默认的标题颜色：

```jsx
 constructor(props) {
	super(props);
    this.state = {
      ...
      mainTitleColor:this.props.mainTitleColor?this.props.mainTitleColor:'gray',//主标题颜色
      cancelTitle:this.props.cancelTitle?this.props.cancelTitle:'Cancel',//取消的文字
      ...
    }
 }
```

我们可以看到，如果用户没有设置``mainTitleColor``和``cancelTitle``这两个属性值，组件内部会提供相应的默认值。



### 数据展示



在React Native里，组件的``render()``函数负责渲染组件。因此这个函数里会使用之前计算好的数据来渲染组件：

```jsx
render() {
   retrun(  
     <View>
        {this._renderTitleItem()}
        {this._renderItemsPart()}
        {this._renderCancelItem()}
    </View>)
}

//render title part
_renderTitleItem(){
    if(!this.props.mainTitle){
        return null;
    }else {
        return (
            <TouchableWithoutFeedback>
                <View style={[styles.contentViewStyle]}>
                    <Text>{this.props.mainTitle}</Text>
                </View>
            </TouchableWithoutFeedback>
        )
    }
}

//render selection items part
_renderItemsPart(){
    var itemsArr = new Array();
    let title = this.state.itemTitles[i];
    let itemView =
        <View key={i}>
            {/* Seperate Line */}
            {this._renderItemSeperateLine(showItemSeperateLine)}
            {/* item for selection*/}
            <TouchableOpacity onPress={this._didSelect.bind(this, i)}>
                <View style={[styles.contentViewStyle]} key={i}>
                    <Text style={[styles.textStyle]}>{title}</Text>
                </View>
            </TouchableOpacity>
        </View>
        itemsArr.push(itemView);

    return itemsArr;
}


//render cancel part
_renderCancelItem(){
    return (
      <View style={{width:this.contentWidth,height: this.real_cancelPartHeight}}>
          {/* Seperate Line */}
          {this._renderCancelSeperateLine(showCancelSeperateLine)}
          {/* Cancel Item */}
            <TouchableOpacity onPress={this._dismiss.bind(this)}>
                <View style={[styles.contentViewStyle]}>
                    <Text style={[styles.textStyle]}>{this.state.cancelTitle}</Text>
                </View>
            </TouchableOpacity>
      </View>
    );
}
```





### 交互



组件的交互可以分为两种：有外部回调的交互以及没有外部回调的交互。这个外部回调是指在组件外部所需要执行的函数。比如底部菜单组件：如果用户点击了某一项，菜单会回落，并调用该组件外部的函数（例如退出登录，清除缓存等等）。类比在iOS开发中，可以使用代理或者block的方式进行回调，而在React Native中实现回调的方式与iOS中block的方式类似。



#### 有回调的交互

在React Native中，如果需要调用外部的函数，就需要在一开始的时候将该函数作为属性传入组件中。然后拦截用户的点击，调用相应的回调函数。这里面分为三个步骤：

1. 传入回调函数
2. 拦截用户操作
3. 调用回调函数



**1. 传入回调函数：**

```jsx
static propTypes = {
  
    //selection items callback
    selectionCallbacks:PropTypes.array,
}
```

> 在这里，``selectionCallbacks``是对应选择项的回调函数数组属性。这里因为选择项数量不确定，所以用数组来保存回调函数。



**2. 拦截用户操作(点击)：**

```jsx
<TouchableOpacity onPress={this._didSelect.bind(this, i)}  activeOpacity = {0.9}>
    <View style={styles.contentViewStyle} key={i}>
        <Text style={styles.textStyle}>{title}</Text>
    </View>
</TouchableOpacity>
```

> 在这里，使用了``TouchableOpacity``组件让``View``组件获得可以被点击的能力，并且绑定了函数``_select(index)``。



**3. 调用回调函数：**

```jsx
//取出相应的回调函数并调用
_select(i) {
    let callback = this.state.selectionCallbacks[i];
    if(callback){
        {callback()}
    }
}
```

> 在这里，_didSelect(index)函数是某个选项被点击后调用的函数。该函数拿到传入的index值，从callback数组里面获取对应index的回调函数并调用。而且为了避免崩溃，还判断了callback是否为空。



#### 没有回调的交互

如果这个交互没有回调就比较简单了，在组件内部做就可以了。比如点击取消后的回落事件：

```jsx
<TouchableOpacity onPress={this._dismiss.bind(this)} activeOpacity = {0.9}>
    <View style={styles.contentViewStyle}>
        <Text style={styles.textStyle}>{this.state.cancelTitle}</Text>
    </View>
</TouchableOpacity>

//dismiss ActionSheet
_dismiss() {
    if (!this.state.hide) {
        this._fade();
    }
}
```

在这里除了使菜单回落以外，再点击取消的时候还给了用户反馈：点击时背景色的透明度改变。实现方法是利用的``TouchableOpacity``的``activeOpacity = {0.9}``



OK，现在讲完了数据和交互，再来看一下React Native是如何支持动画效果的（因为用到了所以就顺带讲一下了）。



### 动画效果

一般来说，底部菜单在弹出和回落的时候是有动画效果的，React Native的动画效果可以用其内置的``Animated``库来实现。



结合菜单弹出的例子来说明一下：

```javascript
//animation of showing
_appear() {
    Animated.parallel([
        Animated.timing(
            this.state.opacity, //动画改编的变量
            {
                easing: Easing.linear,
                duration: 200,  //动画时长，单位是毫秒
                toValue: 0.7,   //终点值
            }
        ),
        Animated.timing(
            this.state.offset,
            {
                easing: Easing.linear,
                duration: 200,
                toValue: 1,
            }
        )
    ]).start();
    }
```

在这里，

- ``Animated.parallel``函数负责执行**同时执行的组合动画**。既然是组合动画，那么传入的就应该是一个动画的数组。仔细看一下就会发现这里有两个``Animated.timing``函数。

- ``Animated.timing``函数负责执行以时间为单位的动画。从注释上不难看出，在这里同时执行的两个动画是：

  - ``this.state.opacity``值在200毫秒内，从0到0.7渐变的动画。
  - ``this.state.offset``值在200毫秒内，从0到1渐变的动画。

- 最底部的``start()``函数触发了这个组合动画。

  ​

> 这里没有提供起点值，因为在这里直接获取的是传入变量的当前值。



相对底部菜单的弹出动画，来看一下底部菜单的回落动画：

```javascript
//animation of fading
_fade() {
    Animated.parallel([
        Animated.timing(
            this.state.opacity,
            {
                easing: Easing.linear,
                duration: 200,
                toValue: 0,
            }
        ),
        Animated.timing(
            this.state.offset,
            {
                easing: Easing.linear,
                duration: 200,
                toValue: 0,
            }
        )
    ]).start((finished) => this.setState({hide: true}));
}
```



有关动画的知识可以查看官方文档[React Native :动画](http://reactnative.cn/docs/0.48/animations.html#conte



其实到这里，对于组件的封装就基本讲完了，讲解的内容还是集中在数据这一块，组件是怎么画出来的就不讲解了。因为毕竟每个组件将数据转化为样式的代码是不一样的，学会一个弹出菜单的画法对于画其他的组件没有太大的借鉴意义。但是对于一个通用组件来说，其定制性必须达到一定标准才可以。所以相对于讲解“组件是如何画出来的”，我认为讲一下“提高组件定制性”应该更实际一些。





### 为提高定制性所做的工作：

最开始做这个控件也仅仅只能设置标题，选项以及回调函数，样式也只有这一种：

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_one.png)



但是为了提高定制性，支持更多的样式，也为了自己能更好地了解React Native，就决定挑战一下，看定制性能提高到什么程度。

如上文所说，在React Native里，组件的数据传递是通过设置其属性来实现的。所以如果想要提高组件的定制性就需要增加该组件的属性。



看一下该组件的所有属性：

- ``itemTitles``*(Array)*:选择项的标题数组
- ``selectionCallbacks``*(Array)*：点击选项的回调数组


- ``mainTitle``*(String)*:标题文字
- ``mainTitleFont``*(Number)*:标题字体
- ``mainTitleColor``*(String)*:标题颜色
- ``mainTitleHeight``*(Number)*:标题栏高度
- ``mainTitleTextAlign``*(String)*:标题对齐方式
- ``mainTitlePadding``*(Number)*:标题内边距
- ``itemTitleFont``*(Number)*:选择项字体
- ``itemTitleColor``*(String)*:选择项颜色
- ``itemHeight``*(Number)*:选择栏高度


- ``cancelTitle``*(String)*:取消项标题，默认为'Cancel'
- ``cancelTitleFont``*(Number)*:取消标题字体
- ``cancelTitleColor``*(String)*:取消标题颜色
- ``cancelHeight``*(Number)*:取消栏高度
- ``hideCancel``*(Bool)*:是否隐藏取消项（默认不隐藏）


- ``fontWeight``*(String)*:所有文字的字体粗细（同时设置标题，选择项，取消项的字体粗细）
- ``titleFontWeight``*(String)*:标题的字体粗细，默认为'normal'
- ``itemFontWeight``*(String)*:选择项的字体粗细，默认为'normal'
- ``cancelFontWeight``*(String)*:取消项的字体粗细，默认为'bold'


- ``contentBackgroundColor``*(String)*:所有项目的背景色（同时设置标题，选择项，取消项的背景色）
- ``titleBackgroundColor``*(String)*:标题的背景色（默认是白色）
- ``itemBackgroundColor``*(String)*:选择项的背景色（默认是白色）
- ``cancelBackgroundColor``*(String)*:取消项的背景色（默认是白色）
- ``itemSpaceColor``*(String)*:选择项之间的分割线颜色（默认是浅灰色）
- ``cancelSpaceColor``*(String)*:取消项和最后一个选择项之间的分割线颜色（默认是浅灰色）


- ``itemVerticalSpace``*(Number)*:选择项之间分割线的高度


- ``cancelVerticalSpace``*(Number)*:取消项和最后一个选择项之间的分割线的高度
- ``bottomSpace``*(Number)*:屏幕底部距离取消项底部的距离
- ``sideSpace``*(Number)*:弹出框左右侧边距离屏幕左右侧边的距离


- ``borderRadius``*(Number)*:弹出框的圆角


- ``maskOpacity``*(Number)*:mask的透明度（默认为0.3）



不难看出，该组件的三个部分（标题，选项，取消）里，每个部分都有各自对应的属性可以设置。因为在设计这个组件的时候就将这三个部分高度解耦了：每个部分都互不影响，有各自的数据（除了少数可以共同使用的数据），并分别进行绘制。

比如，我们可以设置：



#### 每个部分文字内容，字体大小，高度

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_setting1.png)



#### 背景颜色（可以统一设置，也可以单独设置）

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_setting2.png)



#### 分割线高度，距离底部的高度，距离屏幕侧边的距离

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_setting3.png)



#### 分割线的颜色

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_setting4.png)



上面这些图片的效果对应的代码在[demo](https://github.com/knightsj/react-naive-highly-customizable-action-sheet)中都有提供(具体查看Example文件夹)。

另外该组件也支持一些比较极端的情况，虽然可能需求上极少遇到，但还是提供了支持。



#### 极端情况：

![](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/react_native/action_sheet/rn_as_setting5.png)


高度解耦的程度可以通过这最后一张图看出来：主标题，选择项，取消项都可以根据传入属性的情况来展示，互不影响。而且在都不设置的情况下，只展示了灰色的底部mask。







## 最后的话

写这个组件一共花了3天的时间，其实第一天就已经完成了默认样式的开发。而后2天主要做的是提高定制性的工作。因为定制性的工作是与数据处理和应用分不开的，而自己对JavaScript语法了解得不是很好，所以期间写了不少的bug。值得庆幸的是，由于React Native本身搭建UI的能力很强，效率很高，所以数据处理好了之后工作量就不大了。


毕竟是自己封装的第一个React Native组件，我相信它还是有很多提升空间的，比如数据处理这一块可能有不妥的地方，还需要各位能给出宝贵的意见和建议。



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



