---
title: 斯坦福大学iOS开发公开课总结（十二 十三）：CoreData,表格视图，Flickr摄影师资料列表Demo
tags: [iOS,Objective-C]
categories: iOS
---

第十二课和第十三课都介绍了CoreData的知识，并在十三课的中段通过一个Demo来具体实现了CoreData的操作。

笔者之前从未接触过Core Data的相关知识，因此学期这两节课比较吃力，这一篇总结还是有很多需要改进的地方，以后随着对Core Data认识的深入和对这两节课的反复咀嚼，会不断更新该总结。

开始吧！

<!-- more -->


# Core Data
------

Core Data是一种持久化技术，它能将模型对象的状态持久化到磁盘，但它最重要的特点是：Core Data不仅是一个加载、保存数据的框架，它还能和内存中的数据很好的共事。

排除错误认识：Core Data**并不是数据库!** 它只是连接类（Class）和数据库（SQL）的桥梁。通过Core Data的相关功能，我们可以对数据库进行增删改查的操作。



## CoreData是如何工作的呢？



### 1. 创建对象的可视化映射

在看到可视化映射之前，需要了解**实体**的概念：

>**实体的概念**：每个实体是一个表，每个表对应一个对象。
>简单粗暴的理解：实体在数据库领域叫做表，在面向对象领域叫做对象。


>**实体之间的关系**=表之间创建关系，对象之间的关系。

**注意**：两个对象之间的关系在两个对象端具有不同的名称。而且关系的对应数量也是不同的：``to one``,`` to many``。

>举个🌰 ：摄影者和照片的关系：
>摄影者对应多个照片，但是照片只对应一个摄影者。

那么言归正传，如何创建对象的可视化映射呢？
1. 创建模型文件，用来装入各种需要映射的实体。
2. 在模型内部添加实体（Entity），创建实体之间的关系（必要时）。

下面笔者录制了创建实体，增加实体属性，连接实体的操作：


![创建实体，增加属性](http://upload-images.jianshu.io/upload_images/859001-ca646fa9e1e18f5a.gif?imageMogr2/auto-orient/strip)


### 2. 为实体创建NSManagedObjectd子类

我们需要将刚得到的可视化的实体“转变为”具体的类。在Core Data中，这些类都是NSManagedObjectd子类。

下面演示一下其创建过程：

![创建NSManagedObjectd子类](http://upload-images.jianshu.io/upload_images/859001-2eab784599ea6776.gif?imageMogr2/auto-orient/strip)




以实体``Photo``为例，系统为我们生成了``Photo.h``和``Photo.m``。

我们先看一下``Photo.h``:

``

#import <Foundation/Foundation.h>
#import <CoreData/CoreData.h>

NS_ASSUME_NONNULL_BEGIN
@interface Photo : NSManagedObject
// Insert code here to declare functionality of your managed object subclass
@end
NS_ASSUME_NONNULL_END
#import "Photo+CoreDataProperties.h"
``


>我们可以看到，``Photo``类继承了``NSManagedObject``。



但是，有意思的是，系统还为我们自动生成了``Photo+CoreDataProperties.h``和``Photo+CoreDataProperties.m``，详情见动图左侧，创建实体类之后。


**思考**：
生成这两个文件的目的是什么呢？
首先，我们首先要知道这两个文件是什么：
他们构成了``Photo``类的分类(Category)。

那么什么是分类呢？
通过分类，我们可以向原有的类添加方法，而不需要通过继承的方式。分类的局限是：在分类里不能再添加属性。

那么显然，通过``Photo+CoreDataProperties``，我们就可以不用继承``Photo``类来给其添加方法。因为原有的``Photo``类只具有属性，除了获取属性之外，并不能为我们做其他的事情。这时，如果可以在其他的地方给其无限地添加方法还是很具有诱惑力的。令人欣慰的是，系统可以自动为我们生成。



### 3. 通过创建NSManagedObjectContext访问，操作数据库
数据库创建对象，设置对象属性，查询对象都需要``NSManagedObjectContext``。



创建``NSManagedObjectContext``的两个不同的方法：



1.通过其自身的初始化：

``
[NSManagedObjectContext alloc] init];
``

2.通过UIManagedDocument创建：

UIManagedDocument 用于管理存储的机制，将Core Data数据库放入某存储空间。
``
UIManagedDocument *document = [[UIManagedDocument alloc] initWithFileURL:url];
//url:这个core data 数据库存储的地方
``



## 数据库的操作：

#### 1. 向数据库添加对象（实体）：

``
[NSEntityDescription insertNewObjectForEntityForName:@"Photo" inManagedObjectContext:context];
``



#### 2. 从数据库删除对象（实体）：

``
[aDocument.managedObjectContext deleteObject:photo];
``


删除对象后，系统会向所有对象发送这个消息

``
- (void)prepareForDeletion{
   //在这里保持数据同步，比如删掉这个对象的时候会影响到其他对象的数据
    //应该在这个对象被删除前及时更新那个数据
   }

``

#### 3. 在数据库查询对象（实体）：


我们使用``NSFetchRequest``类查询数据库的对象，通过设置其不同属性来查找符合不同标准的数据：

举个🌰 ：查找出100个photo的实体：

```
NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Photo"];
request.fetchLimit = 100;
```



## 关于Core data的线程安全

```
//让context在安全队列中执行的方法
[context performBlock:^{
    [A doSomething];
}];
```





# NSFetchedResultsController

------


NSFetchedResultsController的作用是将NSFetchRequest 和 UITalbleView联系到一起。
和TableView的数据源方法类似：


```
- (NSInteger)numberOfRowsInSection:(NSInteger)section{
  return [[self.fetchedResultsController sections] count];
}

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
   return [[self.fetchedResultsController sections] count] objectAtIndex:section] numberOfObjects];
}
```

详细的使用方法会在Demo讲解部分中告诉大家。



# Demo
-----



## Demo需求

- 每隔20分钟，从flickr拿回最新的摄影者数据。
- 用一个TableView显示当前拿回的摄影者的名字和所照的照片数。



## Demo效果图


![摄影师的信息列表](http://upload-images.jianshu.io/upload_images/859001-72dc6873f1c13917.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 重要代码段

#### 1. 在启动接口获取flickr的数组

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
     self.photoDatabaseContext = [self createMainQueueManagedObjectContext];    
     [self startFlickrFetch];   
     return YES;
}

- (void)startFlickrFetch
{
    [self.flickrDownloadSession getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        if (![downloadTasks count]) {
            NSURLSessionDownloadTask *task = [self.flickrDownloadSession downloadTaskWithURL:[FlickrFetcher URLforRecentGeoreferencedPhotos]];
            task.taskDescription = FLICKR_FETCH;
            [task resume];
        } else {
            for (NSURLSessionDownloadTask *task in downloadTasks) [task resume];
        }
    }];
}
```



#### 2. 每隔20分钟获取新的内容

```
- (void)setPhotoDatabaseContext:(NSManagedObjectContext *)photoDatabaseContext
{
    _photoDatabaseContext = photoDatabaseContext;
    
    //photoDatabaseContext设定成功后，每隔20分钟重新获取信息
    if (self.photoDatabaseContext)
    {
        self.flickrForegroundFetchTimer = [NSTimer scheduledTimerWithTimeInterval:FOREGROUND_FLICKR_FETCH_INTERVAL
                                                                           target:self
                                                                         selector:@selector(startFlickrFetch:)
                                                                         userInfo:nil
                                                                          repeats:YES];
    }
    
    //photoDatabaseContext设定成功后 向控制器发送消息
    NSDictionary *userInfo = self.photoDatabaseContext ? @{ PhotoDatabaseAvailabilityContext : self.photoDatabaseContext } : nil;
    [[NSNotificationCenter defaultCenter] postNotificationName:PhotoDatabaseAvailabilityNotification
                                                        object:self
                                                      userInfo:userInfo];
}
```

#### 3. 在表格视图查询所有摄影师的名字

```
- (void)setManagedObjectContext:(NSManagedObjectContext *)managedObjectContext
{
    //哪个数据库
    _managedObjectContext = managedObjectContext;
    
    NSFetchRequest *requet = [NSFetchRequest fetchRequestWithEntityName:@"Photographer"];
    requet.predicate = nil;//所有的,无过滤
    requet.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES selector:@selector(localizedStandardCompare:)]];

    self.fetchedResultsController = [[NSFetchedResultsController alloc] initWithFetchRequest:requet managedObjectContext:managedObjectContext sectionNameKeyPath:nil cacheName:nil];
    
}
```



#### 4. 重写``tablelViwe:cellForRowAtIndex:``方法，显示摄影师数据

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Photographer cell"];
    
    //拿到摄影师的名字和摄影数量
    Photographer *photographer = [self.fetchedResultsController objectAtIndexPath:indexPath];
    cell.textLabel.text = photographer.name;
    cell.detailTextLabel.text = [NSString stringWithFormat:@"%lu photos", [photographer.photos count]];    
    return cell;
    
}
```



本Demo显然是一个未完成品，它只显示了摄影师的相关信息，并且只有一个页面。在接下来的课程中应该会对该Demo进行更多过的扩展。

# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，可以进入我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)下载哦~    这一系列到现在为止的所有Demo都在里面，分为英文注释版本和中文注释版本两种。

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~


