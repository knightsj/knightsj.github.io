---
title: 从一个实战项目来看一下React Native开发的几个关键技术点
tags: [React Native]
categories: React Native
---


在进行了2个星期的基础学习（Flexbox, React.js, JSX, JavaScript）之后，想通过一个实战项目来提高React Native的开发水平，于是找到了下面这个项目：

# 一. 项目介绍

这是我在学习[贾鹏辉](http://www.devio.org/)老师在慕课网上的一个很火的[React Native实战的教程](http://coding.imooc.com/class/89.html)后，写出的课程Demo。该课程是慕课网里很火的一个React Native课程，当初在看了课程介绍和课程安排觉得讲解的点还是很全的，所以毫不犹豫地买了下来。

从看视频，敲代码到重构，改bug，大概花了2个多星期的时间，除了调用友盟的SDK以及CodePush集成之外，其他的部分都基本完成了，而且同时可以在iOS和Android设备上运行：
![上排是iOS模拟器 | 下排是Android模拟器](http://oih3a9o4n.bkt.clouddn.com/rn_13.png)

而且比较吸引人的是该项目可以实现多个主题的切换：
![多主题切换](http://oih3a9o4n.bkt.clouddn.com/rn_15.png)

>切换的技术实现会在下文给出。

用一个动图来过一遍大致的需求：
![](http://oih3a9o4n.bkt.clouddn.com/github%E5%AE%A2%E6%88%B7%E7%AB%AF_4.gif)




Demo GitHub地址：[GitHubPopular-SJ](https://github.com/knightsj/GitHubPopular-SJ)
可以按照README文件里的方法运行该项目。

>已经贾老师允许上传到GitHub

值得一提的是：这确实是一门物有所值的课程，可以让想入门React Native的开发者少走很多弯路。虽然我上传的Demo可以实现视频里大部分功能，但是经过调试，修改后的代码信息量还是很有限的，而且老师在视频中讲解的很多关于实际开发的知识点在代码中并没有体现出来，所以还是建议各位报名参加课程来提高自己的开发水平。

<!-- more -->

# 二. React Native开发的几个关键技术点

首先用一张思维导图来看一下第二节讲的内容：
![](http://oih3a9o4n.bkt.clouddn.com/rn_16_1.png)

## 2.1 组件化的思想

React Native是React在移动端的跨平台方案。如果想更快地理解和掌握React Native开发，就必须先了解React。

React是FaceBook开源的一个前端框架，它起源于 Facebook 的内部项目，并于 2013 年 5 月开源。因为React 拥有较高的性能，代码逻辑非常简单，所以越来越多的人已开始关注和使用它，目前该框架在Github上已经有7万+star。

React采用组件化的方式开发，通过将view构建成组件，使得代码更加容易得到复用，能够很好的应用在大项目的开发中。有一句话说的很形象：在React中，构建应用就像搭积木一样。

因此，如果想掌握React Native，就必须先了解React中的组件。

那么问题来了，什么是组件呢？

在React中，在UI上每一个功能相对独立的模块就会被定义为组件。  相对小的组件可以通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。

因此，整个UI是一个通过小组件构成的大组件，而且每个组件只关心自己部分的逻辑，彼此独立。

React认为一个组件应该具有如下特征：

- 可组合（Composeable）：一个组件易于和其它组件一起使用，或者嵌套在另一个组件内部。如果一个组件内部创建了另一个组件，那么说父组件拥有它创建的子组件，通过这个特性，一个复杂的UI可以拆分成多个简单的UI组件；
- 可重用（Reusable）：每个组件都是具有独立功能的，它可以被使用在多个UI场景；
- 可维护（Maintainable）：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护；

举个🌰，我们看一下这个Demo使用的导航栏：

![](http://oih3a9o4n.bkt.clouddn.com/rn_1_1_1.png)

封装好的导航栏就可以被称之为一个组件，它符合上述三个特点：

1. 可组合：可以将导航栏组件放在页面组件中作为页面组件的子组件。而且在导航栏组件的内部，也有按钮组件等子组件。
2. 可重用：如果封装好了该组件，就可以放在任意需要导航栏的页面（组件）使用，也可以放在其他项目中使用。
3. 可维护：因为具有独立的功能和展示逻辑，所以便于定位和修改。

在了解了组件的基本概念以后，我们来看一下组件其他的一些相关知识。

## 2.2 组件的属性与状态

在React Native（React.js）里，组件所持有的数据分为两种：

1. 属性（props）：组件的props是不可变的，它只能从其他的组件（例如父组件）传递过来。
2. 状态（state）：组件的state是可变的，它负责处理与用户的交互。在通过用户点击事件等操作以后，如果使得当前组件的某个state发生了改变，那么当前组件就会触发``render()``方法刷新自己。

举一个这个项目的收藏页面来说：

![](http://oih3a9o4n.bkt.clouddn.com/rn_2_1_1.png)

我们可以看到这个页面有两个子页面，一个是‘最热’页面（组件），另一个是‘趋势‘页面（组件）。那么这两个组件都有什么props和state呢？

### props
首先看一下props：
由于props是从其父组件传递过来的，那么可想而知，props的声明应该是在当前组件的父组件里来做。在React Native中，通常props的声明是和当前组件的声明放在一起的：

```javascript
//最热子页面
<FavoriteTabPage  {...this.props} tabLabel='最热' flag={FlAG_STORAGE.flag_popular}/>

//趋势子页面
<FavoriteTabPage  {...this.props} tabLabel='趋势' flag={FlAG_STORAGE.flag_trending}/>
```

在这里，收藏页面是父组件，而最热页面和趋势页面是其子组件。在收藏页面组件里声明了最热页面和趋势页面的组件。

而且我们也可以看到，最热页面和趋势页面组件都用的是同一个组件：``FavoriteTabPage``，而这两个页面的不同点只在于传入的两个props的不同：``tabLabel``和``flag``。

而在``FavoriteTabPage``组件内部，如果想调用flag这个props，可以使用``this.props.flag``来调用。

再来看一下state：

### state
下面是最热和趋势页面的组件：

```javascript
class FavoriteTabPage extends Component{

    //组件的构造方法
    constructor(props){

        super(props);
        this.state={
            dataSource:new ListView.DataSource({rowHasChanged:(r1,r2)=>r1!==r2}),
            isLoading:false,
        }
    }
    ...
}
```

这里面定义了两个state:
1. dataSource:列表的数据源
2. isLoading:是否正在刷新

这两个state都是将来可能经常变化的。比如在网络请求以后，列表的数据源会被替换掉，这个时候就要调用
```javascript
 this.setState({
      //把新的值newDataArr对象传给dataSource
      dataSource:newDataArr
 })
```
来触发``render()``方法来刷新列表组件。


## 2.3 组件的生命周期

和iOS开发里``ViewController``的生命周期类似，组件也有生命周期，大致分为三大阶段：

- Mounting：已插入真实 DOM
- Updating：正在被重新渲染
- Unmounting：已移出真实 DOM

>DOM是前端的一个概念，暂时可以粗略理解为一个页面的树形结构。

在每个阶段都有相应的状态和与之对应的回调函数，具体可以看下图：

![组件的生命周期](https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/master/React%20Native%E4%B9%8BReact%E9%80%9F%E5%AD%A6%E6%95%99%E7%A8%8B/images/component-lifecycle.jpg)

> 上图来自：[贾鹏辉的技术博客：React Native之React速学教程(中)](http://www.devio.org/2016/08/10/React-Native%E4%B9%8BReact%E9%80%9F%E5%AD%A6%E6%95%99%E7%A8%8B-(%E4%B8%AD)/)

从上图中我们可以看到，React 为每个状态都提供了两种回调函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用。

在这里讲一下这其中几个重要的回调函数：

#### render()

该函数是组件的渲染回调函数，该函数是必须实现的，并且必须返回一个组件或一个包含多个子组件的组件。

> 注意：该函数可以被调用多次：初始化时的渲染以及state改变以后的渲染都会调用这个函数。

#### componentDidMount()

在初始化渲染执行之后立刻调用一次，也就是说，在这个函数调用时，当前组件已经渲染完毕了，相当于iOS开发中``ViewController``里的``viewDidLoad``方法。

> 我们通常在这个方法里执行网络请求操作。

#### componentWillReceiveProps(object nextProps)

在当前组件接收到新的 props 的时候调用。此函数可以作为 react 在 prop 传入之后， render() 渲染之前更新 state 的机会。新的props可以从参数里取到，老的 props 可以通过 this.props 获取到。

> 注意：在初始化渲染的时候，该方法不会调用。

#### shouldComponentUpdate(object nextProps, object nextState):

在接收到新的 props 或者 state，将要渲染之前调用。如果确定新的 props 和 state 不会导致组件更新，则此处应该 返回 false，这样组件就不会更新，减少了性能上不必要的损耗。

> 注意：该方法在初始化渲染的时候不会调用。
>
> 

#### componentWillUnmount()

在组件从 DOM 中移除的时候立刻被调用。例如当前页面点击返回键跳转到上一页面的时候就会调用。

> 我们通常在这个方法里移除通知。具体做法在后文会提到。

到此，已经讲解了一些组件相关的知识，下面来看一下我们如何使用组件来搭建界面。

## 2.4 使用组件来搭建界面

在这里我们举几个例子来看一下在React Native里搭建View的方式。

首先我们来看一下最热页面的cell是如何布局的：

### 2.41 搭建cell组件

首先举一个在最热标签页面列表里的一个cell为例，讲解一下一个简单的UI组件是如何实现的：

![最热标签页面的cell](http://oih3a9o4n.bkt.clouddn.com/rn_4_1_1.png)

我们把该组件定名为：``RespositoryCell``，结合代码来看一下具体的实现：

```javascript

export default class RespositoryCell extends Component{
 
    ...
    
    render(){

        //获取当前cell的数据赋值给item
        let item = this.props.projectModel.item?this.props.projectModel.item:this.props.projectModel;

        //收藏按钮
        let favoriteButton = <TouchableOpacity
            onPress={()=>this.onPressFavorite()}
        >
            <Image
                style={[styles.favoriteImageStyle,this.props.theme.styles.tabBarSelectedIcon]}
                source={this.state.favoriteIcon}
            />
        </TouchableOpacity>

        return(
            <TouchableOpacity
                 onPress={this.props.onSelect}
                 style={styles.container}
            >
                //整个cell的view
                <View style={styles.cellContainerViewStyle}>

                    //1. 项目名称
                    <Text style={styles.repositoryTitleStyle}>{item.full_name}</Text>

                    //2. 项目介绍
                    <Text style={styles.repositoryDescriptionStyle}>{item.description}</Text>

                    //3. 底部 container
                    <View style={styles.bottomContainerViewStyle}>

                        //3.1 作者container
                        <View style={styles.authorContainerViewStyle}>

                            //3.11 作者名称
                            <Text style={styles.bottomTextStyle}>Author:</Text>

                            //3.12 作者头像
                            <Image
                                style={styles.authorAvatarImageStyle}
                                source={{uri:item.owner.avatar_url}}
                             />
                        </View>

                        //3.2 star container
                        <View style={styles.starContainerViewStyle}>
                //3.21 star标题
                            <Text style={styles.bottomTextStyle}>Starts:</Text>
                       //3.21 star数量
                            <Text style={styles.bottomTextStyle}>{item.stargazers_count}</Text>
                        </View>

                        //3.3 收藏按钮
                        {favoriteButton}
                     </View>
                 </View>
            </TouchableOpacity>)
    }
}
          
```

> 这里省略了处理交互事件等的函数，为了让大家集中在cell的布局和样式上。

- 这里声明了``RespositoryCell``组件，它继承于``Component``，也就是组件类，即是说，声明组件的时候必须都要继承与这个类。
- 集中看一下该组件的render方法，它返回的是该组件的实际布局：在语法上使用JSX，类似于HTML的标签式语法，很清楚地将cell的层级展现了出来：
  - 最外层被一个``View``组件包裹着，里面第一层有三个子组件：两个``Text``组件和一个作为底部背景的``View``组件。
  - 底部背景的``View``组件又有三个子组件：``View``组件（显示作者信息），``View``组件（显示star信息）,收藏按钮。

试着结合代码来看一下下面的图片，可以看出组件的实际布局与代码的布局是高度一致的：

![Cell 布局 ](http://oih3a9o4n.bkt.clouddn.com/rn_5_2.png)

然而仅仅定义组件的层级关系是不够的，我们还需要定义组件的样式（例如图片组件的大小样式等等），这时候就通过定义一个样式的对象（通常使用常量对象）来定义一些需要使用的样式：

```javascript
//样式常量
const styles =StyleSheet.create({

    //项目cell的背景view的style       
    cellContainerViewStyle:{
          
        //背景色
        backgroundColor:'white',
          
        //内边距
        padding:10,
          
        //外边距
        marginTop:4,
        marginLeft:6,
        marginRight:6,
        marginVertical:2,
          
        //边框
        borderWidth:0.3,
        borderColor:'#dddddd',
        borderRadius:1,
          
        //iOS的阴影
        shadowColor:'#b5b5b5',
        shadowOffset:{width:3,height:2},
        shadowOpacity:0.4,
        shadowRadius:1,
      
        //Android的阴影
        elevation:2
    },
  
    //项目标题的style
    repositoryTitleStyle:{
        fontSize:15,
        marginBottom:2,
        color:'#212121',
    },
  
    //项目介绍的style  
    repositoryDescriptionStyle:{
        fontSize:12,
        marginBottom:2,
        color:'#757575'
    },

    //底部container的style
    bottomContainerViewStyle:{
        flexDirection:'row',
        justifyContent:'space-between'
    },

    //作者container的style
    authorContainerViewStyle:{
        flexDirection:'row',
        alignItems:'center'
    },

    //作者头像图片的style
    authorAvatarImageStyle:{
        width:16,
        height:16
    },

    //星星container的style
    starContainerViewStyle: {
        flexDirection:'row',
        alignItems:'center'
    },

    //底部文字的style
    bottomTextStyle:{
       fontSize:11,
    },

    //收藏按钮的图片的style
    favoriteImageStyle:{
        width:18,
        height:18
    }
    
})
```

在上面这段代码里定义了``RespositoryCell``组件所使用的所有样式，通过将其赋值给对应子组件的style属性来实现对组件样式的修改，例如我们看一下项目标题的组件和其样式的定义：

```javascript
<Text style={styles.repositoryTitleStyle}>{item.full_name}</Text>
```

在这里，我们首先定义了一个Text组件用来显示项目的标题。然后将``styles.repositoryTitleStyle``赋给了当前Text组件的style,而标题的具体内容，则通过``item.full_name``来获取。

需要注意的是，在JSX的语法中，对象需要被{}来包裹住，否则会被认为是常量。比如，如果这里写成：

```javascript
<Text style={styles.repositoryTitleStyle}>item.full_name</Text>
```

那么所有项目cell的标题则都会显示为''item.full_name''，有图有真相：

![](http://oih3a9o4n.bkt.clouddn.com/rn_5_1_1.png)

这是初学者比较常犯的错误，所以要注意：在搭建页面的时候，一定要区分是对象还是常量。如果是对象就必须要用大括号括起来！如果是对象就必须要用大括号括起来！如果是对象就必须要用大括号括起来！

> 这里每个样式里面的长，宽，内外边距，以及``flexDirection``等flexBox相关的布局属性就不介绍了。可以通过查找本文最后的相关链接来学习。

### 2.42 搭建静态表格页

在React Native中搭建个人页，设置页这种静态表格页面的时候，可以用``ScrollView``组件包裹各种封装好的cell组件的形式实现。看一下这个Demo的个人页的效果图和代码实现：

![个人页](http://oih3a9o4n.bkt.clouddn.com/rn_6_1_1.png)

我们在项目中新建一个JavaScript文件，取名为取名为``MinePage.js`` 。该文件就是个人页面的实现。结合代码来看一下它的实现（删除了处理点击cell的逻辑处理代码）：

```javascript
//区域一：引用区：
//引用React，Component(组件类)以及React Native中自带的组件
import React, { Component } from 'react';
import {
    StyleSheet,
    Text,
    View,
    Image,
    ScrollView,
    TouchableHighlight,
} from 'react-native';

//引入项目中定义的其他组件(页面组件)和常量，路径为相对路径
import NavigationBar from '../../common/NavigationBar'
import {MORE_MENU} from '../../common/MoreMenu'
import GlobalStyles from '../../../res/styles/GlobalStyles'
import ViewUtil from '../../util/ViewUtils'
import {FLAG_LANGUAGE}from '../../dao/LanguageDao'
import AboutPage from './AboutPage'

import CustomKeyPage from './CustomKeyPage'
import SortPage from './SortKeyPage'
import AboutMePage from './AboutMePage'
import CustomThemePage from './CustomThemePage'
import BaseComponent from '../../base/BaseCommon'

//区域二：页面组件定义区域：
export default class MinePage extends BaseComponent {

 ...
    
    //渲染页面中List中每个cell的统一函数
    createSettingItem(tag,icon,text){
        return ViewUtil.createSettingItem(()=>this.onClick(tag),icon,text,this.state.theme.styles.tabBarSelectedIcon,null);
    }

    render(){
        return <View style={GlobalStyles.listViewContainerStyle}>
            <NavigationBar
                title={'我的'}
                style={this.state.theme.styles.navBar}
            />
            <ScrollView>

                {/*=============项目信息Section=============*/}
                <TouchableHighlight
                    underlayColor= 'transparent'
                    onPress={()=>this.onClick(MORE_MENU.About)}
                >
                    <View style={styles.itemInfoItemStyle}>
                        <View style={{flexDirection:'row',alignItems:'center'}}>
                            <Image source={require('../../../res/images/ic_trending.png')}
                                   style={[{width:40,height:40,marginRight:10},this.state.theme.styles.tabBarSelectedIcon]}
                            />
                            <Text>GitHub Popular 项目信息</Text>
                        </View>
                        <Image source={require('../../../res/images/ic_tiaozhuan.png')}
                            style={[{height:22,width:22},this.state.theme.styles.tabBarSelectedIcon]}
                        />
                    </View>
                </TouchableHighlight>
                {/*分割线*/}
                <View style={GlobalStyles.cellBottomLineStyle}></View>

                {/*=============趋势管理Section=============*/}
                <Text style={styles.groupTitleStyle}>趋势管理</Text>
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                {/*自定义语言*/}
                {this.createSettingItem(MORE_MENU.Custom_Language,require('../../../res/images/ic_custom_language.png'),'自定义语言')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                <View style={GlobalStyles.cellBottomLineStyle}></View>

                {/*语言排序*/}
                {this.createSettingItem(MORE_MENU.Sort_Language,require('../../../res/images/ic_swap_vert.png'),'语言排序')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>

                {/*=============标签管理Section=============*/}
                <Text style={styles.groupTitleStyle}>标签管理</Text>

                <View style={GlobalStyles.cellBottomLineStyle}></View>
                {/*自定义标签*/}
                {this.createSettingItem(MORE_MENU.Custom_Key,require('../../../res/images/ic_custom_language.png'),'自定义标签')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                {/*标签排序*/}
                {this.createSettingItem(MORE_MENU.Sort_Key,require('../../../res/images/ic_swap_vert.png'),'标签排序')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                {/*标签移除*/}
                {this.createSettingItem(MORE_MENU.Remove_Key,require('../../../res/images/ic_remove.png'),'标签移除')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>

                {/*=============设置Section=============*/}
                <Text style={styles.groupTitleStyle}>设置</Text>
                {/*自定义主题*/}
                <View style={GlobalStyles.cellBottomLineStyle}></View>
                {this.createSettingItem(MORE_MENU.Custom_Theme,require('../../../res/images/ic_view_quilt.png'),'自定义主题')}
                <View style={GlobalStyles.cellBottomLineStyle}></View>

                {/*展示自定义主题页面*/}
                {this.renderCustomTheme()}
            </ScrollView>
        </View>
    }
}

//区域三：定义页面组件样式区：
const styles = StyleSheet.create({

    itemInfoItemStyle:{
        flexDirection:'row',
        justifyContent:'space-between',
        alignItems:'center',
        padding:10,
        height:76,
        backgroundColor:'white'
    },

    groupTitleStyle:{
        marginLeft:10,
        marginTop:15,
        marginBottom:6,
        color:'gray'
    }
});
```

在上面的代码中，我们可以看到一个页面组件的全貌，它大致分为三个区域：

1. 引用区域
2. 定义组件区域
3. 定义样式区域

下面两个区域在上一节已经介绍过。第一个区域，引用区域一般写在组件文件的开头，在这里一般是需要引入该组件需要的其他组件或者常量。

现在看一下该组件的``render()``函数，它返回了用来包裹整个页面的``View``组件，该组件有两个子组件

- NavigationBar组件（导航栏），传入了两个props：title和style。
- ScrollView组件，包裹了项目信息Cell的View组件，分割线，项目Cell的View组件。需要注意的是，每个cell的组件都比较类似，所以在这里将生成它的代码封装起来做一个函数来调用：

```javascript
createSettingItem(tag,icon,text){
        return  ViewUtil.createSettingItem(()=>this.onClick(tag),icon,text,this.state.theme.styles.tabBarSelectedIcon,null);
    }
```

可以看到这个函数传入的参数有三个：用来作标记的tag，图片 和标题文字。它的返回值通过调用ViewUtil组件的``createSettingItem``方法来实现。这个方法用于统一生成类似布局的cell。

看一下这个函数的实现：

```javascript
 
//ViewUtils.js
static createSettingItem(callBack,icon,text,tintColor,expandableIcon){

        //如果不传入icon，则不显示
        let image = null;
        if (icon){
            image = <Image
                source={icon}
                resizeMode='stretch'
                style={[{width:18,height:18,marginRight:10},tintColor]}
            />
        }
        return (
            <View style={{backgroundColor:'white'}}>
                <TouchableHighlight
                    onPress={callBack}
                    underlayColor= 'transparent'
                >
                    <View style={styles.settingItemContainerStyle}>
                        <View style={{flexDirection:'row',alignItems:'center'}}>
                            {image}
                            <Text>{text}</Text>
                        </View>
                        <Image source={expandableIcon?expandableIcon:require('../../res/images/ic_tiaozhuan.png')}
                               style={[{marginRight:0,height:22,width:22},tintColor]}//要用括号
                        />
                    </View>
                </TouchableHighlight>
            </View>
        )
    }
```

这个函数有5个参数：

- callback：点击cell时调用的方法，需要父组件传入
- icon：cell左侧的图片
- text：cell标题
- tintColor：cell的主题颜色
- expandableIcon:cell右侧的图片（三角箭头）

因为在React Native中没有特定的``Button``组件，所以实现组件的点击都是通过被``TouchableHighlight``等可点击组件包裹来实现的。

常用的可以实现点击效果的是``View``组件和``Text``组件。

注意一下``TouchableHighlight``里面传入的两个props：

1. 如果需要在点击时颜色不变，可以将它的``underlayColor``设为`transparent`。
2. 可以把点击时触发的函数传给它的``onPress``属性。所以，如果该cell被点击了，就会触发传入的callback。这个callback就等于当初传过来的箭头函数：

```javascript
ViewUtil.createSettingItem(()=>this.onClick(tag),icon,text,this.state.theme.styles.tabBarSelectedIcon,null);
```

该函数是在个人页被调用的，用来实现点击cell时的跳转等操作。

> 注意，在这个ViewUtils类中，我们可以定义很多常用的View组件，例如这种设置页面的cell，导航栏上的返回按钮等等。

现在cell的实现讲完了，下面讲一下分割线和session的title。

先来看一下分割线：

```javascript
<View style={GlobalStyles.cellBottomLineStyle}></View>
```

它的样式调用了``GlobalStyles``的``cellBottomLineStyle``。因为``GlobalStyles``是全局的样式文件（单独写在了一个js文件中），可以使用它来专门管理一些常用的样式。这样一来，我们就不需要在不同页面的组件页面里面重复声明样式常量了。

我们看一下如何定义全局的样式文件：

```javascript
//GlobalStyles.js
module.exports ={

    //cell分割线样式
    cellBottomLineStyle: {
        height: 0.4,
        opacity:0.5,
        backgroundColor: 'darkgray',
    },

    //cell背景色样式
    cell_container: {
        flex: 1,
        backgroundColor: 'white',
        padding: 10,
        marginLeft: 5,
        marginRight: 5,
        marginVertical: 3,
        borderColor: '#dddddd',
        borderStyle: null,
        borderWidth: 0.5,
        borderRadius: 2,
        shadowColor: 'gray',
        shadowOffset: {width:0.5, height: 0.5},
        shadowOpacity: 0.4,
        shadowRadius: 1,
        elevation:2
    },

    //当前屏幕高度
    window_height:height,
    //当前屏幕宽度
    window_width:width,

};
```

因为使用了``module.exports``方法，在这里定义的全局样式可以在外部随意使用。

最后，Section Title的View就比较简单了，就是一个带有灰色文字的``View``组件。

```
<Text style={styles.groupTitleStyle}>趋势管理</Text>
```
### 2.43 搭建app基本骨架：TabBar + NavigationBar

做移动开发的朋友们应该比较了解，底部TabBar，顶部NavigationBar是移动app很主流的一个全局界面方案。然而在原生的React Native组件里面，没有将二者整合在一起的组件。幸运的是，有一个第三方组件比较好的将二者整合到了一起：[react-native-tab-navigator](https://github.com/happypancake/react-native-tab-navigator).

在它的主页告诉我们其导入方式是在项目主目录下执行：``npm install react-native-tab-navigator —save``命令。但是我建议使用``yarn``来引入所有第三方的组件：``yarn add react-native-tab-navigator``。因为使用npm命令安装第三方组件的时候有时会出现问题。而且建议引入第三方组件的时候都是用``yarn``来操作，比较保险一点。

在确认``react-native-tab-navigator``组件下载到了npm文件夹以后，就可以在项目中导入使用了。下面来看一下使用方法：

```javascript
//导入 react-native-tab-navigator 组件，取名为 TabNavigator(随意取名)
import TabNavigator from 'react-native-tab-navigator';

//每个tab对应的唯一标识，可以在外部获取
export const FLAG_TAB = {
    flag_popularTab: 'flag_popularTab',
    flag_trendingTab: 'flag_trendingTab',
    flag_favoriteTab: 'flag_favoriteTab',
    flag_myTab: 'flag_myTab'
}

export default class HomePage extends BaseComponent {

    constructor(props){
      
        super(props);
        
        let selectedTab = this.props.selectedTab?this.props.selectedTab:FLAG_TAB.flag_popularTab

        this.state = {
            selectedTab:selectedTab,
            theme:this.props.theme
        }
    }

    _renderTab(Component, selectedTab, title, renderIcon) {
        return (
            <TabNavigator.Item
                selected={this.state.selectedTab === selectedTab}
                title={title}
                selectedTitleStyle={this.state.theme.styles.selectedTitleStyle}
                renderIcon={() => <Image style={styles.tabItemImageStyle}
                                         source={renderIcon}/>}
                renderSelectedIcon={() => <Image
                    style={[styles.tabItemImageStyle,this.state.theme.styles.tabBarSelectedIcon]}
                    source={renderIcon}/>}
                    onPress={() => this.onSelected(selectedTab)}>
                <Component {...this.props} theme={this.state.theme} homeComponent={this}/>
            </TabNavigator.Item>
        )
    }

    render() {
        return (
            <View style={styles.container}>
                <TabNavigator
                    tabBarStyle={{opacity: 0.9,}}
                    sceneStyle={{paddingBottom: 0}}
                >
                    {this._renderTab(PopularPage, FLAG_TAB.flag_popularTab, '最热', require('../../../res/images/ic_polular.png'))}
                    {this._renderTab(TrendingPage, FLAG_TAB.flag_trendingTab, '趋势', require('../../../res/images/ic_trending.png'))}
                    {this._renderTab(FavoritePage, FLAG_TAB.flag_favoriteTab, '收藏', require('../../../res/images/ic_favorite.png'))}
                    {this._renderTab(MinePage, FLAG_TAB.flag_myTab, '我的', require('../../../res/images/ic_my.png'))}
                </TabNavigator>
            </View>
        )
    }

}
```

在这里我省略了其他的代码，只保留了关于搭建``TabBar && NavigationBar``的代码。

这里定义的是``HomePage``组件，是这个Demo用来管理这些tab的组件。

因为这个Demo一共有四个tab，所以将渲染的tab的代码抽取出来作为单独的一个函数：``_renderTab``。该函数有四个参数：

- Component：当前tab被点击后显示的组件。
- selectedTab：当前tab的唯一标识。
- title：当前tab的标题。
- renderIcon：当前tab的图标。

在``_renderTab``方法里，我们返回一个``TabNavigator.Item``组件，除了一些关于tab的props的定义以外，我们将属于该tab的组件填充了进去：

```javas
<Component {...this.props} theme={this.state.theme} homeComponent={this}/>
```

在这里，{...this.props}是将当前``HomePage``的所有props赋给这个``Component``。还有另外两个props也定义了进去：``theme``和``homeComponent``。

这里用一个常量定义了四个tab的唯一标识，需要注意的是，这个常量是可以被其他组件获得的，以为它被``export``字段修饰了。

另外，还需要注意一下``HomePage``有一个属性是``selectedTab``，它用来标记当前选择的tab是哪一个。在``constructor``方法里做了一个判断，如果没有从外部组件传进来``selectedTab``，则需要初始化为``FLAG_TAB.flag_popularTab``。

## 2.5 组件间通信

既然React项目是以组件为单位搭建的，那么一定少不了组件之间的数据和事件的传递，也就是组件之间的通信。

组件间通信分为两大类：

1. 有直接关系或间接关系的组件之间通信

2. 无直接关系或间接关系的组件之间通信

 
   ​

### 2.51 有直接关系或间接关系的组件之间通信

我个人是这么理解父组件和子组件的关系的：

如果A组件包含了B组件，或者说在A组件里创建了B组件，那么A组件就是B组件的父组件；反过来B组件就是A组件的子组件，是有直接关系的组件。

比如：

- 一个界面的导航栏组件是整个页面组件的子组件，因为这个导航栏组件被包含在了当前的页面组件当中。

- 从这个页面跳转到的下一个页面是当前页面的子组件：因为被包含在了当前页面组件的``Navigator``里。

  ​

再加上子组件和子组件的通信，直接或间接关系组件之间的通信就分为下面这三种情况：

1. 父组件向子组件传递数据和事件。

2. 子组件向父组件传递消息和事件。

3. 子组件向子组件传递消息和事件。

   ​

#### 父组件向子组件传递数据和事件：通过对子组件的属性赋值来实现。

在上面我们看到，在给页面布局的时候我们使用了导航栏组件：

```javascript
<NavigationBar
      title={'我的'}
      style={this.state.theme.styles.navBar}
 />
```

在这里，当前页面组件将``'我的'``对象，以及``this.state.theme.styles.navBar``对象分别赋值给了导航栏组件。而导航栏接收到这两个值以后，在其内部可以通过``this.props.title``和``this.props.style``来获取到这两个值。这样一来，就实现了父组件向子组件传递数据的功能。

#### 子组件向父组件传递消息、数据：通过父组件给子组件一个闭包（回调函数）来实现

举一个点击最热标签页面的一个cell进行回调后实现界面跳转的例子：

既然这个cell组件是在最热标签页面组件中生成的，那么cell组件就是其子组件：

```javascript
//ListView组件生成每个cell的函数
renderRow(projectModel){
  return <RespositoryCell
            key = {projectModel.item.id}
            theme={this.state.theme}
            projectModel={projectModel}
            onSelect = {()=>this.onSelectRepository(projectModel)}
            onFavorite={(item,isFavorite)=>this.onFavorite(item,isFavorite)}/>
}
```

这个``renderRow()``函数是``ListView``组件用来渲染每一行Cell的函数，必须返回一个Cell组件才可以。在这里我们自定义了一个``RespositoryCell``组件作为其Cell组件。

我们可以看到，这里面有5个props被赋值了，其中，``onSelect``和``onFavorite``被赋予了函数：

- ``onSelect``回调的是点击cell之后在最热标签页面里跳转页面的函数``onSelectRepository()``。
- ``onFavorite``则回调的是更改最热标签页面对应收藏按钮状态的函数``onFavorite``（未被收藏时是空心的星；被收藏的话是实心的星）。

下面在``RespositoryCell``组件内部看一下这两个函数是如何回调的：

```javascript
render(){

        let item = this.props.projectModel.item?this.props.projectModel.item:this.props.projectModel;

        let favoriteButton = <TouchableOpacity
            {/*调用点击收藏的回调函数*/}
            onPress={()=>this.onPressFavorite()}
        >
            <Image
                style={[styles.favoriteImageStyle,this.props.theme.styles.tabBarSelectedIcon]}
                source={this.state.favoriteIcon}
            />
        </TouchableOpacity>

        return(
            <TouchableOpacity
                 {/*点击cell的回调函数*/}
                 onPress={this.props.onSelect}
                 style={styles.container}
            >
               <View style={styles.cellContainerViewStyle}>
                   ...
                   {favoriteButton}
               </View>
            </TouchableOpacity>)

    }
          
   onPressFavorite(){
        this.setFavoriteState(!this.state.isFavorite);
        //点击收藏的回调函数
        this.props.onFavorite(this.props.projectModel.item,!this.state.isFavorite)
   }
```

由上一节我们知道，父组件给子组件的props传值后，子组件里面对应的props就被赋值了。在这``RespositoryCell``组件里面就是``this.props.onSelect``和``this.props.onFavorite``。这两个函数被赋给了两个``TouchableOpacity``组件的``onPress``里面。这里的``()=>``可以理解为为传递事件，表示当该控件被点击后的事件。

不同的是，``this.props.onFavorite()``是可以将两个值回传给其父组件。细心的同学会发现，在给``RespositoryCell``传值的时候，是有两个返回值存在的。

> 注意，在这里的``TouchableOpacity``和上文提到的``TouchableHighlight``类似，都可以让非可点击组件变成可点击组件。区别在于配合``TouchableOpacity``使用时，点击后无高亮效果。而``TouchableHighlight``默认是有高亮效果的。

OK，现在我们知道了父组件和子组件是如何传递数据和事件了：

- 父组件到子组件：通过直接给属性赋值
- 子组件到父组件：通过父组件给子组件传递回调函数

需要注意的是，上面讲的都是直接关系的父子组件，其实还有间接关系的组件，也就是两个组件之间有一个或多个组件连接着，比如父组件的子组件的子组件。这些组件之间的通信都可以通过上述的方法来实现，只不过是中间跨过多少层的区别而已。

> 需要注意的是，这里说的父组件和子组件的通信，不仅仅包括这种直接关系，还包括间接关系，而间接关系的组件就是该组件与其子组件的子组件的关系。

所以无论中间隔了多少组件，只要是存在于这种关系链上的组件，都可以用上述两种方式来传递数据和事件。

#### 兄弟组件之间的通信

虽然不是包含于被包含，由谁创建了谁的关系，但是同一父组件下的几个子组件（兄弟组件）也算得上是有间接关系了（中间夹着共同的父组件）。

那么在同一父组件下的两个子组件是如何传递数据呢？

答案是通过二者所共享的父组件的state来传递数据的

因为我们知道触发组件的渲染是通过``setState``方法的。因此，如果两个子组件都使用了他们的父组件的同一个state来渲染自己。

那么当其中一个子组件触发了``setState``,更新了这个共享的父组件的state，继而触发了父组件的``render()``方法，那么这两个子组件都会依据这个更新后的``state``来刷新自己，这样一来，就实现了子组件的数据传递。

到现在就讲完了有直接或间接关系的组件之间的通信，下面来讲一下无直接关系或间接关系的组件之间的通信：

### 2.52 无直接关系和间接关系的组件之间通信

如果两个组件从属于不同的关系链既没有直接关系，也没有间接关系（例如不同模块下的两个页面组件），那么想实现通信的话，就需要通过通知机制，或者本地持久化方案来实现。在这里先介绍一下通知机制，而本地持久化会在下面单拿出一节来专门讲解。

通知机制可以通过这个Demo的收藏功能来讲解：

先大致介绍一下收藏的需求：

1. 在最热标签页或者语言趋势页面如果点击了收藏按钮，那么在收藏页面就会增加被收藏的项目（注意，点击收藏按钮后不进行网络请求，也就是说，收藏页面是没有网络请求的）。
2. 而如果在收藏页面中取消了收藏，就需要在最热标签页面或语言趋势页面中对应的项目里面更新取消收藏的效果（同样没有网络请求）。

因为这三个页面从属于不同模块， 而且又不是以网络请求的方式刷新列表，所以如果要满足上述需求，就需要使用通知或者本地存储的方式来实现。

在这个Demo中，第一个需求采用的是本地持久化方案，第二个需求采用的是通知机制。本地持久化方案我会在下一节单独介绍，在本节先讲一下在React Native里如何使用通知机制：

在React Native里面有专门的组件专门负责通知这一功能，它的名字是：``DeviceEventEmitter``，它是React Native内置的组件，我们可以直接将它导入到工程里。导入的方式和其他内置的组件一样：

```javascript

import React, { Component } from 'react';
import {
    StyleSheet,
    Text,
    View,
    Image,
    DeviceEventEmitter,
    TouchableOpacity
} from 'react-native';
```

既然是通知，那么自然有接收的一方，也有发送的一方，这两个组件都需要引入该通知组件。

在接收的一方需要注册某个通知：

比如在该Demo里面，如果在收藏页面修改了收藏的状态，就要给最热标签页面发送一个通知。所以首先就需要在最热标签页面注册一个通知，注册通知后才能确保将来可以收到某个频道上的通知

```javascript
componentDidMount() {
    ...
    this.listener = DeviceEventEmitter.addListener('favoriteChanged_popular',()=> {
            this.isFavoriteChanged = true;
     })
}
```

在这里通过给``DeviceEventEmitter``的``addListener``方法传入两个参数来进行通知的注册：

- 第一个参数是通知的频道，用来区别其他的通知。
- 第二个参数是需要调用的函数：在这里只是将``this.isFavoriteChanged``赋值为YES。它的目的是在于将来如果该值等于YES，就进行界面的再渲染，更新收藏状态。

需要注意的是，有注册，就要有注销，在组件被卸载之前，需要将监听解除：

```javascript
componentWillUnmount() {
     if(this.listener){
         this.listener.remove();
     }
}
```

这样，我们搞定了通知的注册，就可以在程序的任意地方发送通知了。在该需求中，我们需要拦截住在收藏页面里对项目的收藏按钮的点击，只要点击了，就发送通知：告知最热标签页面收藏的状态改变了：

```
onFavorite(item,isFavorite){

    ...
    DeviceEventEmitter.emit('favoriteChanged_popular');

}
```

在这里，拦截了收藏按钮的点击。还记得么？这里``onFavorite()``函数就是上面说的点击收藏按钮的回调。

我们在这里发送了通知，只需传入频道名称即可。

是不是很easy？

OK，到这里我们讲完了组件间的通信这一块，简单回想一下各种关系的组件之间的通信方案。

下面我们来讲一下在React Native里的本地持久化的方案。

## 2.6 本地持久化

类似于iOS 中的``NSUserDefault``， AsyncStorage 是React Native中的 Key-Value 存储系统，可以做本地持久化。

首先看它主要的几个接口：

### 2.61 AsyncStorage常用接口

#### 根据键来获取值，获取的结果会放在回调函数中：

```javascript
static getItem(key: string, callback:(error, result))
```

#### 根据键来设置值：

```javascript
static setItem(key: string, value: string, callback:(error))
```

#### 根据键来移除项：

```javascript
static removeItem(key: string, callback:(error))
```

#### 获取所有的键：

```javascript
static getAllKeys(callback:(error, keys))
```

#### 设置多项，其中 keyValuePairs 是字符串的二维数组，比如：[['k1', 'val1'], ['k2', 'val2']]：

```javascript
static multiSet(keyValuePairs, callback:(errors))
```

#### 获取多项，其中 keys 是字符串数组，比如：['k1', 'k2']：

```javascript
static multiGet(keys, callback:(errors, result))
```

#### 删除多项，其中 keys 是字符串数组，比如：['k1', 'k2']：

```javascript
static multiRemove(keys, callback:(errors))
```

#### 清除所有的项目：

```javascript
static clear(callback:(error))
```

### 2.62 AsyncStorage使用注意事项

需要注意的是，在使用AsyncStorage的时候，setItem里面传入的数组或字典等对象需要使用``JSON.stringtify()``方法把他们解析成JSON字符串：

```javascript
AsyncStorage.setItem(this.favoriteKey,JSON.stringify(favoriteKeys));
```

> 这里,favoriteKeys是一个数组。

反过来，在getItem方法里获取数组或字典等对象的时候需要使用``JSON.parse``方法将他们解析成对象：

```javascript
AsyncStorage.getItem(this.favoriteKey,(error,result)=>{
     if (!error) {
          var favoriteKeys=[];
          if (result) {
                favoriteKeys=JSON.parse(result);
          }
     ...
      }
});
```

> 这里，result被解析出来后是一个数组。

### 2.7 网络请求

在React Native中，经常使用Fetch函数来实现网络请求，它支持GET和POST请求并返回一个Promise对象，这个对象包含一个正确的结果和一个错误的结果。

来看一下用Fetch发起的POST请求：

```javascript
fetch('http://www.***.cn/v1/friendList', {
          method: 'POST',
          headers: { //header
                'token': ''
            },
          body: JSON.stringify({ //参数
                'start': '0',
                'limit': '20',
            })
 })
            .then((response) => response.json()) //把response转为json
            .then((responseData) => { // 上面的转好的json
                 //using responseData
            })
            .catch((error)=> {
                alert('返回错误');
            })
```

从上面的代码中，我们可以大致看到：Fetch函数中，第一个参数是请求url，第二个参数是一个字典，包括方法，请求头，请求体等信息。

随后的``then``和``catch``分别捕捉了fetch函数的返回值：一个Promise对象的``正确结果``和``错误结果``。注意，这里面有两个``then``，其中第二个``then``把第一个``then``的结果拿了过来。而第一个``then``做的事情是把网络请求的结果转化为JSON对象。

那么什么是Promise对象呢？

Promise 是异步编程的一种解决方案，Promise对象可以获取某个异步操作的消息。它里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。

它分为三种状态：

`Pending`（进行中）、`Resolved`（已成功）和`Rejected`（已失败）

它的构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`：

`resolve`函数的作用：将Promise对象的状态从“未完成”变成“成功”(即从Pending变为Resolved)，在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；。
`reject`函数的作用：将Promise对象的状态从“未完成”变成“成功”(即从Pending变为Rejected)，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

举个例子来看一下：

```javascript
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

这里resolve和reject的结果会分别被配套使用的Fetch函数的.then和.catch捕捉。

我个人的理解是：如果某个异步操作的返回值是一个Promise对象，那么我们就可以分别使用``.then``和``.catch``来捕捉正确和错误的结果。

再看一下GET请求：

```javascript
fetch(url)
    .then(response=>response.json())
    .then(result=>{
         resolve(result);
     })

     .catch(error=>{
         reject(error)
     })
```

因为只是GET请求，所以不需要配置请求体，而且因为这个fetch函数返回值是一个Promise对象， 所以我们可以用``.then``和``.catch``来捕捉正确和错误的结果。

在项目中，我们可以创建一个抓们负责网络请求的工具HttpUtils类，封装GET和POST请求。看一下一个简单的封装：

```javascript

export default class HttpUtls{
    
    static get(url){
        return new Promise((resolve,reject)=>{
            fetch(url)
                .then(response=>response.json())
                .then(result=>{
                    resolve(result);
                })

                .catch(error=>{
                    reject(error)
                })
        })
    }

    static post(url, data) {
        return new Promise((resolve, reject)=>{
            fetch(url,{
                method:'POST',
                header:{
                    'Accept':'application/json',
                    'Content-Type':'application/json',
                },
                body:JSON.stringify(data)
            })

                .then(result=>{
                    resolve(result);
                })

                .catch(error=>{
                    reject(error)
                })
        })
    }
}
```

### 2.8 离线缓存

离线缓存技术可以利用上文提到的``Fetch``和``AsyncStorage``实现，将请求url作为key，将返回的结果作为值存入本地数据里。

在下一次请求之前查询是否有缓存，缓存是否过期，如果有缓存并且没有过期，则拿到缓存之后，立即返回进行处理。否则继续进行网络请求。

而且即使没有网络，最终返回错误，也可以拿到缓存数据，立即返回。

来看一下在该项目里面是如何实现离线缓存的:

```javascript
//获取数据
    fetchRespository(url) {

        return new Promise((resolve, reject) =>{

            //首先获取本地缓存
            this.fetchLocalRespository(url)
                .then((wrapData)=> {
                    //本地缓存获取成功
                if (wrapData) {
                    //缓存对象存在
                    resolve(wrapData,true);
                } else {
                    //缓存对象不存在，进行网络请求
                    this.fetchNetRepository(url)

                        //网路请求成功
                        .then((data) => {
                            resolve(data);
                        })
                        //网路请求失败
                        .catch(e=> {
                            reject(e);
                        })
                }
            }).catch(e=> {
                    //本地缓存获取失败，进行网络请求
                    this.fetchNetRepository(url)

                        //网路请求成功
                        .then(result => {
                            resolve(result);
                        })
                        //网路请求失败
                        .catch(e=> {
                            reject(e);
                        })
                })
        })
    }
```

在上面的方法中，包含了获取本地缓存和网络请求的两个方法。

首先是尝试获取本地缓存：

```javascript
    //获取本地缓存
    fetchLocalRespository(url){
        return new Promise((resolve,reject)=>{
            // 获取本地存储
            AsyncStorage.getItem(url, (error, result)=>{
                if (!error){
                    try {
                        //必须使用parse解析成对象
                        resolve(JSON.parse(result));
                    }catch (e){
                        //解析失败
                        reject(e);
                    }
                }else {
                    //获取缓存失败
                    reject(error);
                }
            })
        })
    }
```

> 在这里，``AsyncStorage.getItem``方法的结果也可以使用Promise对象来包装。因此，``this.fetchLocalRespository(url)``的结果也就可以被``.then``和``.catch``捕捉到了。

如果获取本地缓存失败，就会调用网络请求：

```javascript
    fetchNetRepository(url){
        return new  Promise((resolve,reject)=>{
            fetch(url)
                .then(response=>response.json())
                .catch((error)=>{
                    reject(error);
                }).then((responseData)=>{
                    resolve(responseData);
                 })
             })
    }
```

### 2.9 主题更换

这个Demo有一个主题更换的需求，在主题设置页点击某个颜色之后，全app的颜色方案就会改变：

![](http://oih3a9o4n.bkt.clouddn.com/rn_14.png)

我们只需要将四个模块的第一个页面的主题修改即可，因为第二个页面的主题都是从第一个页面传进去的，所以只要第一个页面的主题改变了即可。

但是，我们应该不能在选择新主题之后同时向这四个页面都发送通知，命令它们修改自己的页面，而是应该采取一个更加优雅的方法来解决这个问题：使用父类。

新建一个``BaseCommon.js``页面，作为这四个页面的父类。在这个父类里面接收主题更改的通知，并更新自己的主题。这样一来，继承它的这四个页面就都会刷新自己：

来看一下这个父类的定义：

```javascript
import React, { Component } from 'react';
import {
    DeviceEventEmitter
} from 'react-native';

import {ACTION_HOME} from '../pages/Entry/HomePage'

export default class BaseComponent extends Component {

    constructor(props){
        super(props);
        this.state={
            theme:this.props.theme,
        }
    }

    componentDidMount() {
        this.baseListener = DeviceEventEmitter.addListener('ACTION_BASE',(action,parmas)=>this.changeThemeAction(action,parmas));
    }

    //卸载前移除通知
    componentWillUnmount() {
        if(this.baseListener){
            this.baseListener.remove();
        }
    }
    
    //接收通知
    changeThemeAction(action,params){
        if (ACTION_HOME.A_THEME === action){
            this.onThemeChange(params);
        }
    }

    //更新theme
    onThemeChange(theme){
        if(!theme)return;
        this.setState({
            theme:theme
        })
    }

}
```

在更新主题页面的更新主题事件：

```javascript
onSelectTheme(themeKey) {

        this.themeDao.save(ThemeFlags[themeKey]);
        this.props.onClose();
        DeviceEventEmitter.emit('ACTION_BASE',ACTION_HOME.A_THEME,ThemeFactory.createTheme(
            ThemeFlags[themeKey]
        ))
    }
```

### 2.10 功能调试

我们可以使用浏览器的开发者工具来调试React Native项目，可以通过打断点的方式来看数据信息以及方法的调用：

1. 首先在iOS模拟器中点击``command + D``，然后再弹出菜单里点击``Debug JS Remotely``。随后就打开了浏览器进入了调试。

![](http://oih3a9o4n.bkt.clouddn.com/rn_8_1_1.png)

2. 浏览器一般会展示下面的页面，然后点击``command + option + J``进入真生的调试界面。

![](http://oih3a9o4n.bkt.clouddn.com/rn_9.png)

3. 点击最上方的``Sources``，然后点击左侧``debuggerWorker.js``下的``localhost:8081``，就可以看到目录文件。点击需要调试的文件，在行数栏就可以打断点了。

![](http://oih3a9o4n.bkt.clouddn.com/rn_10_1.png)

### 2.11 适配iOS和Android平台

因为React Native讲求的是一份代码跑在两个平台上，而客观上这两个平台又有一些不一样的地方，所以就需要在别要的时候做一下两个平台的适配。

例如导航栏：在iOS设备中是存在导航栏的，而安卓设备上是没有的。所以在定制导航栏的时候，在不同平台下给导航栏设置不同的高度：

```javascript
import {
    StyleSheet,
    Platform,
} from 'react-native'

const NAV_BAR_HEIGHT_IOS = 44;
const NAV_BAR_HEIGHT_ANDROID = 50;

navBarStyle: {
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        height: Platform.OS === 'ios' ? NAV_BAR_HEIGHT_IOS : NAV_BAR_HEIGHT_ANDROID,
 },
      
```

上面的``Platform``是React Native内置的用于区分平台的库，可以在引入后直接使用。

建议在调试程序的时候，同时打开iOS和Android的模拟器进行调试，因为有些地方可能在某个平台上是没问题的，但是另一个平台上有问题，这就需要使用``Platform``来区分平台。

### 2.12 组织项目结构

在终端输入``react-native demo --version 0.44.0``命令以后，就会初始化一个React Native版本为0.44.0的项目。这个最初项目里面直接就包含了iOS和Android的工程文件夹，可以用对应的IDE打开后编译运行。

在新建一个React Native项目之后的根目录结构是这样的：

![](http://oih3a9o4n.bkt.clouddn.com/rn_11.png)



或者也可以根目录下输入``react-native run-ios ``或者`` react-native run-android ``指令， 就会自动打开模拟器运行项目(前提是安装了相应的开发环境)。

但是一个比较完整的项目仅仅有这些类别的文件是不够的，还需要一些工具类，模型类，资源等文件。为了很好地区分它们，使项目结构一目了然，需要组织好项目文件夹以及类的命名，下面是我将教程里的文件夹命名和结构稍加修改后的一个方案，可供大家参考：

![](http://oih3a9o4n.bkt.clouddn.com/rn_12.png)


  ​

# 三. 总结

从最开始的FlexBox布局的学习到现在这个项目的总结完成有了快两个月的时间了。我在这里说一下这段学习过程中的一些感受：

### 关于学习成本

我觉得这一点应该是所有未接触到React Native的人最关心的一点了，所以我将它放到了总结里的第一位。我在这里取两种典型的群体来做比较：

1. 只会某种Native开发但是不会JavaScript等前端知识的人群。
2. 只会前端知识但是不会任何一种Native开发的人群。

对于这两种人群来说，在React Native的学习过程中成本都不小。但不同的是，这两种人群的学习成本在整个学习过程中的不同阶段是不一样的。怎么说呢？

对于第一种人群，因为缺乏前端相关知识，所以在组建的布局，以及JavaScript的语法上会有点吃力。而这两点恰恰是React Native学习的敲门砖，因此，对于这种群体，在学习React Native的初期会比较吃力，学习成本很大。

### 关于如何配合视频来学习

在结合视频学习的时候一定要跟上思路，如果讲师是边写代码边讲解，就一定要弄清楚每一行代码的意义在哪里，为什么要这么写，千万不要怕浪费时间而快速略过。停下脚步来思考实际上是节省时间：因为如果你不试着去理解代码和讲师的思路，在后来你会越来越看不懂，反而浪费大量时间重新回头看。

所以我认为最好是先听一遍讲师讲的内容，理清思路，然后再动手写代码，这样效率会比较高，在将来出现的问题也会更少。

# 四. 学习参考资料

下面是我近1个半月以来收集的比较好的React Native入门资料和博客，分享给大家：

- [React Native中文网](http://reactnative.cn/)
- [贾鹏辉的技术博客](http://www.devio.org/)
- [Marno:给所有开发者的React Native详细入门指南](http://www.jianshu.com/p/fa0874be0827)
- [大漠:一个完整的Flexbox指南](http://www.w3cplus.com/css3/a-guide-to-flexbox.html)
- [阮一峰:Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [八段代码彻底掌握 Promise](https://juejin.im/post/597724c26fb9a06bb75260e8)
- [阮一峰：Promise对象](http://es6.ruanyifeng.com/#docs/promise)
- [asce1885:React Native 高质量学习资料汇总](http://www.jianshu.com/p/454f2e6f28e9#rd)
- [世锋日上:ReactNative 学习资源大汇集](https://juejin.im/post/591ec246da2f60005d30654c)



