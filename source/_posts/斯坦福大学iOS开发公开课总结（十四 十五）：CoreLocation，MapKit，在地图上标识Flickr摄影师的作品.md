---
title: 斯坦福大学iOS开发公开课总结（十四 十五）：CoreLocation，MapKit，在地图上标识Flickr摄影师的作品
tags: [iOS,Objective-C]
categories: iOS
---

本总结将第十四和十五课放在了一起，原因有二：第一是略去了ipad开发Demo的部分（因为笔者木有ipad，无法进行调试）。第二是两节课都讲解了关于地图框架的相关知识，故将二者放在一起总结。

在本篇总结的最后，会给大家讲解在地图上显示Flickr上摄影师的照片作品。


# Network Activity Indicator
--------
顾名思义，该控件叫做网络活动指示器。当app有网络活动时，可以让状态栏左边的小圆圈滚动用来提示用户当前的网络状态。

```
@property(nonatomic,getter=isNetworkActivityIndicatorVisible) BOOL networkActivityIndicatorVisible; 
```
如果设定为YES，状态栏上的小转轮就会转，反之亦然。

>注意：应用中的所有线程都可使用这个转轮，我们需要通过各种方法来向用户准确显示转轮的状态。

<!-- more -->

# Core Location
-------------
通过该框架的基本类：``CLLocation``，我们能获得设备处于地球上的位置信息。

## Core Location几个重要的属性：
#### 1. 坐标属性
```
typedef struct {
CLLocationDegrees latitude;    //double value
CLLocationDegrees longitude;   //double value
} CLLocationCoordinate2D;
```
#### 2. 高度
```
@property(readonly, nonatomic) CLLocationDistance altitude; //单位是米
```

#### 3. 变化精度：

```
@property(readonly, nonatomic) CLLocationAccuracy horizontalAccuracy;//水平精度
@property(readonly, nonatomic) CLLocationAccuracy verticalAccuracy;//高度精度
```

如何获得CLLocation？
通过实例化``CLLocationManager``类，让其告诉它的代理当前设备所处的位置。
下面来介绍一下``CLLocationManager``:

# CLLocationManager

-----
## CLLocationManager的工作步骤：
1.查看硬件是否支持位置更新。
2.实例化``CLLocationManager``让其告诉它的代理当前的位置。
3.设置位置更新的类型(精度)。

```
@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy; //期望的经度
@property(assign, nonatomic) CLLocationDistance distanceFilter;  //更新到该距离之内不要告诉我更新了多少
```

4.开始位置监控。

```
- (void)startUpdatingLocation;//开始更新位置
- (void)stopUpdatingLocation;//停止位置更新
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations;//位置更新的代理方法
- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error;//更新失败
```

## 位置监控的类型：

#### 1. 基于精度的监控

```
extern const CLLocationAccuracy kCLLocationAccuracyBestForNavigation; //最精确，但是非常耗能
extern const CLLocationAccuracy kCLLocationAccuracyBest;
extern const CLLocationAccuracy kCLLocationAccuracyNearestTenMeters;
extern const CLLocationAccuracy kCLLocationAccuracyHundredMeters;
extern const CLLocationAccuracy kCLLocationAccuracyKilometer;
extern const CLLocationAccuracy kCLLocationAccuracyThreeKilometers;
```
> 注意：精度越高，耗电量越大

#### 2. 位置发生重大变化时更新。

```
- (void)startMonitoringSignificantLocationChanges;
- (void)stopMonitoringSignificantLocationChanges ;
```

该方法在前台和后台都能监控位置的变化，甚至关掉app后，也可以启动应用告诉用户位置更新:
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //如果``launchOptions``存在``UIApplicationLaunchOptionsLocationKey``，说明程序启动的原因是因为位置发生了重大变化
    return YES;
}
```

#### 3. 进入某个区域更新。

3.1设定一个圆形的区域，经过该区域的时候会更新

```
- (void)startMonitoringForRegion:(CLRegion *)region;
- (void)requestStateForRegion:(CLRegion *)region;
```

3.2 通过一个信标来监控

```
@property (readonly, nonatomic) CLLocationDistance maximumRegionMonitoringDistance;//设置最大监控距离

- (void)startRangingBeaconsInRegion:(CLBeaconRegion *)region;//设置信标
```

#### 4. 监控前进的方向

# MapKit
---------
MapKit是用于显示地图的框架，它通过``MKMapView``来显示地图。
我们来看一下该框架中几个比较重要的元素：

## 1. MKMapView

MKMapView就是用来显示地图的View。

MKMapView的属性：
```
@property (nonatomic) MKMapType mapType;// MKMapTypeStandard : 标准；MKMapTypeSatellite:卫星；MKMapTypeHybrid：叠加
@property (nonatomic) BOOL showsUserLocation; //显示用户的地点
@property (nonatomic, readonly, getter=isUserLocationVisible) BOOL userLocationVisible;
//用户坐标是否可见
@property (nonatomic, getter=isZoomEnabled) BOOL zoomEnabled; //是否可放大缩小
@property (nonatomic, getter=isRotateEnabled) BOOL rotateEnabled; //是否可旋转
@property (nonatomic, getter=isPitchEnabled) BOOL pitchEnabled; //3D效果

```

## 2. MKAnnotationView
在``MKMapView``视图里，可以显示用于标注具体位置的“大头针” ，它是MapKit框架里的``AnnotationView``。

MKAnnotationView的属性：
```
@property (nonatomic, strong, nullable) id <MKAnnotation> annotation;
@property (nonatomic, strong, nullable) UIImage *image;//大头针的图像
@property (strong, nonatomic, nullable) UIView *leftCalloutAccessoryView;//左附属对话框
@property (strong, nonatomic, nullable) UIView *rightCalloutAccessoryView;//右附属对话框
@property (nonatomic, getter=isDraggable) BOOL draggable //是否可拖动
```

大头针被点击时调用的方法：
```
- (void)mapView:(MKMapView *)mapView didSelectAnnotationView:(MKAnnotationView *)view
```

## 3. id<MKAnnotation>
AnnotationView的数据源就是：id<MKAnnotation>，任何遵从该协议的对象都可以成为AnnotationView的数据源，也就是说，任何遵守    ``MKAnootation``协议的对象你都可以将其放入地图中。

我们先看一下在MKMapView里的关于MKAnnotation的属性：
```
@property (nonatomic, readonly) NSArray<id<MKAnnotation>> *annotations;//包含MapView所显示的所有Annotaion
```

注意：annotations是只读的数组，只能添加或者删除。

```
- (void)addAnnotation:(id <MKAnnotation>)annotation;
- (void)addAnnotations:(NSArray<id<MKAnnotation>> *)annotations;
- (void)removeAnnotation:(id <MKAnnotation>)annotation;
- (void)removeAnnotations:(NSArray<id<MKAnnotation>> *)annotations;
```

MKAnnotation协议的方法：

```


@protocol MKAnnotation <NSObject>
@property (nonatomic, readonly) CLLocationCoordinate2D coordinate; //坐标

@optional
@property (nonatomic, readonly, copy, nullable) NSString *title;//标题
@property (nonatomic, readonly, copy, nullable) NSString *subtitle;//副标题

- (void)setCoordinate:(CLLocationCoordinate2D)newCoordinate ;//设置坐标
```

那么二者是如何关联的呢？
通过MKMapView的代理方法：
```
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
{
    //提供一个 annotation，返回一个 MKAnnotationView
}
```
## 4. Callout(对话框)
点击大头针（MKAnnotationView），会出现一个白底的对话框，它被叫做``callout``,可以设置它的主标题和副标题。另外还有左右附属实图，它们可以显示图片或者箭头，也可被点击。
﻿

# Demo
----
## Demo需求：
- 显示从flickr抓取的摄影师列表。
- 点击列表中的一项，打开地图，在当前摄影师所照照片的地点显示大头针。
- 点击其中的一个大头针，显示照片详情：缩略图和名称。
- 点击箭头按钮，滑入显示照片的页面，显示原始照片。

## Demo效果图：

![在地图显示照片拍摄位置](http://upload-images.jianshu.io/upload_images/859001-269dd622430d4972.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要代码段和知识点：

#### 1. 更改Core Data模型
在上一节课的基础上，我们需要在模型里的``Photo``实体添加经度和纬度的属性，还有大头针缩略图的URL属性。
在更新属性后，一定要重新生成对应该实体的类文件，并且要将原app删除，因为数据库前后是不兼容的。


![更新模型](http://upload-images.jianshu.io/upload_images/859001-0bf538fa1a08fe3d.gif?imageMogr2/auto-orient/strip)



#### 2. 新建PhotosByPhotographerMapViewController.h，用来显示``MKMapView``

因为要在地图上显示摄影师所照照片的位置，因此，该类的数据源来自摄影师模型：``Photographer``。

```
#import <UIKit/UIKit.h>
#import "Photographer.h"
@interface PhotosByPhotographerMapViewController : UIViewController
@property (nonatomic, strong) Photographer *photographer;//公共API：摄影师
@end
```

``
#import "PhotosByPhotographerMapViewController.h"

#import <MapKit/MapKit.h>

@interface PhotosByPhotographerMapViewController ()<MKMapViewDelegate>
@property (strong, nonatomic) IBOutlet MKMapView *mapView;//地图view
@property (nonatomic,strong) NSArray *photosByPhotographer;//装入摄影师拥有的照片的数组
@end

@implementation PhotosByPhotographerMapViewController
@end
``

#### 3. 导入Mapkit的framework

需要注意的是，除了要在类文件引用``<MapKit/MapKit.h>``框架以外，还要手动向项目中添加该框架：

![手动添加MapKit框架.gif](http://upload-images.jianshu.io/upload_images/859001-8fb04dfbac7bbdd7.gif?imageMogr2/auto-orient/strip)


#### 4. 更新photographer和mapView后更新annotation：
```
- (void)setMapView:(MKMapView *)mapView
{
    _mapView = mapView;

    //设置代理
    self.mapView.delegate = self;

    //更新
    [self updateMapViewAnnotations];
}

- (void)setPhotographer:(Photographer *)photographer
{
    _photographer = photographer;
    //导航栏标题
    self.title = photographer.name;
    //准备更新数组，要事先设置其为nil，否则不会生成新的
    self.photosByPhotographer = nil;
    [self updateMapViewAnnotations];
}

- (void)updateMapViewAnnotations
{
    [self.mapView removeAnnotations:self.mapView.annotations];
    [self.mapView addAnnotations:self.photosByPhotographer];
    [self.mapView showAnnotations:self.photosByPhotographer animated:YES];
}
```
#### 5. 自定义点击大头针后显示的view

```
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
{

    //类似UITableviewCell的复用
    static NSString *reuseId = @"PhotosByPhotographerMapViewController";    

    MKPinAnnotationView *view = (MKPinAnnotationView*)[mapView dequeueReusableAnnotationViewWithIdentifier:reuseId];    

    if (!view) {
        view = [[MKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:reuseId];
       //是否显示callout
        view.canShowCallout = YES;
        //设置左部分的callout：UIImageView
        UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 46, 46)];
        view.leftCalloutAccessoryView = imageView;
        //设置右部分的callout：UIButton
        UIButton *disclosurebutton = [[UIButton alloc] init];
        [disclosurebutton setBackgroundImage:[UIImage imageNamed:@"disclosure"] forState:UIControlStateNormal];
        [disclosurebutton sizeToFit];
        view.rightCalloutAccessoryView = disclosurebutton;
    }

    view.annotation = annotation;
    return view;

}
```
#### 6. 点击大头针，更新callout左侧显示的缩略图

```

/**
 *  点击大头针view
 *
 *  @param mapView 大头针所属的mapView
 *  @param view    大头针view
 */
- (void)mapView:(MKMapView *)mapView didSelectAnnotationView:(MKAnnotationView *)view
{
    [self updateLeftCalloutAccessoryViewInAnnotationView:view];
}

/**
 *  更新callout里的图片（在左侧）
 *
 *  @param annotationView 当前被点击的大头针view
 */
- (void)updateLeftCalloutAccessoryViewInAnnotationView:(MKAnnotationView *)annotationView
{
    UIImageView *imageView = nil;
    if ([annotationView.leftCalloutAccessoryView isKindOfClass:[UIImageView class]]) {
        imageView = (UIImageView *)annotationView.leftCalloutAccessoryView;
    }
    if (imageView) {
        Photo *photo = nil;
        if ([annotationView.annotation isKindOfClass:[Photo class]]) {
            photo = (Photo *)annotationView.annotation;
        }
        if (photo) {
            NSString *urlString = photo.thumbnailURL;
            imageView.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:urlString]]];
        }
    }
}
```

>注意：显示图片的代码：`` imageView.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:urlString]]];``方法会阻塞主线程，实际操作中应该放在子线程中执行。详情请参考笔者另一篇讲解关于多线程的博客：[最浅显易懂的iOS多线程技术 - GCD的教程](http://www.jianshu.com/p/6e74f5438f2c)。


#### 7. 点击callout，在下一页面显示原图
```
/**
 *  点击callout实行跳转
 *
 *  @param mapView 当前的mapView
 *  @param view    当前callout所属的AnnotationView
 *  @param control callout内部被点击的控件
 */
- (void)mapView:(MKMapView *)mapView annotationView:(MKAnnotationView *)view calloutAccessoryControlTapped:(UIControl *)control
{
    [self performSegueWithIdentifier:@"Show Photo" sender:view];
}

/**
 *  调转执行前的代码
 *
 *  @param segue  连接前后两个控制器的segue
 *  @param sender 被点击的AnnotaionView
 */
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
 
    if ([sender isKindOfClass:[MKAnnotationView class]]) {
        [self prepareViewController:segue.destinationViewController
                           forSegue:segue.identifier
                   toShowAnnotation:((MKAnnotationView *)sender).annotation];
    }
}

/**
 *  为目标控制器准备数据（图片的URL）
 *
 *  @param vc              目标控制器
 *  @param segueIdentifier segue.identifier
 *  @param annotation      被点击的AnnotaionView
 */
- (void)prepareViewController:(id)vc
                     forSegue:(NSString *)segueIdentifier
             toShowAnnotation:(id <MKAnnotation>)annotation
{
    Photo *photo = nil;
    if ([annotation isKindOfClass:[Photo class]]) {
        photo = (Photo *)annotation;
    }
    if (photo) {
        if (![segueIdentifier length] || [segueIdentifier isEqualToString:@"Show Photo"]) {
            if ([vc isKindOfClass:[ImageViewController class]]) {
                ImageViewController *ivc = (ImageViewController *)vc;
                ivc.imageURL = [NSURL URLWithString:photo.imageURL];
                ivc.title = photo.title;
            }
        }
    }
}
```

>注意：这里的`` ivc.imageURL = [NSURL URLWithString:photo.imageURL];``代码同样会阻塞主线程，实际操作中应该放在子线程来做！

>而且,本demo的图片地址应该都是在墙外的，所以最好先让电脑翻墙，然后在模拟器上运行比较好。

# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，可以进入我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)下载哦~    这一系列到现在为止的所有Demo都在里面，分为英文注释版本和中文注释版本两种。

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~


