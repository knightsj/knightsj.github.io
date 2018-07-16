
---
title: 基于MVVM，用于快速搭建设置页，个人信息页的框架
tags: [iOS,Objective-C]
categories: Production
---

### 更新记录：

**2017.4.23：新增支持数据源完全依赖网络请求的情况。**
**2017.4.22：新增支持请求新数据后刷新表格。**
**2017.4.21： 新增CocoaPods支持：pod 'SJStaticTableView', '~> 1.2.0'。**

---
写一个小小轮子～

写UITableView的时候，我们经常遇到的是完全依赖于网络请求，需要自定义的动态cell的需求（比如微博帖子列表）。但是同时，大多数app里面几乎也都有设置页，个人页等其他以静态表格为主的页面。

而且这些页面的共性比较多：
1. 大多数情况下在进入页面之前就已经拿到所有数据。
2. cell样式单一，自定义cell出现的几率比较小（几乎都是高度为44的cell）。
3. 多数都分组。

因为自己非常想写一个开源的东西出来（也可以暴露自己的不足），同时又受限于水平，所以就打算写这么一个比较简单，又具有通用性的框架：**一个定制性比较高的适合在个人页和设置页使用的UITableView**。

在真正写之前，看了几篇类似的文章，挑出三篇自己觉得比较好的：
1. [Clean Table View Code](https://www.objc.io/issues/1-view-controllers/table-views/) 
2. [如何写好一个UITableView](http://www.jianshu.com/p/504c61a9dc82)
3. [利用MVVM设计快速开发个人中心、设置等模块](http://www.jianshu.com/p/81d0c573f7a8)

看完总结之后，利用上周3天的业余时间写好了这个框架，为了它实用性，我仿照了微信客户端的发现页，个人页和设置页写了一个Demo，来看一下效果图：

![发现页 | 个人页 | 个人信息页 | 设置页](https://user-gold-cdn.xitu.io/2017/4/24/85789de80b62c93ff0c9f77b603dba70)

>项目所用资源来自：[GitHub:zhengwenming/WeChat](https://github.com/zhengwenming/WeChat)
>Demo地址：[GitHub: knightsj/SJStaticTableView](https://github.com/knightsj/SJStaticTableView)

<!-- more -->

为了体现出这个框架的定制性，我自己也在里面添加了两个页面，入口在设置页里面：

![分组定制 | 同组定制](https://user-gold-cdn.xitu.io/2017/4/24/a38f9f8459fbd168e3f434e5cff27526)

>先不要纠结分组定制和同组定制的具体意思，在后面讲到定制性的时候我会详细说明。现在只是让大家看一下效果。

在大概了解了功能之后，开始详细介绍这个框架。写这篇介绍的原因倒不是希望有多少人来用，而是表达一下我自己的思路而已。各位觉得不好的地方请多批评。

在正式讲解之前，先介绍一下本篇的基本目录：
1. 用到的技术点。
2. 功能说明。
3. 使用方法。
4. 定制性介绍。
5. 新增支持刷新功能。
6. 新增支持数据源完全依赖网络请求。

# 1. 用到的技术点
------

框架整体来说还是比较简单的，主要还是基于苹果的``UITableView``组件，为了解耦和责任分离，主要运用了以下技术点：
- **MVVM**：采用MVVM架构，将每一行“纯粹”的数据交给一个单独的``ViewModel``,让其持有每个cell的数据（行高，cell类型，文本宽度，图片高度等等）。而且每一个section也对应一个``ViewModel``，它持有当前section的配置数据（title，header和footer的高度等等）。
- **轻UIViewController**：分离``UITableViewDataSource``与``UIViewController``，让单独一个类来实现``UITableViewDataSource``的职能。
- **block**：使用block来调用cell的绘制方法。
- **分类**：使用分类来定义每一种不同的cell的绘制方法。

知道了主要运用的技术点以后，给大家详细介绍一下该框架的功能。

# 2. 功能介绍
----


这个框架可以用来快速搭建设置页，个人信息页能静态表格页面，使用者只需要给tableView的DataSource传入元素是viewModel的数组就可以了。

虽说这类页面的布局还是比较单一的，但是还是会有几种不同的情况（cell的布局类型），我对比较常见的cell布局做了封装，使用者可以直接使用。

我在定义这些cell的类型的时候，大致划分了两类：
1. 第一类是系统风格的cell，大多数情况下，cell高度为44；在cell左侧会有一张图，一个label，也可以只存在一种（但是只存在图片的情况很少）；在cell右侧一般都有一个向右的箭头，而且有时这个箭头的左侧还可能有label，image，也可以两个都有。
2. 第二类就是自定义的cell了，它的高度不一定是44，而且布局和系统风格的cell很不一样，需要用户自己添加。

基于这两大类，再细分了几种情况，可以由下面这张图来直观看一下：

既然是cell的类型，那么就类型的枚举就需要定义在cell的viewModel里面：
```objc
typedef NS_ENUM(NSInteger, SJStaticCellType) {
    
    //系统风格的各种cell类型，已封装好，可以直接用
    SJStaticCellTypeSystemLogout,                          //退出登录cell
    SJStaticCellTypeSystemAccessoryNone,                   //右侧没有任何控件
    SJStaticCellTypeSystemAccessorySwitch,                 //右侧是开关
    SJStaticCellTypeSystemAccessoryDisclosureIndicator,    //右侧是三角箭头(箭头左侧可以有一个image或者一个label，或者二者都有，根据传入的参数决定)
    
    //需要用户自己添加的自定义cell类型
    SJStaticCellTypeMeAvatar,                              //个人页“我”cell    
};
```

来一张图直观得体会一下：

![支持cell类型](https://user-gold-cdn.xitu.io/2017/4/24/1f8db817518311481d64d30ec481cf71)

在这里有三点需要说一下：

1. 这里面除了自定义的cell以外，其他类型的cell都不需要开发者自己布局，都已经被我封装好，只需要在cell的``ViewModel``里面传入相应的类型和数据（文字，图片）即可。
2. 因为左侧的两个控件（图片和文字）是至少存在一个而且左右顺序固定（图片永远在最左侧），所以该框架通过开发者传入的左侧需要显示的图片和文字，可以自己进行cell的布局。所以类型的判断主要作用于cell的右侧。
3. 值得一提的是，在"最右侧是一个箭头"子分支的五个类型其实都属于一个类型，只需要传入文字和图片，以及文字图片的显示顺序参数（这个参数只在同时存在图片和文字的时候有效）就可以自行判断布局。


在了解了该框架的功能之后，我们先看一下如何使用这个框架：


# 3. 使用方法
----

## 集成方法：
1. 静态：手动将SJStaticTableViewComponent文件夹拖入到工程中。
2. 动态：CocoaPods：``pod 'SJStaticTableView', '~> 1.1.2``。


具体的方法先用文字说明一下：

2. 将要开发的页面的ViewController继承``SJStaticTableViewController``。
3. 在新ViewController里实现``createDataSource``方法，将viewModel数组传给控制器的``dataSource``属性。
4. 根据不同的cell类型，调用不同的cell绘制方法。
5. 如果需要接受cell的点击，需要实现``didSelectViewModel``方法。


可能感觉比较抽象，我拿设置页来具体说明一下：

先看一下设置页的布局：

![设置页](https://user-gold-cdn.xitu.io/2017/4/24/d1ea6240a3760acbcb92198f1c454044)

然后我们看一下设置的ViewController的代码：
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    self.navigationItem.title = @"设置";
}


- (void)createDataSource
{
    self.dataSource = [[SJStaticTableViewDataSource alloc] initWithViewModelsArray:[Factory settingPageData] configureBlock:^(SJStaticTableViewCell *cell, SJStaticTableviewCellViewModel *viewModel) {
        
        switch (viewModel.staticCellType)
        {
            case SJStaticCellTypeSystemAccessoryDisclosureIndicator:
            {
                [cell configureAccessoryDisclosureIndicatorCellWithViewModel:viewModel];
            }
                break;
                
            case SJStaticCellTypeSystemAccessorySwitch:
            {
                [cell configureAccessorySwitchCellWithViewModel:viewModel];
            }
                break;
                
            case SJStaticCellTypeSystemLogout:
            {
                [cell configureLogoutTableViewCellWithViewModel:viewModel];
            }
                break;
                
            case SJStaticCellTypeSystemAccessoryNone:
            {
                [cell configureAccessoryNoneCellWithViewModel:viewModel];
            }
                break;
                
            default:
                break;
        }
    }];
}


- (void)didSelectViewModel:(SJStaticTableviewCellViewModel *)viewModel atIndexPath:(NSIndexPath *)indexPath
{
    
    switch (viewModel.identifier)
    {
            
        case 6:
        {
            NSLog(@"退出登录");
            [self showAlertWithMessage:@"真的要退出登录嘛？"];
        }
            break;
            
        case 8:
        {
            NSLog(@"清理缓存");
        }
            break;
            
        case 9:
        {
            NSLog(@"跳转到定制性cell展示页面 - 分组");
            SJCustomCellsViewController *vc = [[SJCustomCellsViewController alloc] init];
            [self.navigationController pushViewController:vc animated:YES];
        }
            break;
            
        case 10:
        {
            NSLog(@"跳转到定制性cell展示页面 - 同组");
            SJCustomCellsOneSectionViewController *vc = [[SJCustomCellsOneSectionViewController alloc] init];
            [self.navigationController pushViewController:vc animated:YES];
        }
            break;
            
        default:
            break;
    }
}
```

看到这里，你可能会有这些疑问：
1. UITableViewDataSource方法哪儿去了？
2. viewModel数组是如何设置的？
3. cell的绘制方法是如何区分的？
4. UITableViewDelegate的方法哪里去了？


下面我会一一解答，看完了下面的解答，就能几乎完全掌握这个框架的思路了：

###  问题1：UITableViewDataSource方法哪儿去了？

我自己封装了一个类``SJStaticTableViewDataSource``专门作为数据源，需要控制器给它一个viewModel数组。

来看一下它的实现文件：
```objc
//SJStaticTableViewDataSource.m
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return self.viewModelsArray.count;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    SJStaticTableviewSectionViewModel *vm = self.viewModelsArray[section];
    return vm.cellViewModelsArray.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    //获取section的ViewModel
    SJStaticTableviewSectionViewModel *sectionViewModel = self.viewModelsArray[indexPath.section];
    //获取cell的viewModel
    SJStaticTableviewCellViewModel *cellViewModel = sectionViewModel.cellViewModelsArray[indexPath.row];
    
    SJStaticTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellViewModel.cellID];
    if (!cell) {
        cell = [[SJStaticTableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellViewModel.cellID];
    }
    self.cellConfigureBlock(cell,cellViewModel);
    
    return cell;
    
}

- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
    SJStaticTableviewSectionViewModel *vm = self.viewModelsArray[section];
    return vm.sectionHeaderTitle;  
}

- (NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section
{
    SJStaticTableviewSectionViewModel *vm = self.viewModelsArray[section];
    return vm.sectionFooterTitle;
}
```

>表格的cell和section都设置了与其对应的viewModel，用于封装其对应的数据：

cell的viewModel(大致看一下即可，后面有详细说明)：
```objc
typedef NS_ENUM(NSInteger, SJStaticCellType) {
    
    //系统风格的各种cell类型，已封装好，可以直接用
    SJStaticCellTypeSystemLogout,                          //退出登录cell（已封装好）
    SJStaticCellTypeSystemAccessoryNone,                   //右侧没有任何控件
    SJStaticCellTypeSystemAccessorySwitch,                 //右侧是开关
    SJStaticCellTypeSystemAccessoryDisclosureIndicator,    //右侧是三角箭头(箭头左侧可以有一个image或者一个label，或者二者都有，根据传入的参数决定)
    
    //需要用户自己添加的自定义cell类型
    SJStaticCellTypeMeAvatar,                              //个人页“我”cell
    
};


typedef void(^SwitchValueChagedBlock)(BOOL isOn);           //switch开关切换时调用的block


@interface SJStaticTableviewCellViewModel : NSObject

@property (nonatomic, assign) SJStaticCellType staticCellType;                  //类型


@property (nonatomic, copy)   NSString *cellID;                                  //cell reuser identifier
@property (nonatomic, assign) NSInteger identifier;                              //区别每个cell，用于点击

// =============== 系统默认cell左侧 =============== //
@property (nonatomic, strong) UIImage  *leftImage;                               //左侧的image，按需传入
@property (nonatomic, assign) CGSize leftImageSize;                              //左侧image的大小，存在默认设置

@property (nonatomic, copy)   NSString *leftTitle;                               //cell主标题，按需传入
@property (nonatomic, strong) UIColor *leftLabelTextColor;                       //当前组cell左侧label里文字的颜色
@property (nonatomic, strong) UIFont *leftLabelTextFont;                         //当前组cell左侧label里文字的字体

@property (nonatomic, assign) CGFloat leftImageAndLabelGap;                      //左侧image和label的距离，存在默认值


// =============== 系统默认cell右侧 =============== //
@property (nonatomic, copy)   NSString *indicatorLeftTitle;                      //右侧箭头左侧的文本，按需传入
@property (nonatomic, strong) UIColor *indicatorLeftLabelTextColor;              //右侧文字的颜色，存在默认设置，也可以自定义
@property (nonatomic, strong) UIFont *indicatorLeftLabelTextFont;                //右侧文字的字体，存在默认设置，也可以自定义
@property (nonatomic, strong) UIImage *indicatorLeftImage;                       //右侧箭头左侧的image，按需传入
@property (nonatomic, assign) CGSize indicatorLeftImageSize;                     //右侧尖头左侧image大小，存在默认设置，也可以自定义

@property (nonatomic, assign, readonly)  BOOL hasIndicatorImageAndLabel;         //右侧尖头左侧的文本和image是否同时存在，只能通过内部计算

@property (nonatomic, assign) CGFloat indicatorLeftImageAndLabelGap;             //右侧尖头左侧image和label的距离，存在默认值
@property (nonatomic, assign) BOOL isImageFirst;                                 //右侧尖头左侧的文本和image同时存在时，是否是image挨着箭头，默认为YES
@property (nonatomic, copy) SwitchValueChagedBlock switchValueDidChangeBlock;    //切换switch开关的时候调用的block


// =============== 长宽数据 =============== //
@property (nonatomic, assign) CGFloat cellHeight;                                //cell高度,默认是44，可以设置
@property (nonatomic, assign) CGSize  leftTitleLabelSize;                        //左侧默认Label的size，传入text以后内部计算
@property (nonatomic, assign) CGSize  indicatorLeftLabelSize;                    //右侧label的size


// =============== 自定义cell的数据放在这里 =============== //
@property (nonatomic, strong) UIImage *avatarImage;
@property (nonatomic, strong) UIImage *codeImage;
@property (nonatomic, copy)   NSString *userName;
@property (nonatomic, copy)   NSString *userID;
```

section的viewModel(大致看一下即可，后面有详细说明)：
```objc
@interface SJStaticTableviewSectionViewModel : NSObject

@property (nonatomic, copy)   NSString *sectionHeaderTitle;         //该section的标题
@property (nonatomic, copy)   NSString *sectionFooterTitle;         //该section的标题
@property (nonatomic, strong) NSArray  *cellViewModelsArray;        //该section的数据源

@property (nonatomic, assign) CGFloat  sectionHeaderHeight;         //header的高度
@property (nonatomic, assign) CGFloat  sectionFooterHeight;         //footer的高度

@property (nonatomic, assign) CGSize leftImageSize;                 //当前组cell左侧image的大小
@property (nonatomic, strong) UIColor *leftLabelTextColor;          //当前组cell左侧label里文字的颜色
@property (nonatomic, strong) UIFont *leftLabelTextFont;            //当前组cell左侧label里文字的字体
@property (nonatomic, assign) CGFloat leftImageAndLabelGap;         //当前组左侧image和label的距离，存在默认值

@property (nonatomic, strong) UIColor *indicatorLeftLabelTextColor; //当前组cell右侧label里文字的颜色
@property (nonatomic, strong) UIFont *indicatorLeftLabelTextFont;   //当前组cell右侧label里文字的字体
@property (nonatomic, assign) CGSize indicatorLeftImageSize;        //当前组cell右侧image的大小
@property (nonatomic, assign) CGFloat indicatorLeftImageAndLabelGap;//当前组cell右侧image和label的距离，存在默认值


- (instancetype)initWithCellViewModelsArray:(NSArray *)cellViewModelsArray;
```

>你可能会觉得属性太多了，但这些属性的存在意义是为cell的定制性服务的，在后文会有解释。

现在了解了我封装好的数据源，cell的viewModel，section的viewModel以后，我们看一下第二个问题：

## 问题2： viewModel数组是如何设置的？

我们来看一下设置页的viewModel数组的设置：
```objc
+ (NSArray *)settingPageData
{
    // ========== section 0
    SJStaticTableviewCellViewModel *vm0 = [[SJStaticTableviewCellViewModel alloc] init];
    vm0.leftTitle = @"账号与安全";
    vm0.identifier = 0;
    vm0.indicatorLeftTitle = @"已保护";
    vm0.indicatorLeftImage = [UIImage imageNamed:@"ProfileLockOn"];
    vm0.isImageFirst = NO;
    
    SJStaticTableviewSectionViewModel *section0 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm0]];
    
    

    // ========== section 1
    SJStaticTableviewCellViewModel *vm1 = [[SJStaticTableviewCellViewModel alloc] init];
    vm1.leftTitle = @"新消息通知";
    vm1.identifier = 1;
    
    //额外添加switch
    SJStaticTableviewCellViewModel *vm7 = [[SJStaticTableviewCellViewModel alloc] init];
    vm7.leftTitle = @"夜间模式";
    vm7.switchValueDidChangeBlock = ^(BOOL isON){
        NSString *message = isON?@"打开夜间模式":@"关闭夜间模式";
        NSLog(@"%@",message);
    };
    vm7.staticCellType = SJStaticCellTypeSystemAccessorySwitch;
    vm7.identifier = 7;
    
    SJStaticTableviewCellViewModel *vm8 = [[SJStaticTableviewCellViewModel alloc] init];
    vm8.leftTitle = @"清理缓存";
    vm8.indicatorLeftTitle = @"12.3M";
    vm8.identifier = 8;
    
    SJStaticTableviewCellViewModel *vm2 = [[SJStaticTableviewCellViewModel alloc] init];
    vm2.leftTitle = @"隐私";
    vm2.identifier = 2;
    
    
    SJStaticTableviewCellViewModel *vm3 = [[SJStaticTableviewCellViewModel alloc] init];
    vm3.leftTitle = @"通用";
    vm3.identifier = 3;
    
    SJStaticTableviewSectionViewModel *section1 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm1,vm7,vm8,vm2,vm3]];
    



    // ========== section 2
    SJStaticTableviewCellViewModel *vm4 = [[SJStaticTableviewCellViewModel alloc] init];
    vm4.leftTitle = @"帮助与反馈";
    vm4.identifier = 4;
    
    SJStaticTableviewCellViewModel *vm5 = [[SJStaticTableviewCellViewModel alloc] init];
    vm5.leftTitle = @"关于微信";
    vm5.identifier = 5;
    
    SJStaticTableviewSectionViewModel *section2 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm4,vm5]];
    


      // ========== section 4
    SJStaticTableviewCellViewModel *vm9 = [[SJStaticTableviewCellViewModel alloc] init];
    vm9.leftTitle = @"定制性cell展示页面 - 分组";
    vm9.identifier = 9;
    
    SJStaticTableviewCellViewModel *vm10 = [[SJStaticTableviewCellViewModel alloc] init];
    vm10.leftTitle = @"定制性cell展示页面 - 同组";
    vm10.identifier = 10;
    
    SJStaticTableviewSectionViewModel *section4 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm9,vm10]];
    
    

    // ========== section 3
    SJStaticTableviewCellViewModel *vm6 = [[SJStaticTableviewCellViewModel alloc] init];
    vm6.staticCellType = SJStaticCellTypeSystemLogout;
    vm6.cellID = @"logout";
    vm6.identifier = 6;
   
    SJStaticTableviewSectionViewModel *section3 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm6]];
    
    return @[section0,section1,section2,section4,section3];
}
```

我们可以看到，交给dataSource的数组是一个二维数组：
- 第一维是section数组，元素是每一个section对应的viewModel：``SJStaticTableviewSectionViewModel``。
- 第二维是cell数组，元素是每一个cell对应的viewModel:``SJStaticTableviewCellViewModel``。

有几个``SJStaticTableviewCellViewModel``的属性需要强调一下：
1. isImageFirst：因为该页面第一组的cell右侧的箭头左边同时存在一个image和一个label，所以需要额外设置二者的顺序。因为默认紧挨着箭头的是图片，所以我们需要重新设置它为NO，作用是让label紧挨着箭头。
2. identifier：这个属性是一个整数，它用来标记每个cell，用于在用户点击cell的时候进行判断。我没有将用户的点击与cell的index相关联，是因为有的时候因为需求我们可能会更改cell的顺序或者删除某个cell，所以依赖cell的index是不妥的，容易出错。
3. cellID：这个属性用来cell的复用。因为总是有个别cell的布局是不同的：在这里出现了一个退出登录的cell，所以需要和其他的cell区别开来（cellID可以不用设置，有默认值，用来标记最常用的cell类型）。

显然，``Factory``类属于``Model``，它将“纯数据”交给了dataSource使用的两个viewModel。这个类是我自己定义的，读者在使用这个框架的时候可以根据需求自己定义。

现在知道了数据源的设置方法，我们看一下第三个问题：

### 问题3：cell的绘制方法是如何区分的？

心细的同学会发现，在dataSource的``cellForRow:``方法里，我用了block方法来绘制了cell。

先看一下这个block的定义：
```objc
typedef void(^SJStaticCellConfigureBlock)(SJStaticTableViewCell *cell, SJStaticTableviewCellViewModel * viewModel);
```

这个block在控制器里面回调，通过判断cell的类型来绘制不同的cell。

那么不同类型的cell是如何区分的呢？
--- 我用的是分类。

有分类，就一定有一个被分类的类： ``SJStaticTableViewCell``

看一下它的头文件：
```objc
//所有cell都是这个类的分类

@interface SJStaticTableViewCell : UITableViewCell

@property (nonatomic, strong) SJStaticTableviewCellViewModel *viewModel;

// =============== 系统风格cell的所有控件 =============== //

//左半部分
@property (nonatomic, strong) UIImageView *leftImageView;               //左侧的ImageView
@property (nonatomic, strong) UILabel *leftTitleLabel;                  //左侧的Label

//右半部分
@property (nonatomic, strong) UIImageView *indicatorArrow;              //右侧的箭头
@property (nonatomic, strong) UIImageView *indicatorLeftImageView;      //右侧的箭头的左边的imageview
@property (nonatomic, strong) UILabel *indicatorLeftLabel;              //右侧的箭头的左边的Label
@property (nonatomic, strong) UISwitch *indicatorSwitch;                //右侧的箭头的左边的开关
@property (nonatomic, strong) UILabel *logoutLabel;                     //退出登录的label

// =============== 用户自定义的cell里面的控件 =============== //

//MeViewController里面的头像cell
@property (nonatomic, strong) UIImageView *avatarImageView;
@property (nonatomic, strong) UIImageView *codeImageView;
@property (nonatomic, strong) UIImageView *avatarIndicatorImageView;
@property (nonatomic, strong) UILabel *userNameLabel;
@property (nonatomic, strong) UILabel *userIdLabel;


//统一的，布局cell左侧部分的内容（标题 ／ 图片 + 标题），所有系统风格的cell都要调用这个方法
- (void)layoutLeftPartSubViewsWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;

@end
```

在这里我定义了所有的控件和一个布局cell左侧的控件的方法。因为几乎所有的分类的左侧几乎都是类似的，所以将它抽取出来。

那么究竟有几个分类呢？（可以参考上面cellViewModel头文件里的枚举类型）

```objc
//右侧有剪头的cell（最常见）
@interface SJStaticTableViewCell (AccessoryDisclosureIndicator)
- (void)configureAccessoryDisclosureIndicatorCellWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;
@end
```

```objc
//右侧没有控件的cell
@interface SJStaticTableViewCell (AccessoryNone)
- (void)configureAccessoryNoneCellWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;
@end
```

```objc
//右侧是开关的 cell
@interface SJStaticTableViewCell (AccessorySwitch)
- (void)configureAccessorySwitchCellWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;
@end
```

```objc
//退出登录cell
@interface SJStaticTableViewCell (Logout)
- (void)configureLogoutTableViewCellWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;
@end
```


```objc
//一个自定义的cell（在个人页的第一排）
@interface SJStaticTableViewCell (MeAvatar)
- (void)configureMeAvatarTableViewCellWithViewModel:(SJStaticTableviewCellViewModel *)viewModel;
@end
```

在使用这个框架的时候，如果遇到不满足当前需求的情况，可以自己添加分类。

### 问题4：UITableViewDelegate的方法哪里去了？

说到``UITableViewDelegate``的代理方法，我们最熟悉的莫过于``didSelectRowAtIndexPath:``了。

但是我在写这个框架的时候，自己定义了一个继承于``UITableViewDelegate``的代理：``SJStaticTableViewDelegate``，并给它添加了一个代理方法：
``
```objc
@protocol SJStaticTableViewDelegate <UITableViewDelegate>

@optional

- (void)didSelectViewModel: (SJStaticTableviewCellViewModel *)viewModel atIndexPath:(NSIndexPath *)indexPath;

@end
```

这个方法返回的是当前点击的cell对应的viewModel，弱化了indexPath的作用。

为什么要这么做？

想一想原来点击cell的代理方法：``didSelectRowAtIndexPath:``。我们通过这个点击方法，拿到的是cell对应的indexPath，然后再通过这个indexPath，就可以在数据源里面查找对应的模型（viewModel或者model）。

因此，我定义的这个方法直接返回了被点击cell对应的viewModel，等于说帮使用者节省了一个步骤。当然如果要使用的话也可以使用系统原来的``didSelectRowAtIndexPath:``方法。

来看一下这个新的代理方法是如何实现的：
```objc
//SJStaticTableView.m
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    if ((self.sjDelegate) && [self.sjDelegate respondsToSelector:@selector(didSelectViewModel:atIndexPath:)]) {
        
        SJStaticTableviewCellViewModel *cellViewModel = [self.sjDataSource tableView:tableView cellViewModelAtIndexPath:indexPath];
        [self.sjDelegate didSelectViewModel:cellViewModel atIndexPath:indexPath];
        
    }else if((self.sjDelegate)&& [self.sjDelegate respondsToSelector:@selector(tableView:didSelectRowAtIndexPath:)]){
        
        [self.sjDelegate tableView:tableView didSelectRowAtIndexPath:indexPath];
        
    }
}
```

现在读者应该大致了解了这个框架的实现思路，现在我讲一下这个框架的定制性。


# 4. 定制性
----


这个框架有一个配置文件：SJConst.h，它定义了这个框架的所有默认数据和默认配置，比如cell左侧lable的字体，颜色；左侧label和image的距离；右侧label的字体和颜色，右侧图片的默认大小等等。来看一下代码：

```objc
#ifndef SJConst_h
#define SJConst_h

//distance
#define SJScreenWidth      [UIScreen mainScreen].bounds.size.width
#define SJScreenHeight     [UIScreen mainScreen].bounds.size.height

#define SJTopGap 8               //same as bottom gap
#define SJLeftGap 12             //same as right gap
#define SJLeftMiddleGap 10       //in left  part: the gap between image and label
#define SJRightMiddleGap 6       //in right part: the gap between image and label
#define SJImgWidth 30            //default width and height
#define SJTitleWidthLimit 180    //limt width of left and right labels

//image
#define SJIndicatorArrow @"arrow"

//font
#define SJLeftTitleTextFont               [UIFont systemFontOfSize:15]
#define SJLogoutButtonFont                [UIFont systemFontOfSize:16]
#define SJIndicatorLeftTitleTextFont      [UIFont systemFontOfSize:13]

//color
#define SJColorWithRGB(R,G,B,A)           [UIColor colorWithRed:R/255.0 green:G/255.0 blue:B/255.0 alpha:A]
#define SJLeftTitleTextColor              [UIColor blackColor]
#define SJIndicatorLeftTitleTextColor     SJColorWithRGB(136,136,136,1)

#endif /* SJConst_h */
```

这里定义的默认配置在cellViewModel和sectionViewModel初始化的时候使用：

cell的viewModel：
```objc
//SJStaticTableviewCellViewModel.m
- (instancetype)init
{
    self = [super init];
    if (self) {        
        _cellHeight = 44;
        _cellID = @"defaultCell";
        _staticCellType = SJStaticCellTypeSystemAccessoryDisclosureIndicator;//默认是存在三角箭头的cell
        _isImageFirst = YES;
        
        //都是默认配置
        _leftLabelTextFont = SJLeftTitleTextFont;
        _leftLabelTextColor = SJLeftTitleTextColor;
        _leftImageSize = CGSizeMake(SJImgWidth, SJImgWidth);
        _leftImageAndLabelGap = SJLeftMiddleGap;
        _indicatorLeftLabelTextFont = SJIndicatorLeftTitleTextFont;
        _indicatorLeftLabelTextColor = SJIndicatorLeftTitleTextColor;
        _indicatorLeftImageSize = CGSizeMake(SJImgWidth, SJImgWidth);
        _indicatorLeftImageAndLabelGap = SJRightMiddleGap;
    }
    return self;
}
```

section的viewModel：
```objc
- (instancetype)initWithCellViewModelsArray:(NSArray *)cellViewModelsArray
{
    self = [super init];
    if (self) {
        _sectionHeaderHeight = 10;
        _sectionFooterHeight = 10;
        _leftLabelTextFont = SJLeftTitleTextFont;
        _leftLabelTextColor = SJLeftTitleTextColor;
        _leftImageSize = CGSizeMake(SJImgWidth, SJImgWidth);
        _leftImageAndLabelGap = SJLeftMiddleGap;
        _indicatorLeftLabelTextFont = SJIndicatorLeftTitleTextFont;
        _indicatorLeftLabelTextColor = SJIndicatorLeftTitleTextColor;
        _indicatorLeftImageSize = CGSizeMake(SJImgWidth, SJImgWidth);
        _indicatorLeftImageAndLabelGap = SJRightMiddleGap;
        _cellViewModelsArray = cellViewModelsArray;        
    }
    return self;
}
```

显然，这个默认配置只有一组，但是可能一个app里面同时存在一个设置页和一个个人页。而这两个页面的风格也可能是不一样的，所以这个默认配置只能给其中一个页面，另一个页面需要另外配置，于是就有了定制性的功能。

再来看一下展示定制性效果的图：

![分组定制 | 同组定制](https://user-gold-cdn.xitu.io/2017/4/24/dc3d7675b3eedb3dbe4ac3b7bb20d7db)

参照这个效果图，我们看一下这两个页面的数据源是如何设置的：

分组页面：
```objc
+ (NSArray *)customCellsPageData
{
    //默认配置
    SJStaticTableviewCellViewModel *vm1 = [[SJStaticTableviewCellViewModel alloc] init];
    vm1.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm1.leftTitle = @"全部默认配置，用于对照";
    vm1.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm1.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section1 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm1]];
    
    SJStaticTableviewCellViewModel *vm2 = [[SJStaticTableviewCellViewModel alloc] init];
    vm2.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm2.leftTitle = @"左侧图片变小";
    vm2.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm2.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section2 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm2]];
    section2.leftImageSize = CGSizeMake(20, 20);
    
    SJStaticTableviewCellViewModel *vm3 = [[SJStaticTableviewCellViewModel alloc] init];
    vm3.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm3.leftTitle = @"字体变小变红";
    vm3.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm3.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section3 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm3]];
    section3.leftLabelTextFont = [UIFont systemFontOfSize:8];
    section3.leftLabelTextColor = [UIColor redColor];
    
    
    SJStaticTableviewCellViewModel *vm4 = [[SJStaticTableviewCellViewModel alloc] init];
    vm4.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm4.leftTitle = @"左侧两个控件距离变大";
    vm4.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm4.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section4 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm4]];
    section4.leftImageAndLabelGap = 20;
    
    
    SJStaticTableviewCellViewModel *vm5 = [[SJStaticTableviewCellViewModel alloc] init];
    vm5.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm5.leftTitle = @"右侧图片变小";
    vm5.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm5.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section5 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm5]];
    section5.indicatorLeftImageSize = CGSizeMake(15, 15);
    
    
    SJStaticTableviewCellViewModel *vm6= [[SJStaticTableviewCellViewModel alloc] init];
    vm6.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm6.leftTitle = @"右侧字体变大变蓝";
    vm6.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm6.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section6 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm6]];
    section6.indicatorLeftLabelTextFont = [UIFont systemFontOfSize:18];
    section6.indicatorLeftLabelTextColor = [UIColor blueColor];
    
    
    SJStaticTableviewCellViewModel *vm7= [[SJStaticTableviewCellViewModel alloc] init];
    vm7.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm7.leftTitle = @"右侧两个控件距离变大";
    vm7.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm7.indicatorLeftTitle = @"王者荣耀!";
    
    SJStaticTableviewSectionViewModel *section7 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm7]];
    section7.indicatorLeftImageAndLabelGap = 18;
    
    
    return @[section1,section2,section3,section4,section5,section6,section7];
    
}
```
>我们可以看到，定制的代码都作用于section的viewModel。

同组页面：

```objc
+ (NSArray *)customCellsOneSectionPageData
{
    //默认配置
    SJStaticTableviewCellViewModel *vm1 = [[SJStaticTableviewCellViewModel alloc] init];
    vm1.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm1.leftTitle = @"全部默认配置，用于对照";
    vm1.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm1.indicatorLeftTitle = @"王者荣耀!";
    
    
    SJStaticTableviewCellViewModel *vm2 = [[SJStaticTableviewCellViewModel alloc] init];
    vm2.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm2.leftTitle = @"左侧图片变小";
    vm2.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm2.indicatorLeftTitle = @"王者荣耀!";
    vm2.leftImageSize = CGSizeMake(20, 20);
    
    
    SJStaticTableviewCellViewModel *vm3 = [[SJStaticTableviewCellViewModel alloc] init];
    vm3.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm3.leftTitle = @"字体变小变红";
    vm3.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm3.indicatorLeftTitle = @"王者荣耀!";
    vm3.leftLabelTextFont = [UIFont systemFontOfSize:8];
    vm3.leftLabelTextColor = [UIColor redColor];
    
    
    SJStaticTableviewCellViewModel *vm4 = [[SJStaticTableviewCellViewModel alloc] init];
    vm4.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm4.leftTitle = @"左侧两个控件距离变大";
    vm4.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm4.indicatorLeftTitle = @"王者荣耀!";
    vm4.leftImageAndLabelGap = 20;
    
    
    SJStaticTableviewCellViewModel *vm5 = [[SJStaticTableviewCellViewModel alloc] init];
    vm5.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm5.leftTitle = @"右侧图片变小";
    vm5.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm5.indicatorLeftTitle = @"王者荣耀!";
    vm5.indicatorLeftImageSize = CGSizeMake(15, 15);
    
    
    SJStaticTableviewCellViewModel *vm6= [[SJStaticTableviewCellViewModel alloc] init];
    vm6.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm6.leftTitle = @"右侧字体变大变蓝";
    vm6.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm6.indicatorLeftTitle = @"王者荣耀!";
    vm6.indicatorLeftLabelTextFont = [UIFont systemFontOfSize:18];
    vm6.indicatorLeftLabelTextColor = [UIColor blueColor];
    
    
    SJStaticTableviewCellViewModel *vm7= [[SJStaticTableviewCellViewModel alloc] init];
    vm7.leftImage = [UIImage imageNamed:@"MoreGame"];
    vm7.leftTitle = @"右侧两个控件距离变大";
    vm7.indicatorLeftImage = [UIImage imageNamed:@"wzry"];
    vm7.indicatorLeftTitle = @"王者荣耀!";
    vm7.indicatorLeftImageAndLabelGap = 18;
    
    SJStaticTableviewSectionViewModel *section1 = [[SJStaticTableviewSectionViewModel alloc] initWithCellViewModelsArray:@[vm1,vm2,vm3,vm4,vm5,vm6,vm7]];
    
    return @[section1];
}
```
>为了方便比较，同组页面的定制和分组是一致的。我们可以看到，定制代码都作用于cell的viewModel上了。

为什么要有同组和分组展示？


同组和分组展示的目的，是为了展示这个框架的两种定制性。

- 分组页面所展示的是section级的定制性：cell的配置任务交给section层的viewModel。一旦设置，该section里面的所有cell都能保持这一配置。

- 同组页面所展示的是cell级的定制性：cell的配置任务交给cell层的viewModel。一旦设置，只有当前cell具有这个配置，不影响其他cell。

其实为了省事，只在section层的viewModel上配置即可（如果给每个cell都给设置相同的配置太不优雅了），因为从设计角度来看，一个section里面的cell的风格不一致的情况比较少见（我觉得不符合设计）：比如在一个section里面，不太可能两个cell里面的图片大小是不一样的，或者字体大小也不一样。

还是看一下section级的定制代码吧：
```objc
//重新设置了该组全部cell里面左侧label的字体
- (void)setLeftLabelTextFont:(UIFont *)leftLabelTextFont
{
    if (_leftLabelTextFont != leftLabelTextFont) {
        
        if (![self font1:_leftLabelTextFont hasSameFontSizeOfFont2:leftLabelTextFont]) {
            
            _leftLabelTextFont = leftLabelTextFont;
            
            //如果新的宽度大于原来的宽度，需要重新设置，否则不需要
            [_cellViewModelsArray enumerateObjectsUsingBlock:^(SJStaticTableviewCellViewModel * viewModel, NSUInteger idx, BOOL * _Nonnull stop) {
                viewModel.leftLabelTextFont = _leftLabelTextFont;
                CGSize size = [self sizeForTitle:viewModel.leftTitle withFont:_leftLabelTextFont];
                if (size.width > viewModel.leftTitleLabelSize.width) {
                    viewModel.leftTitleLabelSize = size;
                }
            }];
            
        }
    }
}

//重新设置了该组全部cell里面左侧label的字的颜色
- (void)setLeftLabelTextColor:(UIColor *)leftLabelTextColor
{
    if (![self color1:_leftLabelTextColor hasTheSameRGBAOfColor2:leftLabelTextColor]) {
         _leftLabelTextColor = leftLabelTextColor;
        [_cellViewModelsArray makeObjectsPerformSelector:@selector(setLeftLabelTextColor:) withObject:_leftLabelTextColor];
    }
}

//重新设置了该组全部cell里面左侧图片等大小
- (void)setLeftImageSize:(CGSize)leftImageSize
{
    SJStaticTableviewCellViewModel *viewMoel = _cellViewModelsArray.firstObject;
    
    CGFloat cellHeight = viewMoel.cellHeight;
    if ( (!CGSizeEqualToSize(_leftImageSize, leftImageSize)) && (leftImageSize.height < cellHeight)) {
        _leftImageSize = leftImageSize;
        [_cellViewModelsArray enumerateObjectsUsingBlock:^(SJStaticTableviewCellViewModel *viewModel, NSUInteger idx, BOOL * _Nonnull stop)
        {
            viewMoel.leftImageSize = _leftImageSize;
        }];
    }
}
```

因为每个section都持有它内部的所有cell的viewModel，所以在set方法里面，如果发现传进来的配置与当前配置不一致，就需要更新所有cell的viewModel对应的属性。


既然section的ViewModel能做这些，为什么还要有一个cell层的配置呢？

-- 只是为了提高配置的自由度罢了，万一突然来个需求需要某个cell很独特呢？（大家应该知道我说的神么意思 ^^）


cell的viewModel属性的set方法的实现和section的一致，这里就不上代码了。

# 5. 新增支持刷新功能

在1.1.2版本支持了：在更新数据源后，刷新数据源。
举个例子：在发现页模拟网络请求，在请求结束后更新某个cell的viewmodel：

```objc
//模拟网络请求
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        
        //请求成功x
        NSDictionary *responseDict = @{@"title_info":@"新游戏上架啦",
                                       @"title_icon":@"game_1",
                                       @"game_info":@"一起来玩斗地主呀！",
                                       @"game_icon":@"doudizhu"
                                       };
        //将要刷新cell的indexPath
        NSIndexPath *indexPath = [NSIndexPath indexPathForRow:1 inSection:3];
        
        //获取cell对应的viewModel
        SJStaticTableviewCellViewModel *viewModel = [self.dataSource tableView:self.tableView cellViewModelAtIndexPath:indexPath];
        
        if (viewModel) {
            //更新viewModel
            viewModel.leftTitle = responseDict[@"title_info"];
            viewModel.leftImage = [UIImage imageNamed:responseDict[@"title_icon"]];
            viewModel.indicatorLeftImage = [UIImage imageNamed:responseDict[@"game_icon"]];
            viewModel.indicatorLeftTitle = responseDict[@"game_info"];
            
            //刷新tableview
            [self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
        }
 });
```





效果图：

![更新数据源后刷新表格](https://user-gold-cdn.xitu.io/2017/4/24/8906383a490f9507e37f9f122e2a4eee)


# 6. 新增支持数据源完全依赖网络请求

在1.2.0版本支持了：数据源完全依赖网络请求的情况。

现在的最新版本里，SJStaticViewController在创建的时候分为两种情况:

1. SJDefaultDataTypeExist：在表格生成之前就存在数据，可以是表格的全部数据，也可以是表格的默认数据（后来通过网络请求来更新部分数据，参考上一节）。
2. SJDefaultDataTypeNone：意味着当前没有任何的默认数据可以使用，也就是无法生成tableview，需要在网络请求拿到数据后，再手动调用生成数据源，生成表格的方法。

```objc
//SJStaticTableViewController.h
typedef enum : NSUInteger {
    
    SJDefaultDataTypeExist,    //在表格生成之前就有数据（1. 完全不依赖网络请求，有现成的完整数据 2. 先生成默认数据，然后通过网络请求来更新数据并刷新表格）
    SJDefaultDataTypeNone,     //无法生成默认数据，需要完全依赖网络请求，在拿到数据后，生成表格
    
}SJDefaultDataType;

- (instancetype)initWithDefaultDataType:(SJDefaultDataType)defualtDataType;
```

```objc
//SJStaticTableViewController.m
- (instancetype)initWithDefaultDataType:(SJDefaultDataType)defualtDataType
{
    self = [super init];
    if (self) {
        self.defualtDataType = defualtDataType;
    }
    return self;
}

- (instancetype)init
{
    self = [self initWithDefaultDataType:SJDefaultDataTypeExist];//默认是SJDefaultDataTypeExist
    return self;
}

- (void)viewDidLoad {
    
    [super viewDidLoad];
    [self configureNav];
    
    //在能够提供给tableivew全部，或者部分数据源的情况下，可以先构造出tableview；
    //否则，需要在网络请求结束后，手动调用configureTableView方法
    if (self.defualtDataType == SJDefaultDataTypeExist) {
        [self configureTableView];
    }
}

//只有在SJDefaultDataTypeExist的时候才会自动调用，否则需要手动调用
- (void)configureTableView
{
    [self createDataSource];//生成数据源
    [self createTableView];//生成表格
}
```
看一个例子，我们将表情页设置为``SJDefaultDataTypeNone``，那么就意味着我们需要手动调用``configureTableView``方法：

```objc
- (void)viewDidLoad {
    
    [super viewDidLoad];
    
     self.navigationItem.title = @"表情";
    [self networkRequest];
}


- (void)networkRequest
{
    [MBProgressHUD showHUDAddedTo: self.view animated:YES];
    
    //模拟网络请求
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        
        [MBProgressHUD hideHUDForView: self.view animated:YES];
         self.modelsArray = [Factory emoticonPage];//网络请求后，将数据保存在self.modelsArray里面
        [self configureTableView];//手动调用
        
    });
}

- (void)createDataSource
{
    self.dataSource = [[SJStaticTableViewDataSource alloc] initWithViewModelsArray:self.modelsArray configureBlock:^(SJStaticTableViewCell *cell, SJStaticTableviewCellViewModel *viewModel) {
        
        switch (viewModel.staticCellType) {
                
            case SJStaticCellTypeSystemAccessoryDisclosureIndicator:
            {
                [cell configureAccessoryDisclosureIndicatorCellWithViewModel:viewModel];
            }
                break;
                
            default:
                break;
        }
    }];
}
```

看一下效果图：
![](https://user-gold-cdn.xitu.io/2017/4/24/dddb4eed3379c043133ba38d2d8ef1a4.png)
好了，到这里就讲差不多了，代码量虽然不多，但是都说清楚还是感觉挺需要时间想的。

希望如果各位觉得哪里不好，可以给出您的宝贵意见～




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



