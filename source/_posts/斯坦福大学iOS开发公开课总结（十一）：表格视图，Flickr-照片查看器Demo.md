---
title: 斯坦福大学iOS开发公开课总结（十一）：表格视图，Flickr-照片查看器Demo
tags: [iOS,Objective-C]
categories: iOS
---


# UITableview
----

UITableview是iOS软件中最常见的视图，用来以表格的形式显示数据。

## 数据源方法

```
- (NSInteger)numberOfRowsInSection:(NSInteger)section;//表格的总section数，默认为返回1，可以不实现
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView; //返回当前section的行数，必须实现
- (nullable __kindof UITableViewCell *)cellForRowAtIndexPath:(NSIndexPath *)indexPath; //返回某section某row的cell，必须实现
```

## 代理方法

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;//cell被点击是调用的方法
```

## 根据cell获得对应的indexPath
```
 NSIndexPath *indexPath = [self.tableView  indexPathForCell :sender];
```

## UITableView Spinner
顶部加载时显示的小圆圈动画
```
@property (nonatomic, strong, nullable) UIRefreshControl *refreshControl;

- (void)beginRefreshing;
- (void)endRefreshing;
```

## 模型改变，刷新表格

```
- (void)reloadData; //刷新全部表格：一般在模型大部分变化的时候才调用，在某个数据变化时不推荐使用

- (void)reloadRowsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths withRowAnimation:(UITableViewRowAnimation)animation;//只刷新某一个cell，在某行货少数行数据变化时推荐使用
```

<!-- more -->

# Universal Application
--------

Universal Application通用应用是指既可以在iPhone上运行，也可以在iPad上运行的应用，它有两个故事版文件，一个是针对iphone的，另一个是针对ipad的。

iPad有两种独有的视图：

1. Split View：拆分视图
2. Popover:弹窗

识别是否是ipad
```
BOOL iPad  = ([{UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPad)
```

# UISplitViewController
---------

## UISplitViewController包括
- Master View Controller
- Detail View Controller

UISplitViewController是storyboard的最顶层，不能被加入到UIViewController里面

## 获得SplitViewController：
返回当前UIViewcontroller所在的SplitViewController:
```
UIViewController.h

@property (strong) UISplitViewController  *splitViewController;
```

## 获得SplitViewController的master和detail：
```
@property (copy) NSArray *viewControllers;//0：master;1: detail
```

## UISplitViewControllerDelegate

在awakeFromNib设置此代理,代理负责 控制master和detail何时出现

代理的几个方法：

```
- (BOOL)splitViewController:(UISplitViewController *)svc shouldHideViewController:(UIViewController *)vc inOrientation:(UIInterfaceOrientation)orientation{
           return NO; //永远不隐藏master，master和detail将一直在屏幕上显示，无论是横屏或竖屏 
           return UIInterfaceOrientationIsPortrait(orientation);//竖屏不显示master 但是竖屏时左上角有按键可以显示master，但是
不实现这个代理就不能出现按钮了。
}
```
在横屏或竖屏是否该隐藏master

# Popovers
------
Popover是弹窗控件，它的作用是控制另一个视图控制器弹出到屏幕上，也是ipad独有的控件。
因为ipad的面积比较大，所以有时可以只以弹窗的形式提供信息而不用跳转到下一页面。

注意：这个控件并没有继承UIViewController，是一个NSObject

## Popover的Segue是``UIStroyboardPopoverSegue``。     
在Popover出现之前：
```
- (void)prepareForSegue: (UIStoryboardSegue *)segue sender: (id)sender
{
    if([segue isKindOfClass:[UIStroyboardPopoverSegue class]]){
        
         UIPopoverController *popoverController = ((UIStroyboardPopoverSegue *)segue.)popoverController;

     }
}
```

## 使Popover消失：

```
- (void)dismissPopoverAnimated:(BOOL)animated;
```

默认情况下，点击外部任何的地方都能使它消失，除非我们给它指定即使点击也不会消失的``UIVIew``。

```
@property (copy) NSArray *passthroughViews;
```

# Demo
-----------------

该Demo是同时适用iPad 和iPhone的，可惜笔者没有iPad，无法调试，于是只适配了iPhone，以后有机会会补上适配iPad的代码的。

## Demo需求
- 第一个页面用表格显示从Flickr抓取的图片数据，只显示图片名和图片详情。
- 点击第一个页面的cell，跳转到图片详情页。
- 图片详情页显示具体的大图，可以伸缩，可以移动。

## 效果图

![效果图](http://upload-images.jianshu.io/upload_images/859001-b0a8e98131ae59b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要代码段

#### 1. 获取Flickr上的数据

Flickr提供了公共的接口提供了自家的照片，接口文件在本Demo里的``Flickr Fetcher``文件里，唯一注意的是需要申请``APIKEY``,[申请网址]([http://www.flickr.com/services/api/misc.api_keys.html](http://www.flickr.com/services/api/misc.api_keys.html))。

解析照片数据的过程是比较耗时的，所以需要分配到子线程来进行。获得数组后，在主线程将数组赋予当前类的属性里。

```
- (void)fetchPhotos
{
    self.photos = nil;
    NSURL *url = [FlickrFetcher URLforRecentGeoreferencedPhotos];

    //手动创建一个子线程
    dispatch_queue_t fetchQ = dispatch_queue_create("flickr fetcher", NULL);

    dispatch_async(fetchQ, ^{

        //获得json数据，比较耗时
        NSData *jsonResults = [NSData dataWithContentsOfURL:url];

        //获得字典
        NSDictionary *propertyListResults = [NSJSONSerialization JSONObjectWithData:jsonResults options:0 error:NULL];
        NSArray *photos = [propertyListResults valueForKeyPath:FLICKR_RESULTS_PHOTOS];
   
        dispatch_async(dispatch_get_main_queue(), ^{

           //回到主线程     
            self.photos = photos;

        });    
    });
}

```

>NULL是C指针，代表指向OC指针的指针没有指向任何对象

什么是指向OC指针的指针？：&error是指向error的指针
如果我们这样写，就可以获得error：
```
 NSError *error = nil;
 NSDictionary *propertyListResults = [NSJSONSerialization JSONObjectWithData:jsonResults options:0 error:&error];

```

如果我们不关心error，就可以传NULL。

好了，现在我们获得了数据，需要刷新表格：

#### 2. 刷新表格
```
- (void)setPhotos:(NSArray *)photos
{
    _photos = photos;
    [self.tableView reloadData];
}
```

只是刷新表格是不够的，还要实现``UITableView``的数据源方法来告诉``TableView``如何显示数据。（调用``reload``方法后会调用这些数据源方法）

#### 3. 实现数据源方法
```
#pragma mark - Table view data source

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    //只有一组
     return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
   //行数为图片的个数
    return self.photos.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

    //从重用池中拿到cell
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Flickr Photo Cell" forIndexPath:indexPath];

    //通过indexPath来获得在数据里对应的图片数据
    NSDictionary *photoDict = self.photos[indexPath.row];
    //设置主标题和副标题

    cell.textLabel.text = [photoDict valueForKeyPath:FLICKR_PHOTO_TITLE];

    cell.detailTextLabel.text = [photoDict valueForKeyPath:FLICKR_PHOTO_DESCRIPTION];    

    return cell;

}

```
#### 4. 点击cell，实现跳转

```
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation

- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {

    if ([sender isKindOfClass:[UITableViewCell class]]) {

        NSIndexPath *indexPath = [self.tableView indexPathForCell:sender];    

        if (indexPath) {
         
            if ([segue.destinationViewController isKindOfClass:[ImageViewController class]]) {

                [self prepareImageViewController:segue.destinationViewController toDisplayPhoto:self.photos[indexPath.row]];                

            }
        }
    }
}

- (void)prepareImageViewController:(ImageViewController *)ivc toDisplayPhoto:(NSDictionary*)photo
{

   //获得图像的URL传给ImageViewController
    ivc.imageURL = [FlickrFetcher URLforPhoto:photo format:FlickrPhotoFormatLarge];
   //导航栏的标题为图片的名字
    ivc.title = [photo valueForKey:FLICKR_PHOTO_TITLE];

}

```

>这里的``ImageViewController``复用了[斯坦福大学iOS开发公开课总结（十） ：多线程，UIScrollView，图片浏览器Demo](http://www.jianshu.com/p/ddb4f528b334)里第二个页面。

#### 5. 优化
每次跳转到图片详情页，将图片的原点设置在最左上端，并且大小恢复到该图片的原始大小

```
- (void)setImage:(UIImage *)image
{

    //重置缩放大小为1
    self.scrollView.zoomScale = 1.0;
    self.imageView.image = image;
    [self.imageView sizeToFit];

    //将视图框的原点设在左上角
    self.imageView.frame = CGRectMake(0, 0, self.image.size.width, self.image.size.height);

    self.scrollView.contentSize = self.image? self.image.size : CGSizeZero;

    [self.spinner stopAnimating];

}
```


# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，可以进入我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)下载哦~    这一系列到现在为止的所有Demo都在里面，分为英文注释版本和中文注释版本两种。

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~


