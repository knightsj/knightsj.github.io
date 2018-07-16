
---
title: 高度封装FMDB框架：各用一句代码更新（添加&修改），查询，删除用户信息
tags: [iOS,Objective-C]
categories: Production
---


在移动开发中，有时不得不在客户端本地保存一些数据。在iOS端，我们可以使用plist，属性列表等技术来存储数据，而相比而下更高端一点的，我们也可以使用**数据库**来存储数据。

有趣的是，很多iOS开发者没有去选择使用苹果自家的Core Data技术来操作数据库，而是选择了[FMDB]([https://github.com/ccgus/fmdb](https://github.com/ccgus/fmdb))这个第三方框架。

该框架很好地封装了操作繁琐的SQLite语句，让数据库的操作更加面向对象，而且上手快，门槛低，不用学习数据库的相关知识就可以使用自如。如此优秀流行的框架是值得学习的，于是笔者这两天研究了一下FMDB。研究后，略有所思，将它封装了一下，写了一个``Manager``类，最后结合了一个Demo演示如何使用这个类。

该博客分为两个部分:第一个部分讲解笔者封装的这个``Manager``类；第二部分结合Demo来体现该类的实用性。

<!-- more -->

# 1. 封装FMDB
-----

## 1.1 封装类的功能

首先看一下这个类的大名：SJUserInfoManager

- SJ：笔者的名字缩写，作为前缀，都懂得。
- UserInfo:说明这个类用于操作用户信息。
- Manager:这个类只具有工厂方法，省内存，而且方便使用。


**这个类的功能是：它可以创建一个关于用户信息的表（数据库），可以保存，修改，读取，删除用户信息**。

>现在几乎每个app都有登录功能，有登录就意味着有用户信息。有的时候，我们需要将一些用户信息存储到本地，方便今后的读取和操作。

>而对于用户信息，几乎永远不变的一项就是用户id，因为用户可以改自己的名字，自己的注册手机号，用户签名等等，然而用户id是唯一一成不变的，后端查找用户信息一般都通过用户id来查找，这不难理解。

因此，这个封装类基于用户id（user_id），让使用者可以自主添加**一项**自定义的用户信息（可以是用户名，用户签名，用户性别等等），从而形成一个只具有两个纵行的表（第一个纵行是默认的字段：user_id,第二个纵行是自定义字段，可以是user_name等等）。

这样一来，我们就可以生成很多基于用户id的表：可以是用户姓名的表，可以是用户出生日期的表，可以是用户手机号的表等等。

## 1.2 API介绍

该封装类的API一共有五个，分别是：
1. 创建表格
2. 更新用户信息（添加&修改）
3. 查询某个用户的信息
4. 查询全部用户的信息
5. 删除某个用户的信息

下面开始一一讲解：

#### 1. 创建表格

```objc
/*
 ********** Create table with tableName and fieldName **********
 *
 * @param   dataBaseName    tableNameString (表的名字)
 * @param   userInfoField   fieldNameString（自定义字段名）
 *
 * @return  if the table is successfully created
 *
 */
+ (BOOL)createDataBaseWithName:(NSString *)dataBaseName andUserInfoField:(NSString *)userInfoField;
```

>这个方法的意图很明显，只要传进去表的名字和自定义字段的名字就能创造一个表。
- 创建成功，返回YES；
- 创建失败，返回NO。

>而且这个表有一个默认的字段：user_id。因为通过这个类创建用户信息的表是基于用户id来操作的，前面已有说明。

#### 2. 更新用户信息（添加&修改）

```objc
/*
 ********** update specific userInfo with specific userID and userInfoField and userInfoValue **********
 *
 * @param   dataBaseName    tableNameString (表的名字)
 * @param   userID          userIDString（用户id的值）
 * @param   userInfoField   fieldNameString（自定义字段名）
 * @param   userInfoValue   userInfoValueString（字段对应的值）
 *
 * @return  the result of updating specific userInfo 
 *
 */
+ (NSString *)updateUserInfoIntoDataBase:(NSString *)dataBaseName withUserID:(NSString *)userID andUserInfoField:(NSString *)userInfoField andUserInfoValue:(NSString *)userInfoValue;
```

>这个方法用于更新用户信息，传入表的名字，用户id，自定义字段名，和自定义字段对应的值。
- 如果输入的表不存在，则返回相应的错误信息。
- 如果输入的用户id已经存在，就更新对应的用户信息。
- 如果输入的用户id不存在，就添加这个用户的信息。


#### 3. 查询某个用户的信息

```objc
/*
 ********** Query specific userInfoValue with tableName and userID and userInfoField **********
 *
 * @param   dataBaseName    tableNameString (表的名字)
 * @param   userID          userIDString（用户id的值）
 * @param   userInfoField   fieldNameString（自定义字段名）
 *
 * @return  specific userInfoValue （表内某用户id对应的用户信息）
 *
 */
+ (NSString *)queryUserInfoInDataBase:(NSString *)dataBaseName WithUserID:(NSString *)userId andUserInfoField:(NSString *)userInfoField;

```
>该方法用于查询某个用户的信息，传入表的名字，用户id和自定义字段名。
- 如果输入的表不存在，则返回相应的错误信息。
- 如果输入的用户id存在，则返回对应的信息。
- 如果输入的用户id不存在，则返回相应的错误信息。


#### 4. 查询全部用户的信息

```objc
/*
 ********** Query all userInfos in this table with userInfoField **********
 *
 * @param   dataBaseName    tableNameString (表的名字)
 * @param   userInfoField   fieldNameString（自定义字段名）
 *
 * @return  all the userInfos in this table （表内所有的用户信息）
 *
 */
+ (NSDictionary *)queryUserInfosInDataBase:(NSString *)dataBaseName andUserInfoField:(NSString *)userInfoField;
```

>该方法用户获取某个表内的所有用户信息，传入表的名字和自定义字段名即可。
返回的字典里包含一个数组，数组元素为表的每一行的信息。每一行的信息是一个字典：
- 其中一个key为默认的字段名：user_id，它的值为对应的user_id的值。
- 另一个key为添加的自定义字段名，它对应的值为该字段对应的值。

#### 5. 删除某个用户的信息

```objc
/*
 ********** Delete specific userInfo with specific userID **********
 *
 * @param   dataBaseName    tableNameString (表的名字)
 * @param   userId          userIDString（用户id的值）
 *
 * @return  the result of deleting specific userInfo （删除的结果）
 *
 */
+ (NSString *)deleteUserInfoInDataBase:(NSString *)dataBaseName WithUserID:(NSString *)userId;
```
>该方法用于删除表里的某个用户信息，只要传入表的名字和用户id即可，可以删除表中对应的一整行信息。同样地，如果输入的表不存在，则返回相应的错误信息。


如果觉得有点抽象的话，可以看下面这个Demo，可以看到该封装类的具体使用方法。

# 2. Demo演示
-----

在这个Demo中，我们要在表里添加的自定义字段名(fieldNameString)为：“user_name”。也就是说这个表将保存用户id和用户名信息。

## 2.1 需求

1. 在插入板块中插入用户信息（用户id和用户名）。
2. 在查询板块中，根据输入的用户id，可以显示对应的用户名。如果没有对应的用户id，则显示“没有对应id”。
3. 在删除板块中，根据输入的用户id，可以删除对应的用户信息（包括用户id和用户名，也就是删除了表的一整行）。如果没有对应的用户id，则显示“没有对应id”。
4. 点击导航栏右侧的按钮，进入“用户信息列表页”。在这一页显示当前表里的全部用户的信息（在cell的textLabel里显示用户名；在cell的detailTextLabel里显示用户id）。

## 2.2 效果图


![左图：数据库操作页面（插入，查询，删除板块）| 右图：数据库全部用户信息](http://upload-images.jianshu.io/upload_images/859001-404e4004af3b981a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![本地沙盒中sqlite表文件内容](http://upload-images.jianshu.io/upload_images/859001-f32a934c4eb45af6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2.3 代码讲解


**1. 最先导入这个封装类和FMDB框架：**

```objc
#import "SJUserInfoManager.h"
```

**2. 因为表的名字和自定义字段是固定的，所以就以宏来定义了：**

```objc
#define DATABASENAME  @"userInfoTable" //表的名字
#define USERINFOFIELD @"user_name"     //自定义字段的名字
```

**3. 添加几个输入框的属性，获取输入框的内容；和查询结果显示**Label：

```objc
@property (strong, nonatomic) IBOutlet UITextField *insertUserIdTextfield;       //插入输入框：输入用户id
@property (strong, nonatomic) IBOutlet UITextField *insertUserInfoValueTextfiled;//插入收入框：输入用户名
@property (strong, nonatomic) IBOutlet UITextField *queryUserIdTextfield;        //查询输入框：输入用户id
@property (strong, nonatomic) IBOutlet UILabel *queryUserInfoValueLabel;         //查询结果显示Label:显示用户id对应的用户名
@property (strong, nonatomic) IBOutlet UITextField *deleteUserIdTextField;       //删除输入框：输入用户id
```


**4. 下面看一下封装的增改&查&删的代码：**

```objc
//插入用户信息
- (IBAction)insertAction:(id)sender {
   
    NSString *result = @"";
    
    if (self.insertUserInfoValueTextfiled.text.length == 0) {
        
         result = @"Please Input UserID!";//没有输入用户id就点击了插入按钮
        
    }else{
        
         result = [SJUserInfoManager updateUserInfoIntoDataBase:DATABASENAME withUserID:self.insertUserIdTextfield.text andUserInfoField:USERINFOFIELD andUserInfoValue:self.insertUserInfoValueTextfiled.text];
    }
   
   [self showAlertWithTitle:result];
    
}
//查询用户信息
- (IBAction)queryUserInfoValue:(UIButton *)sender {
    
    NSString *result = @"";
    
    if (self.queryUserIdTextfield.text.length == 0) {
        
        result = @"Please Input UserID!";//没有输入用户id就点击了查询按钮
        self.queryUserInfoValueLabel.text = @"";//重置查询输入框
        
    }else{
        
        result =  [SJUserInfoManager queryUserInfoInDataBase:DATABASENAME WithUserID:self.queryUserIdTextfield.text andUserInfoField:USERINFOFIELD];
        self.queryUserInfoValueLabel.text = result;
        [self showAlertWithTitle:result];
        
    }    
    [self showAlertWithTitle:result];
}
//删除用户信息
- (IBAction)deleteUserInfoWithUserID:(UIButton *)sender {
    
    NSString *result = @"";
    
    if (self.deleteUserIdTextField.text.length == 0) {
        
        result = @"Please Input UserID!";
        
    }else{
        
        result =  [SJUserInfoManager deleteUserInfoInDataBase:DATABASENAME WithUserID:self.deleteUserIdTextField.text];
    }
    [self showAlertWithTitle:result];
}
```

>其实不难看出，除了处理错误信息的代码以外，数据库的操作代码是非常简洁的:都是一行结束，而且返回一个结果字符串或者布尔值。

**5. 在进入第二页之前，需要查询表内所有的用户信息并传递给第二个页面的VC：**

```objc
-(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
    if ([segue.destinationViewController isKindOfClass:[DataListTableViewController class]]) {
        if ([segue.identifier isEqualToString:@"userInfosList"]) {
            
            NSDictionary *userInfosDict = [SJUserInfoManager queryUserInfosInDataBase:DATABASENAME andUserInfoField:USERINFOFIELD];
            DataListTableViewController *dataListVc = (DataListTableViewController *)segue.destinationViewController;
            dataListVc.userInfosDict = userInfosDict;
        }
    }    
}
```


**6. 在第二页的viewDidLoad方法里获取用户信息列表，并刷新表格将其显示出来：**

```objc
- (void)viewDidLoad {
    
    [super viewDidLoad];
    
    NSString *alertTitle = [self.userInfosDict objectForKey:@"result"];
    [self showAlertWithTitle:alertTitle];
    
    NSArray *userInfosArray = [self.userInfosDict objectForKey:@"userInfosArray"];
    if ([userInfosArray count] != 0) {
         self.userInfosArray = userInfosArray;
        [self.tableView reloadData];
    }
    
}
```

想知道Demo是如何实现的么？

Demo传送门：[SJUserInfoManager - GitHub](https://github.com/Shijie0111/SJUserInfoManager)

如果可以给星星，或者给笔者提意见，那就再好不过啦~

# 最后的话
----

说来惭愧，笔者封装的这个类的功能还是很有局限性的：
1. 操作表格前必须创建一次表格（一次就可以）。
2. 只支持NSString类型的值。
3. 除user_id字段以外只支持一个自定义字段。
4. 不同的自定义字段必须对应不同的表格名，也就是说不同的自定义字段不能对应同一个表格名。
5. 数据库操作API的返回值还不够完善，最好以枚举类型返回，有待改进。
6. 没有使用FMDB的队列API。

虽然这个封装很简单，但是笔者始终赞同**学习的最终目的在于应用和创造**这句话。如果只是用眼睛看FMDB框架的API或是复制粘贴一些别人写好的FMDB的应用代码，那么其实是意义不大的。

如果在学习后，可以融会贯通，将学到的知识可以按照自己的意图加以改进和运用的话，那么相对于“搬运工”式学习来说，显然收获更大。

这是笔者第一个开源项目，虽然简单，算上优化和改bug只写了大概3个小时，但是毕竟是开源的第一步，对自己的鼓励还是蛮大的，以后还会封装优化更多的库，加油~



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



