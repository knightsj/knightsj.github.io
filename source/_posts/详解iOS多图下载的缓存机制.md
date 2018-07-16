---
title: 详解iOS多图下载的缓存机制
tags: [iOS,Objective-C]
categories: iOS
---



做iOS开发也有半年多了，想想自己对一些第三方库还只是停留在简单运用的阶段，感觉心慌慌的。于是决定用一个月的时间深入了解一些好的第三方库。

第一个想到了SDWebImage，这个库很不错，几乎每个iOS项目都会有它的影子，因为它很完美地解决了下载图片并显示的处理逻辑。那么深究它之前，笔者准备先了解一下多图下载的缓存机制，因为它和SDWebImage的方案类似。

有一个多图缓存机制的教程是来自李明杰小码哥的，笔者觉得讲得挺不错的，于是就花了2个小时好好学习了一下。

<!-- more -->


# 1.需求点是什么？
-----
这里所说的**多图下载**，就是要在tableview的每一个cell里显示一张图片,而且这些图片都需要从网上下载。

# 2.容易遇到的问题
----
如果不知道或不使用**异步操作**和**缓存机制**，那么写出来的代码很可能会是这样：

```objc
cell.textLabel.text = app.name;
cell.detailTextLabel.text = app.download;
NSData *imageData = [NSData dataWithContentsOfURL:app.url];
cell.imageView.image = [UIImage imageWithData:imageData];
```

这样写有什么后果呢？

#### 后果1：不可避免的卡顿（因为没有异步下载操作）

>dataWithContentsOfURL：是耗时操作，将其放在主线程会造成卡顿。如果图片很多，图片很大，而且网络情况不好的话肯定会卡出翔！

#### 后果2：同一图片重复下载，耗费流量和系统开销（因为没有建立缓存机制）
>由于没有缓存机制，即使下载完成并显示了当前cell的图片，但是当该cell再一次需要显示的时候还是会下载它所对应的图片：耗费了下载流量，而且还导致重复操作。

很显然，要达到Tableview滚动的**如丝滑般的享受**必须二者兼得才可以，具体怎么做呢？

# 3.解决方案
------

#### 1.先看一下解决方案的流程图   

小码哥将他的解决方案在PPT里用流程图画了出来，笔者觉得很不错，但是颜值略低（毕竟人家是一心搞技术，没时间在意这些外在的东西），笔者理了理思路，自己重新画了一张（好看么？）：

![多图下载解决方案流程图](http://upload-images.jianshu.io/upload_images/859001-addf3137097c3912.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要想快速看懂此图，需要先了解该流程所需的所有数据源：

**1. 图片的URL**：因为每张图片对应的URL都是唯一的，所以我们可以通过它来建立**图片缓存**和**下载操作的缓存**的键，以及拼接**沙盒缓存**的路径字符串。
**2. 图片缓存（字典）**：存放于内存中；键为图片的URL，值为UIImage对象。作用：读取速度快，直接使用UIImage对象。
**3. 下载操作缓存（字典）**：存放与内存中，键为图片的URL，值为NSBlockOperation对象。作用：用来避免对于同一张图片还要开启多个下载线程。
**4. 沙盒缓存(文件路径对应NSData)**：存放于磁盘中，位于Cache文件夹内，路径为“Cache/图片URL的最后的部分”，值为NSData对象（将UIImage转化为NSData才能写入磁盘里）。作用：程序断网，再次启动也可以直接在磁盘中拿到图片。


#### 2.再看一下解决方案的代码


**2.1图片缓存，下载操作缓存，沙盒缓存路径**
```objc
/**
 *  存放所有下载完的图片
 */
@property (nonatomic, strong) NSMutableDictionary *images;

/**
 *  存放所有的下载操作（key是url，value是operation对象）
 */
@property (nonatomic, strong) NSMutableDictionary *operations;

/**
 *  拼接Cache文件夹的路径与url最后的部分，合并成唯一约定好的缓存路径
 */
#define CachedImageFile(url) [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:[url lastPathComponent]]
```


**2.2 图片下载之前的查询缓存部分**：

```objc
    // 先从images缓存中取出图片url对应的UIImage
    UIImage *image = self.images[app.icon];
    if (image) { 

        // 存在：说明图片已经下载成功，并缓存成功）
        cell.imageView.image = image;

    } else { 

         // 不存在：说明图片并未下载成功过，或者成功下载但是在images里缓存失败，需要在沙盒里寻找对于的图片

         // 获得url对于的沙盒缓存路径
        NSString *file = CachedImageFile(app.icon);
        
        // 先从沙盒中取出图片
        NSData *data = [NSData dataWithContentsOfFile:file];

        if (data) {
    
            //data不为空，说明沙盒中存在这个文件
            cell.imageView.image = [UIImage imageWithData:data];

        } else {

             // 反之沙盒中不存在这个文件
             // 在下载之前显示占位图片
            cell.imageView.image = [UIImage imageNamed:@"placeholder"];
            
            // 下载图片
            [self download:app.icon indexPath:indexPath];
        }
    }
```

**2.3 图片的下载部分**：

```objc
/**
 *  下载图片
 *
 *  @param imageUrl 图片的url
 */
- (void)download:(NSString *)imageUrl indexPath:(NSIndexPath *)indexPath
{
    // 取出当前图片url对应的下载操作（operation对象）
    NSBlockOperation *operation = self.operations[imageUrl];
    if (operation) return;
    
    // 创建操作，下载图片
    __weak typeof(self) appsVc = self;
    operation = [NSBlockOperation blockOperationWithBlock:^{
        NSURL *url = [NSURL URLWithString:imageUrl];
        NSData *data = [NSData dataWithContentsOfURL:url]; // 下载
        UIImage *image = [UIImage imageWithData:data]; // NSData -> UIImage
        
        // 回到主线程
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                     
            if (image) {
                // 如果存在图片（下载完成），存放图片到图片缓存字典中
                appsVc.images[imageUrl] = image;
                
                //将图片存入沙盒中
                //1. 先将图片转化为NSData
                NSData *data = UIImagePNGRepresentation(image);
                
                //2.  再生成缓存路径            
                [data writeToFile:CachedImageFile(imageUrl) atomically:YES];
            }
            
            // 从字典中移除下载操作 (保证下载失败后，能重新下载)
            [appsVc.operations removeObjectForKey:imageUrl];
            
            // 刷新当前表格，减少系统开销
            [appsVc.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationNone];
        }];
    }];
    
    // 添加下载操作到队列中
    [self.queue addOperation:operation];
    
    // 将当前下载操作添加到下载操作缓存中 (为了解决重复下载)
    self.operations[imageUrl] = operation;
}
```

#### 3. 有哪些点是值得注意的？

要说值得注意的地方，还是离不开对于缓存内容的添加和删除操作。

**3.1 关于图片缓存**：
很简单，成功下载，拿到了图片，就将图片添加到图片缓存中；下载失败，什么都不做，反正没有图。在这种机制下，就没有删除缓存里某个图片项的情况，因为图片缓存永远不会出现重复添加多个相同图片的情况，缓存中只要有一张对应的图，就直接拿去用了，不会去再下载了。

**3.2 关于沙盒缓存**：
同样地，对于沙盒缓存也是一个道理：有图就将其转化为NSData，写入磁盘，并对应唯一的路径，没有图就不写。所以即使是要下载相同的图片，因为当前url对应的沙盒路径已经存在文件了，所以直接拿就可以了，不会再下载。

但是！
下载操作缓存是不同的！

**3.3 关于下载操作缓存**
我们需要在下载回调完成后，立即将当前的下载操作从下载操作缓存中删去！
因为要避免下载失败后，无法再次下载的情况的发生！

为什么呢？
注意一下将下载操作加入到下载操作缓存的时机：
是在**下载开始的那一刻**而不是**下载成功的那一刻**！

如果在下载开始的那一刻加入到缓存中的话，这个缓存信息就包括两个情况：下载成功和下载失败：

- 如果未来下载成功了，那么我们就不会来到判断当前下载操作是否在下载操作缓存这一步，在这之前直接就可以拿图去用了，下载操作是否存在下载操作缓存里并没有什么影响。

- 但是！如果未来下载失败了，那就肯定不会有对应的图片缓存和沙盒缓存，也就肯定会来到判断当前的下载操作是否在下载操作缓存里这一步。不幸的是，因为没有被删去，它是存在的。存在的话就不做任何其他操作，放任自流，导致曾经下载失败的图片永远不会再次下载。

忘了那段代码了么？回看一下代码（看我多好）：
```objc
NSBlockOperation *operation = self.operations[imageUrl];
 if (operation) return;//转身就走，毫不留情
```

因此，无论下载成功或是失败，在图片下载的回调里都要将当前的下载操作从下载操作队列中移走：用来保证如果下载失败了，就可以重新开启对应的下载操作进行下载，逻辑上更加严谨。

# 4.最后的话
------

异步+缓存这两个机制双剑合璧的话会对程序新能带来很大的改观。这应该app开发进阶的必经之路。

小码哥讲述的这套流程还算比较完整的了，更重要的还是学习其中的思想：

>1. 将缓存分级：内存缓存，沙盒缓存，下载操作缓存。

>2. 而且还要经常使用二分法，将我们的逻辑考虑得滴水不漏。
>  如果我们没有认识到将下载操作添加到下载操作缓存的时机是包含下载成功和下载失败两个情况，那么就不会考虑到即时要将下载操作从下载操作缓存中删去的操作，很容易引起bug。所以在以后的开发中，成功和失败两个情况都要考虑进去，也就是说有if一定要有else！




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



