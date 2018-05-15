---
title: YYCache 源码解析
tags: [iOS,Objective-C,源码解析]
categories: iOS
---

[YYCache](https://github.com/ibireme/YYCache)是国内开发者[ibireme](https://blog.ibireme.com/)开源的一个线程安全的高性能键值缓存组件，代码风格简洁清晰，在GitHub上已经有了1600+颗星。

阅读它的源码有助于建立比较完整的缓存设计的思路，同时也能巩固一下双向链表，线程锁，数据库操作相关的知识。如果你还没有看过YYCache的源码，那么恭喜你，阅读此文会对理解YYCache的源码有比较大的帮助。

<!-- more -->

在正式开始讲解源码之前，先简单看一下该框架的使用方法。


# 基本使用方法

举一个缓存用户姓名的例子来看一下YYCache的几个API：

```objc
    //需要缓存的对象
    NSString *userName = @"Jack";
   
   //需要缓存的对象在缓存里对应的键
    NSString *key = @"user_name";
    
    //创建一个YYCache实例:userInfoCache
    YYCache *userInfoCache = [YYCache cacheWithName:@"userInfo"];
    
    //存入键值对
    [userInfoCache setObject:userName forKey:key withBlock:^{
        NSLog(@"caching object succeed");
    }];
    
    //判断缓存是否存在
    [userInfoCache containsObjectForKey:key withBlock:^(NSString * _Nonnull key, BOOL contains) {
        if (contains){
            NSLog(@"object exists");
        }
    }];

    //根据key读取数据
    [userInfoCache objectForKey:key withBlock:^(NSString * _Nonnull key, id<NSCoding>  _Nonnull object) {
        NSLog(@"user name : %@",object);
    }];

    //根据key移除缓存
    [userInfoCache removeObjectForKey:key withBlock:^(NSString * _Nonnull key) {
        NSLog(@"remove user name %@",key);
    }];
    
    //移除所有缓存
    [userInfoCache removeAllObjectsWithBlock:^{
        NSLog(@"removing all cache succeed");
    }];

    //移除所有缓存带进度
    [userInfoCache removeAllObjectsWithProgressBlock:^(int removedCount, int totalCount) {
        NSLog(@"remove all cache objects: removedCount :%d  totalCount : %d",removedCount,totalCount);
    } endBlock:^(BOOL error) {
        if(!error){
            NSLog(@"remove all cache objects: succeed");
        }else{
            NSLog(@"remove all cache objects: failed");
        }
    }];
```

总体来看这些API与``NSCache``是差不多的。
下面接着看一下该框架的架构：

# 架构与职责划分

首先看一下架构图：


## 架构图

![](https://user-gold-cdn.xitu.io/2018/1/22/1611c66be5bd7907?w=604&h=344&f=png&s=21874)

## 职责划分

从架构图上来看，该组件里面的成员并不多：

- YYCache：提供了最外层的接口，调用了YYMemoryCache与YYDiskCache的相关方法。
- YYMemoryCache：负责处理容量小，相对高速的内存缓存。线程安全，支持自动和手动清理缓存等功能。
- _YYLinkedMap：YYMemoryCache使用的双向链表类。
- _YYLinkedMapNode：是_YYLinkedMap使用的节点类。
- YYDiskCache：负责处理容量大，相对低速的磁盘缓存。线程安全，支持异步操作，自动和手动清理缓存等功能。
- YYKVStorage：YYDiskCache的底层实现类，用于管理磁盘缓存。
- YYKVStorageItem：内置在YYKVStorage中，是YYKVStorage内部用于封装某个缓存的类。

每个成员的详细的功能会在下文结合代码介绍。




# 代码讲解

知道了YYCache的架构和职责划分以后，现在结合代码开始正式讲解。
讲解分为下面6个部分：

- YYCache
- YYMemoryCache
- YYDiskCache
- 保证线程安全的不同方案
- 提高缓存性能的几个尝试
- 其他知识点


## YYCache

YYCache给用户提供所有最外层的缓存操作接口，而这些接口的内部内部实际上是调用了YYMemoryCache和YYDiskCache对象的相关方法。

>因为YYMemoryCache和YYDiskCache的实例作为YYCache的两个公开的属性，所以用户**无法直接使用YYMemoryCache和YYDiskCache对象**，只能通过属性的方式来间接使用它们。

我们来看一下YYCache的属性和接口：

### YYCache的属性和接口


```objc

@interface YYCache : NSObject


@property (copy, readonly) NSString *name;//缓存名称
@property (strong, readonly) YYMemoryCache *memoryCache;//内存缓存
@property (strong, readonly) YYDiskCache *diskCache;//磁盘缓存

//是否包含某缓存，无回调
- (BOOL)containsObjectForKey:(NSString *)key;
//是否包含某缓存，有回调
- (void)containsObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, BOOL contains))block;

//获取缓存对象，无回调
- (nullable id<NSCoding>)objectForKey:(NSString *)key;
//获取缓存对象，有回调
- (void)objectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, id<NSCoding> object))block;

//写入缓存对象，无回调
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key;
//写入缓存对象，有回调
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key withBlock:(nullable void(^)(void))block;

//移除某缓存，无回调
- (void)removeObjectForKey:(NSString *)key;
//移除某缓存，有回调
- (void)removeObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key))block;

//移除所有缓存，无回调
- (void)removeAllObjects;
//移除所有缓存，有回调
- (void)removeAllObjectsWithBlock:(void(^)(void))block;
//移除所有缓存，有进度和完成的回调
- (void)removeAllObjectsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                                 endBlock:(nullable void(^)(BOOL error))end;

@end
```

从上面的接口可以看出YYCache的接口和NSCache很相近，而且在接口上都区分了有无回调的功能。
下面结合代码看一下这些接口是如何实现的：

### YYCache的接口实现

下面省略了带有回调的接口，因为与无回调的接口非常接近。

```objc
- (BOOL)containsObjectForKey:(NSString *)key {
    
    //先检查内存缓存是否存在，再检查磁盘缓存是否存在
    return [_memoryCache containsObjectForKey:key] || [_diskCache containsObjectForKey:key];
}

- (id<NSCoding>)objectForKey:(NSString *)key {
    
    //首先尝试获取内存缓存，然后获取磁盘缓存
    id<NSCoding> object = [_memoryCache objectForKey:key];
    
    //如果内存缓存不存在，就会去磁盘缓存里面找：如果找到了，则再次写入内存缓存中；如果没找到，就返回nil
    if (!object) {
        object = [_diskCache objectForKey:key];
        if (object) {
            [_memoryCache setObject:object forKey:key];
        }
    }
    return object;
}


- (void)setObject:(id<NSCoding>)object forKey:(NSString *)key {
    //先写入内存缓存，后写入磁盘缓存
    [_memoryCache setObject:object forKey:key];
    [_diskCache setObject:object forKey:key];
}


- (void)removeObjectForKey:(NSString *)key {
    //先移除内存缓存，后移除磁盘缓存
    [_memoryCache removeObjectForKey:key];
    [_diskCache removeObjectForKey:key];
}

- (void)removeAllObjects {
    //先全部移除内存缓存，后全部移除磁盘缓存
    [_memoryCache removeAllObjects];
    [_diskCache removeAllObjects];
}

```

从上面的接口实现可以看出：在YYCache中，永远都是先访问内存缓存，然后再访问磁盘缓存（包括了写入，读取，查询，删除缓存的操作）。而且关于内存缓存（_memoryCache）的操作，是不存在block回调的。

现在了解了YYCache的接口以及实现，下面我分别讲解一下YYMemoryCache（内存缓存）和YYDiskCache（磁盘缓存）这两个类。

## YYMemoryCache

YYMemoryCache负责处理容量小，相对高速的内存缓存：它将需要缓存的对象与传入的key关联起来，操作类似于NSCache。

但是与NSCache不同的是，YYMemoryCache的内部有：

- 缓存淘汰算法：使用LRU(least-recently-used) 算法来淘汰（清理）使用频率较低的缓存。
- 缓存清理策略：使用三个维度来标记，分别是count（缓存数量），cost（开销），age（距上一次的访问时间）。YYMemoryCache提供了分别针对这三个维度的清理缓存的接口。用户可以根据不同的需求（策略）来清理在某一维度超标的缓存。

一个是淘汰算法，另一个是清理维度，乍一看可能没什么太大区别。我在这里先简单区分一下：

缓存淘汰算法的目的在于区分出使用频率高和使用频率低的缓存，当缓存数量达到一定限制的时候会优先清理那些使用频率低的缓存。**因为使用频率已经比较低的缓存在将来的使用频率也很有可能会低**。

缓存清理维度是给每个缓存添加的标记：

- 如果用户需要删除age（距上一次的访问时间）超过1天的缓存，在YYMemoryCache内部，就会从使用频率最低的那个缓存开始查找，直到所有距上一次的访问时间超过1天的缓存都清理掉为止。

- 如果用户需要将缓存总开销清理到总开销小于或等于某个值，在YYMemoryCache内部，就会从使用频率最低的那个缓存开始清理，直到总开销小于或等于这个值。

- 如果用户需要将缓存总数清理到总开销小于或等于某个值，在YYMemoryCache内部，就会从使用频率最低的那个缓存开始清理，直到总开销小于或等于这个值。

可以看出，无论是以哪个维度来清理缓存，都是从缓存使用频率最低的那个缓存开始清理。而YYMemoryCache保留的所有缓存的使用频率的高低，是由LRU这个算法决定的。

现在知道了这二者的区别，下面来具体讲解一下缓存淘汰算法和缓存清理策略：

### YYMemoryCache的缓存淘汰算法

在详细讲解这个算法之前我觉得有必要先说一下该算法的核心：

我个人认为LRU缓存替换策略的核心在于**如果某个缓存访问的频率越高，就认定用户在将来越有可能访问这个缓存**。
所以在这个算法中，将那些最新访问（写入），最多次被访问的缓存移到最前面，然后那些很早之前写入，不经常访问的缓存就被自动放在了后面。这样一来，在保留的缓存个数一定的情况下，留下的缓存都是访问频率比较高的，这样一来也就提升了缓存的命中率。谁都不想留着一些很难被用户再次访问的缓存，毕竟缓存本身也占有一定的资源不是么？

其实这个道理和一些商城类app的商品推荐逻辑是一样的：
如果首页只能展示10个商品，对于一个程序员用户来说，可能推荐的是于那些他最近购买商品类似的机械键盘鼠标，技术书籍或者显示屏之类的商品，而不是一些洋娃娃或是钢笔之类的商品。

那么LRU算法具体是怎么做的呢？

在YYMemoryCache中，使用了双向链表这个数据结构来保存这些缓存：

- 当写入一个新的缓存时，要把这个缓存节点放在链表头部，并且并且原链表头部的缓存节点要变成现在链表的第二个缓存节点。
- 当访问一个已有的缓存时，要把这个缓存节点移动到链表头部，原位置两侧的缓存要接上，并且原链表头部的缓存节点要变成现在链表的第二个缓存节点。
- （根据清理维度）自动清理缓存时，要从链表的最后端逐个清理。

这样一来，就可以保证链表前端的缓存是最近写入过和经常访问过的。而且该算法总是从链表的最后端删除缓存，这也就保证了留下的都是一些“比较新鲜的”缓存。

下面结合代码来讲解一下这个算法的实现：

YYMemoryCache**用一个链表节点类来保存某个单独的内存缓存的信息（键，值，缓存时间等），然后用一个双向链表类来保存和管理这些节点**。这两个类的名称分别是：


- _YYLinkedMapNode：链表内的节点类，可以看做是对某个单独内存缓存的封装。
- _YYLinkedMap：双向链表类，用于保存和管理所有内存缓存(节点)


#### _YYLinkedMapNode

_YYLinkedMapNode可以被看做是对某个缓存的封装：它包含了该节点上一个和下一个节点的指针，以及缓存的key和对应的值（对象），还有该缓存的开销和访问时间。


```objc
@interface _YYLinkedMapNode : NSObject {
    
    @package
    __unsafe_unretained _YYLinkedMapNode *_prev; // retained by dic
    __unsafe_unretained _YYLinkedMapNode *_next; // retained by dic
    id _key;              		  //缓存key
    id _value;              	          //key对应值
    NSUInteger _cost;                     //缓存开销
    NSTimeInterval _time;                 //访问时间
    
}
@end

@implementation _YYLinkedMapNode
@end
```

下面看一下双向链表类：

#### _YYLinkedMap

```objc
@interface _YYLinkedMap : NSObject {
    @package
    CFMutableDictionaryRef _dic; 	// 用于存放节点
    NSUInteger _totalCost;   		//总开销
    NSUInteger _totalCount;  		//节点总数
    _YYLinkedMapNode *_head;            // 链表的头部结点
    _YYLinkedMapNode *_tail; 		// 链表的尾部节点
    BOOL _releaseOnMainThread; 	        //是否在主线程释放，默认为NO
    BOOL _releaseAsynchronously; 	//是否在子线程释放，默认为YES
}

//在链表头部插入某节点
- (void)insertNodeAtHead:(_YYLinkedMapNode *)node;

//将链表内部的某个节点移到链表头部
- (void)bringNodeToHead:(_YYLinkedMapNode *)node;

//移除某个节点
- (void)removeNode:(_YYLinkedMapNode *)node;

//移除链表的尾部节点并返回它
- (_YYLinkedMapNode *)removeTailNode;

//移除所有节点（默认在子线程操作）
- (void)removeAll;

@end
```

从链表类的属性上看：链表类内置了CFMutableDictionaryRef，用于保存节点的键值对，它还持有了链表内节点的总开销，总数量，头尾节点等数据。



可以参考下面这张图来看一下二者的关系：

![](https://user-gold-cdn.xitu.io/2018/1/22/1611c66be5e027e5?w=1001&h=333&f=png&s=34319)



看一下_YYLinkedMap的接口的实现：

将节点插入到链表头部：

```objc
- (void)insertNodeAtHead:(_YYLinkedMapNode *)node {
    
    //设置该node的值
    CFDictionarySetValue(_dic, (__bridge const void *)(node->_key), (__bridge const void *)(node));
    
    //增加开销和总缓存数量
    _totalCost += node->_cost;
    _totalCount++;
    
    if (_head) {
        
        //如果链表内已经存在头节点，则将这个头节点赋给当前节点的尾指针（原第一个节点变成了现第二个节点）
        node->_next = _head;
        
        //将该节点赋给现第二个节点的头指针（此时_head指向的节点是先第二个节点）
        _head->_prev = node;
        
        //将该节点赋给链表的头结点指针（该节点变成了现第一个节点）
        _head = node;
        
    } else {
        
        //如果链表内没有头结点，说明是空链表。说明是第一次插入，则将链表的头尾节点都设置为当前节点
        _head = _tail = node;
    }
}
```

要看懂节点操作的代码只要了解双向链表的特性即可。在双向链表中：

- 每个节点都有两个分别指向前后节点的指针。所以说每个节点都知道它前一个节点和后一个节点是谁。
- 链表的头部节点指向它前面节点的指针为空；链表尾部节点指向它后侧节点的指针也为空。

为了便于理解，我们可以把这个抽象概念类比于幼儿园手拉手的小朋友们：
每个小朋友的左手都拉着前面小朋友的右手；每个小朋友的右手都拉着后面小朋友的左手；
而且最前面的小朋友的左手和最后面的小朋友的右手都没有拉任何一个小朋友。



将某个节点移动到链表头部：

```objc

- (void)bringNodeToHead:(_YYLinkedMapNode *)node {
    
    //如果该节点已经是链表头部节点，则立即返回，不做任何操作
    if (_head == node) return;
    
    
    if (_tail == node) {
        
        //如果该节点是链表的尾部节点
        //1. 将该节点的头指针指向的节点变成链表的尾节点（将倒数第二个节点变成倒数第一个节点，即尾部节点）
        _tail = node->_prev;
        
        //2. 将新的尾部节点的尾部指针置空
        _tail->_next = nil;
        
    } else {
        
        //如果该节点是链表头部和尾部以外的节点（中间节点）
        //1. 将该node的头指针指向的节点赋给其尾指针指向的节点的头指针
        node->_next->_prev = node->_prev;
        
        //2. 将该node的尾指针指向的节点赋给其头指针指向的节点的尾指针
        node->_prev->_next = node->_next;
    }
    
    //将原头节点赋给该节点的尾指针（原第一个节点变成了现第二个节点）
    node->_next = _head;
    
    //将当前节点的头节点置空
    node->_prev = nil;
    
    //将现第二个节点的头结点指向当前节点（此时_head指向的节点是现第二个节点）
    _head->_prev = node;
    
    //将该节点设置为链表的头节点
    _head = node;
}
```


第一次看上面的代码我自己是懵逼的，不过如果结合上面小朋友拉手的例子就可以快一点理解。
如果要其中一个小朋友放在队伍的最前面，需要

- 将原来这个小朋友前后的小朋友的手拉上。
- 然后将这个小朋友的右手和原来排在第一位的小朋友的左手拉上。

上面说的比较简略，但是相信对大家理解整个过程会有帮助。



也可以再结合链表的图解来看一下：

![](https://user-gold-cdn.xitu.io/2018/1/22/1611c66be5e027e5?w=1001&h=333&f=png&s=34319)

读者同样可以利用这种思考方式理解下面这段代码：


移除链表中的某个节点：

```objc
- (void)removeNode:(_YYLinkedMapNode *)node {
    
    //除去该node的键对应的值
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(node->_key));
    
    //减去开销和总缓存数量
    _totalCost -= node->_cost;
    _totalCount--;
    
    //节点操作
    //1. 将该node的头指针指向的节点赋给其尾指针指向的节点的头指针
    if (node->_next) node->_next->_prev = node->_prev;
    
    //2. 将该node的尾指针指向的节点赋给其头指针指向的节点的尾指针
    if (node->_prev) node->_prev->_next = node->_next;
    
    //3. 如果该node就是链表的头结点，则将该node的尾部指针指向的节点赋给链表的头节点（第二变成了第一）
    if (_head == node) _head = node->_next;
    
    //4. 如果该node就是链表的尾节点，则将该node的头部指针指向的节点赋给链表的尾节点（倒数第二变成了倒数第一）
    if (_tail == node) _tail = node->_prev;
}
```


移除并返回尾部的node:

```objc
- (_YYLinkedMapNode *)removeTailNode {
    
    //如果不存在尾节点，则返回nil
    if (!_tail) return nil;
    
    _YYLinkedMapNode *tail = _tail;
    
    //移除尾部节点对应的值
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(_tail->_key));
    
    //减少开销和总缓存数量
    _totalCost -= _tail->_cost;
    _totalCount--;
    
    if (_head == _tail) {
        //如果链表的头尾节点相同，说明链表只有一个节点。将其置空
        _head = _tail = nil;
        
    } else {
        
        //将链表的尾节指针指向的指针赋给链表的尾指针（倒数第二变成了倒数第一）
        _tail = _tail->_prev;
        //将新的尾节点的尾指针置空
        _tail->_next = nil;
    }
    return tail;
}
```

OK，现在了解了YYMemoryCache底层的节点操作的代码。现在来看一下YYMemoryCache是如何使用它们的。

#### YYMemoryCache的属性和接口

```objc
//YYMemoryCache.h
@interface YYMemoryCache : NSObject

#pragma mark - Attribute

//缓存名称，默认为nil
@property (nullable, copy) NSString *name;

//缓存总数量
@property (readonly) NSUInteger totalCount;

//缓存总开销
@property (readonly) NSUInteger totalCost;


#pragma mark - Limit

//数量上限，默认为NSUIntegerMax，也就是无上限
@property NSUInteger countLimit;

//开销上限，默认为NSUIntegerMax，也就是无上限
@property NSUInteger costLimit;

//缓存时间上限，默认为DBL_MAX，也就是无上限
@property NSTimeInterval ageLimit;

//清理超出上限之外的缓存的操作间隔时间，默认为5s
@property NSTimeInterval autoTrimInterval;

//收到内存警告时是否清理所有缓存，默认为YES
@property BOOL shouldRemoveAllObjectsOnMemoryWarning;

//app进入后台是是否清理所有缓存，默认为YES
@property BOOL shouldRemoveAllObjectsWhenEnteringBackground;

//收到内存警告的回调block
@property (nullable, copy) void(^didReceiveMemoryWarningBlock)(YYMemoryCache *cache);

//进入后台的回调block
@property (nullable, copy) void(^didEnterBackgroundBlock)(YYMemoryCache *cache);

//缓存清理是否在后台进行，默认为NO
@property BOOL releaseOnMainThread;

//缓存清理是否异步执行，默认为YES
@property BOOL releaseAsynchronously;


#pragma mark - Access Methods

//是否包含某个缓存
- (BOOL)containsObjectForKey:(id)key;

//获取缓存对象
- (nullable id)objectForKey:(id)key;

//写入缓存对象
- (void)setObject:(nullable id)object forKey:(id)key;

//写入缓存对象，并添加对应的开销
- (void)setObject:(nullable id)object forKey:(id)key withCost:(NSUInteger)cost;

//移除某缓存
- (void)removeObjectForKey:(id)key;

//移除所有缓存
- (void)removeAllObjects;

#pragma mark - Trim

// =========== 缓存清理接口 =========== 
//清理缓存到指定个数
- (void)trimToCount:(NSUInteger)count;

//清理缓存到指定开销
- (void)trimToCost:(NSUInteger)cost;

//清理缓存时间小于指定时间的缓存
- (void)trimToAge:(NSTimeInterval)age;
```

#### YYMemoryCache的接口实现

在YYMemoryCache的初始化方法里，会实例化一个_YYLinkedMap的实例来赋给_lru这个成员变量。

```objc

- (instancetype)init{
      ....
      _lru = [_YYLinkedMap new];
      ...
  
  }
  
```

  然后所有的关于缓存的操作，都要用到_lru这个成员变量，因为它才是在底层持有这些缓存（节点）的双向链表类。下面我们来看一下这些缓存操作接口的实现：

  ```objc

//是否包含某个缓存对象
- (BOOL)containsObjectForKey:(id)key {

    //尝试从内置的字典中获得缓存对象
    if (!key) return NO;
    pthread_mutex_lock(&_lock);
    BOOL contains = CFDictionaryContainsKey(_lru->_dic, (__bridge const void *)(key));
    pthread_mutex_unlock(&_lock);
    return contains;
}

//获取某个缓存对象
- (id)objectForKey:(id)key {
    
    if (!key) return nil;
    
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
        //如果节点存在，则更新它的时间信息（最后一次访问的时间）
        node->_time = CACurrentMediaTime();
        [_lru bringNodeToHead:node];
    }
    pthread_mutex_unlock(&_lock);
    
    return node ? node->_value : nil;
}

//写入某个缓存对象，开销默认为0
- (void)setObject:(id)object forKey:(id)key {
    [self setObject:object forKey:key withCost:0];
}


//写入某个缓存对象，并存入缓存开销
- (void)setObject:(id)object forKey:(id)key withCost:(NSUInteger)cost {
    
    if (!key) return;
    
    if (!object) {
        [self removeObjectForKey:key];
        return;
    }
    
    pthread_mutex_lock(&_lock);
    
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    NSTimeInterval now = CACurrentMediaTime();
    
    if (node) {
        //如果存在与传入的key值匹配的node，则更新该node的value,cost,time，并将这个node移到链表头部
        
        //更新总cost
        _lru->_totalCost -= node->_cost;
        _lru->_totalCost += cost;
        
        //更新node
        node->_cost = cost;
        node->_time = now;
        node->_value = object;
        
        //将node移动至链表头部
        [_lru bringNodeToHead:node];
        
    } else {
        
        //如果不存在与传入的key值匹配的node，则新建一个node，将key,value,cost,time赋给它，并将这个node插入到链表头部
        //新建node,并赋值
        node = [_YYLinkedMapNode new];
        node->_cost = cost;
        node->_time = now;
        node->_key = key;
        node->_value = object;
        
        //将node插入至链表头部
        [_lru insertNodeAtHead:node];
    }
    
    //如果cost超过了限制，则进行删除缓存操作（从链表尾部开始删除，直到符合限制要求）
    if (_lru->_totalCost > _costLimit) {
        dispatch_async(_queue, ^{
            [self trimToCost:_costLimit];
        });
    }
    
    //如果total count超过了限制，则进行删除缓存操作（从链表尾部开始删除，删除一次即可）
    if (_lru->_totalCount > _countLimit) {
        _YYLinkedMapNode *node = [_lru removeTailNode];
        if (_lru->_releaseAsynchronously) {
            dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                [node class]; //hold and release in queue
            });
        } else if (_lru->_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [node class]; //hold and release in queue
            });
        }
    }
    pthread_mutex_unlock(&_lock);
}

//移除某个缓存对象
- (void)removeObjectForKey:(id)key {
    
    if (!key) return;
    
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
    
        //内部调用了链表的removeNode：方法
        [_lru removeNode:node];
        if (_lru->_releaseAsynchronously) {
            dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                [node class]; //hold and release in queue
            });
        } else if (_lru->_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [node class]; //hold and release in queue
            });
        }
    }
    pthread_mutex_unlock(&_lock);
}


//内部调用了链表的removeAll方法
- (void)removeAllObjects {
    pthread_mutex_lock(&_lock);
    [_lru removeAll];
    pthread_mutex_unlock(&_lock);
}
  ```

上面的实现是针对缓存的查询，写入，获取操作的，接下来看一下缓存的清理策略。

### YYMemoryCache的缓存清理策略

如上文所说，在YYCache中，缓存的清理可以从缓存总数量，缓存总开销，缓存距上一次的访问时间来清理缓存。而且每种维度的清理操作都可以分为自动和手动的方式来进行。

#### 缓存自动清理

缓存的自动清理功能在YYMemoryCache初始化之后就开始了，是一个递归调用的实现：


```objc
//YYMemoryCache.m
- (instancetype)init{
    
    ...
    
    //开始定期清理
    [self _trimRecursively];
    
    ...
}


//递归清理，相隔时间为_autoTrimInterval，在初始化之后立即执行
- (void)_trimRecursively {
    
    __weak typeof(self) _self = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(_autoTrimInterval * NSEC_PER_SEC)),
                   
        dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
            
        __strong typeof(_self) self = _self;
        if (!self) return;
        
        //在后台进行清理操作
        [self _trimInBackground];
        
        //调用自己，递归操作
        [self _trimRecursively];
            
    });
}

//清理所有不符合限制的缓存，顺序为：cost，count，age
- (void)_trimInBackground {
    dispatch_async(_queue, ^{
        
        [self _trimToCost:self->_costLimit];
        [self _trimToCount:self->_countLimit];
        [self _trimToAge:self->_ageLimit];
        
    });
}

```

```objc
//YYMemoryCache.m
- (void)trimToCount:(NSUInteger)count {
    if (count == 0) {
        [self removeAllObjects];
        return;
    }
    [self _trimToCount:count];
}

- (void)trimToCost:(NSUInteger)cost {
    [self _trimToCost:cost];
}

- (void)trimToAge:(NSTimeInterval)age {
    [self _trimToAge:age];
}
```

可以看到，YYMemoryCache是按照缓存数量，缓存开销，缓存时间的顺序来自动清空缓存的。我们结合代码看一下它是如何按照缓存数量来清理缓存的（其他两种清理方式类似，暂不给出）：

```objc
//YYMemoryCache.m

//将内存缓存数量降至等于或小于传入的数量；如果传入的值为0，则删除全部内存缓存
- (void)_trimToCount:(NSUInteger)countLimit {
    
    BOOL finish = NO;
    
    pthread_mutex_lock(&_lock);
    
    //如果传入的参数=0，则删除所有内存缓存
    if (countLimit == 0) {
        
        [_lru removeAll];
        finish = YES;
        
    } else if (_lru->_totalCount <= countLimit) {
    
        //如果当前缓存的总数量已经小于或等于传入的数量，则直接返回YES，不进行清理
        finish = YES;
    }
    pthread_mutex_unlock(&_lock);
    if (finish) return;
    
    NSMutableArray *holder = [NSMutableArray new];
    while (!finish) {
        
        //==0的时候说明在尝试加锁的时候，获取锁成功，从而可以进行操作；否则等待10秒（但是不知道为什么是10s而不是2s，5s，等等）
        if (pthread_mutex_trylock(&_lock) == 0) {
            if (_lru->_totalCount > countLimit) {
                _YYLinkedMapNode *node = [_lru removeTailNode];
                if (node) [holder addObject:node];
            } else {
                finish = YES;
            }
            pthread_mutex_unlock(&_lock);
        } else {
            usleep(10 * 1000); //10 ms
        }
    }
    if (holder.count) {
        dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
        dispatch_async(queue, ^{
            [holder count]; // release in queue
        });
    }
}
```

#### 缓存手动清理

其实上面这三种清理的方法在YYMemoryCache封装成了接口，所以用户也可以通过YYCache的memoryCache这个属性来手动清理相应维度上不符合传入标准的缓存：

```objc
//YYMemoryCache.h

// =========== 缓存清理接口 =========== 
//清理缓存到指定个数
- (void)trimToCount:(NSUInteger)count;

//清理缓存到指定开销
- (void)trimToCost:(NSUInteger)cost;

//清理缓存时间小于指定时间的缓存
- (void)trimToAge:(NSTimeInterval)age;
```

看一下它们的实现：

```objc
//清理缓存到指定个数
- (void)trimToCount:(NSUInteger)count {
    if (count == 0) {
        [self removeAllObjects];
        return;
    }
    [self _trimToCount:count];
}

//清理缓存到指定开销
- (void)trimToCost:(NSUInteger)cost {
    [self _trimToCost:cost];
}

//清理缓存时间小于指定时间的缓存
- (void)trimToAge:(NSTimeInterval)age {
    [self _trimToAge:age];
}
```


## YYDiskCache

YYDiskCache负责处理容量大，相对低速的磁盘缓存。线程安全，支持异步操作。作为YYCache的第二级缓存，它与第一级缓存YYMemoryCache的相同点是：

- 都具有查询，写入，读取，删除缓存的接口。
- 不直接操作缓存，也是间接地通过另一个类（YYKVStorage）来操作缓存。
- 它使用LRU算法来清理缓存。
- 支持按 cost，count 和 age 这三个维度来清理不符合标准的缓存。


它与YYMemoryCache不同点是：

- 1. 根据缓存数据的大小来采取不同的形式的缓存：
  - 数据库sqlite: 针对小容量缓存，缓存的data和元数据都保存在数据库里。
  - 文件+数据库的形式: 针对大容量缓存，缓存的data写在文件系统里，其元数据保存在数据库里。
- 2. 除了 cost，count 和 age 三个维度之外，还添加了一个磁盘容量的维度。


这里需要说明的是：
对于上面的第一条：我看源码的时候只看出来有这两种缓存形式，但是从内部的缓存type枚举来看，其实是分为三种的：

```objc
typedef NS_ENUM(NSUInteger, YYKVStorageType) {
    
    YYKVStorageTypeFile = 0,
    YYKVStorageTypeSQLite = 1,
    YYKVStorageTypeMixed = 2,
};
```

也就是说我只找到了第二，第三种缓存形式，而第一种纯粹的文件存储（YYKVStorageTypeFile）形式的实现我没有找到：当type为
YYKVStorageTypeFile和YYKVStorageTypeMixed的时候的缓存实现都是一致的：都是讲data存在文件里，将元数据放在数据库里面。

在YYDiskCache的初始化方法里，没有发现正确的将缓存类型设置为YYKVStorageTypeFile的方法：

```objc
//YYDiskCache.m

- (instancetype)init {
    @throw [NSException exceptionWithName:@"YYDiskCache init error" reason:@"YYDiskCache must be initialized with a path. Use 'initWithPath:' or 'initWithPath:inlineThreshold:' instead." userInfo:nil];
    return [self initWithPath:@"" inlineThreshold:0];
}

- (instancetype)initWithPath:(NSString *)path {
    return [self initWithPath:path inlineThreshold:1024 * 20]; // 20KB
}

- (instancetype)initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold {

   ...    
    YYKVStorageType type;
    if (threshold == 0) {
        type = YYKVStorageTypeFile;
    } else if (threshold == NSUIntegerMax) {
        type = YYKVStorageTypeSQLite;
    } else {
        type = YYKVStorageTypeMixed;
    }
    
   ...
}

```

从上面的代码可以看出来，当给指定初始化方法``initWithPath:inlineThreshold:``的第二个参数传入0的时候，缓存类型才是YYKVStorageTypeFile。而且比较常用的初始化方法``initWithPath:``的实现里，是将20kb传入了指定初始化方法里，结果就是将type设置成了YYKVStorageTypeMixed。

而且我也想不出如果只有文件形式的缓存的话，其元数据如何保存。如果有读者知道的话，麻烦告知一下，非常感谢了~~

在本文暂时对于上面提到的”文件+数据库的形式”在下文统一说成文件缓存了。


在接口的设计上，YYDiskCache与YYMemoryCache是高度一致的，只不过因为有些时候大文件的访问可能会比较耗时，所以框架作者在保留了与YYMemoryCache一样的接口的基础上，还在原来的基础上添加了block回调，避免阻塞线程。来看一下YYDiskCache的接口(省略了注释)：

```objc
//YYDiskCache.h

- (BOOL)containsObjectForKey:(NSString *)key;
- (void)containsObjectForKey:(NSString *)key withBlock:(void(^)(NSString *key, BOOL contains))block;


- (nullable id<NSCoding>)objectForKey:(NSString *)key;
- (void)objectForKey:(NSString *)key withBlock:(void(^)(NSString *key, id<NSCoding> _Nullable object))block;


- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key;
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key withBlock:(void(^)(void))block;


- (void)removeObjectForKey:(NSString *)key;
- (void)removeObjectForKey:(NSString *)key withBlock:(void(^)(NSString *key))block;


- (void)removeAllObjects;
- (void)removeAllObjectsWithBlock:(void(^)(void))block;
- (void)removeAllObjectsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                                 endBlock:(nullable void(^)(BOOL error))end;


- (NSInteger)totalCount;
- (void)totalCountWithBlock:(void(^)(NSInteger totalCount))block;


- (NSInteger)totalCost;
- (void)totalCostWithBlock:(void(^)(NSInteger totalCost))block;


#pragma mark - Trim
- (void)trimToCount:(NSUInteger)count;
- (void)trimToCount:(NSUInteger)count withBlock:(void(^)(void))block;


- (void)trimToCost:(NSUInteger)cost;
- (void)trimToCost:(NSUInteger)cost withBlock:(void(^)(void))block;

- (void)trimToAge:(NSTimeInterval)age;
- (void)trimToAge:(NSTimeInterval)age withBlock:(void(^)(void))block;
```

从上面的接口代码可以看出，YYDiskCache与YYMemoryCache在接口设计上是非常相似的。但是，YYDiskCache有一个非常重要的属性，它**作为用sqlite做缓存还是用文件做缓存的分水岭**：

```objc
//YYDiskCache.h
@property (readonly) NSUInteger inlineThreshold;
```

这个属性的默认值是20480byte，也就是20kb。即是说，如果缓存数据的长度大于这个值，就使用文件存储；如果小于这个值，就是用sqlite存储。来看一下这个属性是如何使用的：

首先我们会在YYDiskCache的指定初始化方法里看到这个属性：

```objc
//YYDiskCache.m
- (instancetype)initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold {
   ...
    _inlineThreshold = threshold;
    ...
}
```

在这里将_inlineThreshold赋值，也是唯一一次的赋值。然后在写入缓存的操作里判断写入缓存的大小是否大于这个临界值，如果是，则使用文件缓存：

```objc
//YYDiskCache.m
- (void)setObject:(id<NSCoding>)object forKey:(NSString *)key {
   
   ...
    NSString *filename = nil;
    if (_kv.type != YYKVStorageTypeSQLite) {
        //如果长度大临界值，则生成文件名称，使得filename不为nil
        if (value.length > _inlineThreshold) {
            filename = [self _filenameForKey:key];
        }
    }
    
    Lock();
    //在该方法内部判断filename是否为nil，如果是，则使用sqlite进行缓存；如果不是，则使用文件缓存
    [_kv saveItemWithKey:key value:value filename:filename extendedData:extendedData];
    Unlock();
}
```

现在我们知道了YYDiskCache相对于YYMemoryCache最大的不同之处是缓存类型的不同。
细心的朋友会发现上面这个写入缓存的方法（saveItemWithKey:value:filename:extendedData:）实际上是属于_kv的。这个_kv就是上面提到的YYKVStorage的实例，它在YYDiskCache的初始化方法里被赋值：

```objc
//YYDiskCache.m
- (instancetype)initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold {
    ...
    
    YYKVStorage *kv = [[YYKVStorage alloc] initWithPath:path type:type];
    if (!kv) return nil;
    _kv = kv;
    ...
}
```

同样地，再举其他两个接口为例，内部也是调用了_kv的方法：

```objc
- (BOOL)containsObjectForKey:(NSString *)key {
    if (!key) return NO;
    Lock();
    BOOL contains = [_kv itemExistsForKey:key];
    Unlock();
    return contains;
}

- (void)removeObjectForKey:(NSString *)key {
    if (!key) return;
    Lock();
    [_kv removeItemForKey:key];
    Unlock();
} 
```

所以是时候来看一下YYKVStorage的接口和实现了：

### YYKVStorage

YYKVStorage实例负责保存和管理所有磁盘缓存。和YYMemoryCache里面的_YYLinkedMap将缓存封装成节点类_YYLinkedMapNode类似，YYKVStorage也将某个单独的磁盘缓存封装成了一个类，这个类就是YYKVStorageItem，它保存了某个缓存所对应的一些信息(key, value, 文件名，大小等等)：

```objc
//YYKVStorageItem.h

@interface YYKVStorageItem : NSObject

@property (nonatomic, strong) NSString *key;                //键
@property (nonatomic, strong) NSData *value;                //值
@property (nullable, nonatomic, strong) NSString *filename; //文件名
@property (nonatomic) int size;                             //值的大小，单位是byte
@property (nonatomic) int modTime;                          //修改时间戳
@property (nonatomic) int accessTime;                       //最后访问的时间戳
@property (nullable, nonatomic, strong) NSData *extendedData; //extended data

@end
```

既然在这里将缓存封装成了YYKVStorageItem实例，**那么作为缓存的管理者，YYKVStorage就必然有操作YYKVStorageItem的接口**了：

```objc
//YYKVStorage.h

//写入某个item
- (BOOL)saveItem:(YYKVStorageItem *)item;

//写入某个键值对，值为NSData对象
- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value;

//写入某个键值对，包括文件名以及data信息
- (BOOL)saveItemWithKey:(NSString *)key
                  value:(NSData *)value
               filename:(nullable NSString *)filename
           extendedData:(nullable NSData *)extendedData;

#pragma mark - Remove Items

//移除某个键的item
- (BOOL)removeItemForKey:(NSString *)key;

//移除多个键的item
- (BOOL)removeItemForKeys:(NSArray<NSString *> *)keys;

//移除大于参数size的item
- (BOOL)removeItemsLargerThanSize:(int)size;

//移除时间早于参数时间的item
- (BOOL)removeItemsEarlierThanTime:(int)time;

//移除item，使得缓存总容量小于参数size
- (BOOL)removeItemsToFitSize:(int)maxSize;

//移除item，使得缓存数量小于参数size
- (BOOL)removeItemsToFitCount:(int)maxCount;

//移除所有的item
- (BOOL)removeAllItems;

//移除所有的item，附带进度与结束block
- (void)removeAllItemsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                               endBlock:(nullable void(^)(BOOL error))end;


#pragma mark - Get Items
//读取参数key对应的item
- (nullable YYKVStorageItem *)getItemForKey:(NSString *)key;

//读取参数key对应的data
- (nullable NSData *)getItemValueForKey:(NSString *)key;

//读取参数数组对应的item数组
- (nullable NSArray<YYKVStorageItem *> *)getItemForKeys:(NSArray<NSString *> *)keys;

//读取参数数组对应的item字典
- (nullable NSDictionary<NSString *, NSData *> *)getItemValueForKeys:(NSArray<NSString *> *)keys;
```

大家最关心的应该是写入缓存的接口是如何实现的，下面重点讲一下写入缓存的接口：


```objc
//写入某个item
- (BOOL)saveItem:(YYKVStorageItem *)item;

//写入某个键值对，值为NSData对象
- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value;

//写入某个键值对，包括文件名以及data信息
- (BOOL)saveItemWithKey:(NSString *)key
                  value:(NSData *)value
               filename:(nullable NSString *)filename
           extendedData:(nullable NSData *)extendedData;
```

这三个接口都比较类似，上面的两个方法都会调用最下面参数最多的方法。在详细讲解写入缓存的代码之前，我先讲一下写入缓存的大致逻辑，有助于让大家理解整个YYDiskCache写入缓存的流程：

1. 首先判断传入的key和value是否符合要求，如果不符合要求，则立即返回NO，缓存失败。
2. 再判断是否type==YYKVStorageTypeFile并且文件名为空字符串（或nil）：如果是，则立即返回NO，缓存失败。
3. 判断filename是否为空字符串：
  1. 如果不为空：写入文件，并将缓存的key，等信息写入数据库，但是不将key对应的data写入数据库。
  2. 如果为空：
        1. 如果缓存类型为YYKVStorageTypeSQLite：将缓存文件删除
            2. 如果缓存类型不为YYKVStorageTypeSQLite：则将缓存的key和对应的data等其他信息存入数据库。



```objc
- (BOOL)saveItem:(YYKVStorageItem *)item {
    return [self saveItemWithKey:item.key value:item.value filename:item.filename extendedData:item.extendedData];
}

- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value {
    return [self saveItemWithKey:key value:value filename:nil extendedData:nil];
}

- (BOOL)saveItemWithKey:(NSString *)key value:(NSData *)value filename:(NSString *)filename extendedData:(NSData *)extendedData {
    
    if (key.length == 0 || value.length == 0) return NO;
    
    if (_type == YYKVStorageTypeFile && filename.length == 0) {
        return NO;
    }
    
    if (filename.length) {
        
        //如果文件名不为空字符串，说明要进行文件缓存
        if (![self _fileWriteWithName:filename data:value]) {
            return NO;
        }
        
        //写入元数据
        if (![self _dbSaveWithKey:key value:value fileName:filename extendedData:extendedData]) {
            //如果缓存信息保存失败，则删除对应的文件
            [self _fileDeleteWithName:filename];
            return NO;
        }
        
        return YES;
        
    } else {
        
        //如果文件名为空字符串，说明不要进行文件缓存
        if (_type != YYKVStorageTypeSQLite) {
            
            //如果缓存类型不是数据库缓存，则查找出相应的文件名并删除
            NSString *filename = [self _dbGetFilenameWithKey:key];
            if (filename) {
                [self _fileDeleteWithName:filename];
            }
        }
        
        // 缓存类型是数据库缓存，把元数据和value写入数据库
        return [self _dbSaveWithKey:key value:value fileName:nil extendedData:extendedData];
    }
}
```

从上面的代码可以看出，在底层写入缓存的方法是``_dbSaveWithKey:value:fileName:extendedData:``，这个方法使用了两次:
- 在以文件（和数据库）存储缓存时
- 在以数据库存储缓存时

不过虽然调用了两次，我们可以从传入的参数是有差别的：第二次filename传了nil。那么我们来看一下``_dbSaveWithKey:value:fileName:extendedData:``内部是如何区分有无filename的情况的：

- 当filename为空时，说明在外部没有写入该缓存的文件：则把data写入数据库里
- 当filename不为空时，说明在外部有写入该缓存的文件：则不把data也写入了数据库里

下面结合代码看一下：


```objc
//数据库存储
- (BOOL)_dbSaveWithKey:(NSString *)key value:(NSData *)value fileName:(NSString *)fileName extendedData:(NSData *)extendedData {
    
    //sql语句
    NSString *sql = @"insert or replace into manifest (key, filename, size, inline_data, modification_time, last_access_time, extended_data) values (?1, ?2, ?3, ?4, ?5, ?6, ?7);";
    
    sqlite3_stmt *stmt = [self _dbPrepareStmt:sql];
    
    if (!stmt) return NO;
    
    int timestamp = (int)time(NULL);
    
    //key
    sqlite3_bind_text(stmt, 1, key.UTF8String, -1, NULL);
    
    //filename
    sqlite3_bind_text(stmt, 2, fileName.UTF8String, -1, NULL);
    
    //size
    sqlite3_bind_int(stmt, 3, (int)value.length);
    
    //inline_data
    if (fileName.length == 0) {
        
        //如果文件名长度==0，则将value存入数据库
        sqlite3_bind_blob(stmt, 4, value.bytes, (int)value.length, 0);
        
    } else {
        
        //如果文件名长度不为0，则不将value存入数据库
        sqlite3_bind_blob(stmt, 4, NULL, 0, 0);
    }
    
    //modification_time
    sqlite3_bind_int(stmt, 5, timestamp);
    
    //last_access_time
    sqlite3_bind_int(stmt, 6, timestamp);
    
    //extended_data
    sqlite3_bind_blob(stmt, 7, extendedData.bytes, (int)extendedData.length, 0);
    
    int result = sqlite3_step(stmt);
    
    if (result != SQLITE_DONE) {
        if (_errorLogsEnabled) NSLog(@"%s line:%d sqlite insert error (%d): %s", __FUNCTION__, __LINE__, result, sqlite3_errmsg(_db));
        return NO;
    }
    
    return YES;
}
```

框架作者用数据库的一条记录来保存关于某个缓存的所有信息。
而且数据库的第四个字段是保存缓存对应的data的，从上面的代码可以看出当filename为空和不为空的时候的处理的差别。

上面的``sqlite3_stmt``可以看作是一个已经把sql语句解析了的、用sqlite自己标记记录的内部数据结构。
而sqlite3_bind_text和sqlite3_bind_int是绑定函数，可以看作是将变量插入到字段的操作。

OK，现在看完了写入缓存，我们再来看一下获取缓存的操作：

```objc
//YYKVSorage.m
- (YYKVStorageItem *)getItemForKey:(NSString *)key {
    
    if (key.length == 0) return nil;
    
    YYKVStorageItem *item = [self _dbGetItemWithKey:key excludeInlineData:NO];
    
    if (item) {
        //更新内存访问的时间
        [self _dbUpdateAccessTimeWithKey:key];
        
        if (item.filename) {
            //如果有文件名，则尝试获取文件数据
            item.value = [self _fileReadWithName:item.filename];
            //如果此时获取文件数据失败，则删除对应的item
            if (!item.value) {
                [self _dbDeleteItemWithKey:key];
                item = nil;
            }
        }
    }
    return item;
}
```

从上面这段代码我们可以看到获取YYKVStorageItem的实例的方法是``_dbGetItemWithKey:excludeInlineData:``
我们来看一下它的实现：

1. 首先根据查找key的sql语句生成stmt
2. 然后将传入的key与该stmt进行绑定
3. 最后通过这个stmt来查找出与该key对应的有关该缓存的其他数据并生成item。

来看一下代码：

```objc
- (YYKVStorageItem *)_dbGetItemWithKey:(NSString *)key excludeInlineData:(BOOL)excludeInlineData {
    NSString *sql = excludeInlineData ? @"select key, filename, size, modification_time, last_access_time, extended_data from manifest where key = ?1;" : @"select key, filename, size, inline_data, modification_time, last_access_time, extended_data from manifest where key = ?1;";
    sqlite3_stmt *stmt = [self _dbPrepareStmt:sql];
    if (!stmt) return nil;
    sqlite3_bind_text(stmt, 1, key.UTF8String, -1, NULL);
    
    YYKVStorageItem *item = nil;
    int result = sqlite3_step(stmt);
    if (result == SQLITE_ROW) {
        //传入stmt来生成YYKVStorageItem实例
        item = [self _dbGetItemFromStmt:stmt excludeInlineData:excludeInlineData];
    } else {
        if (result != SQLITE_DONE) {
            if (_errorLogsEnabled) NSLog(@"%s line:%d sqlite query error (%d): %s", __FUNCTION__, __LINE__, result, sqlite3_errmsg(_db));
        }
    }
    return item;
}
```

我们可以看到最终生成YYKVStorageItem实例的是通过``_dbGetItemFromStmt:excludeInlineData:``来实现的：


```objc
- (YYKVStorageItem *)_dbGetItemFromStmt:(sqlite3_stmt *)stmt excludeInlineData:(BOOL)excludeInlineData {
    
    //提取数据
    int i = 0;
    char *key = (char *)sqlite3_column_text(stmt, i++);
    char *filename = (char *)sqlite3_column_text(stmt, i++);
    int size = sqlite3_column_int(stmt, i++);
    
    //判断excludeInlineData
    const void *inline_data = excludeInlineData ? NULL : sqlite3_column_blob(stmt, i);
    int inline_data_bytes = excludeInlineData ? 0 : sqlite3_column_bytes(stmt, i++);
    
    
    int modification_time = sqlite3_column_int(stmt, i++);
    int last_access_time = sqlite3_column_int(stmt, i++);
    const void *extended_data = sqlite3_column_blob(stmt, i);
    int extended_data_bytes = sqlite3_column_bytes(stmt, i++);
    
    //将数据赋给item的属性
    YYKVStorageItem *item = [YYKVStorageItem new];
    if (key) item.key = [NSString stringWithUTF8String:key];
    if (filename && *filename != 0) item.filename = [NSString stringWithUTF8String:filename];
    item.size = size;
    if (inline_data_bytes > 0 && inline_data) item.value = [NSData dataWithBytes:inline_data length:inline_data_bytes];
    item.modTime = modification_time;
    item.accessTime = last_access_time;
    if (extended_data_bytes > 0 && extended_data) item.extendedData = [NSData dataWithBytes:extended_data length:extended_data_bytes];
    return item;
}
```

上面这段代码分为两个部分：

- 获取数据库里每一个字段对应的数据
- 将数据赋给YYKVStorageItem的实例

需要注意的是：

1. 字符串类型需要使用``stringWithUTF8String:``来转成NSString类型。
2. 这里面会判断``excludeInlineData``：
  - 如果为TRUE，就提取存入的data数据
  - 如果为FALSE，就不提取




## 保证线程安全的方案

我相信对于某个设计来说，它的产生一定是基于某种个特定问题下的某个场景的

由上文可以看出：

- YYMemoryCache 使用了 pthread_mutex 线程锁（互斥锁）来确保线程安全
- YYDiskCache 则选择了更适合它的 dispatch_semaphore。

### 内存缓存操作的互斥锁

在YYMemoryCache中，是使用互斥锁来保证线程安全的。
首先在YYMemoryCache的初始化方法中得到了互斥锁，并在它的所有接口里都加入了互斥锁来保证线程安全，包括setter，getter方法和缓存操作的实现。举几个例子：

```objc
- (NSUInteger)totalCost {
    pthread_mutex_lock(&_lock);
    NSUInteger totalCost = _lru->_totalCost;
    pthread_mutex_unlock(&_lock);
    return totalCost;
}

- (void)setReleaseOnMainThread:(BOOL)releaseOnMainThread {
    pthread_mutex_lock(&_lock);
    _lru->_releaseOnMainThread = releaseOnMainThread;
    pthread_mutex_unlock(&_lock);
}

- (BOOL)containsObjectForKey:(id)key {
    
    if (!key) return NO;
    pthread_mutex_lock(&_lock);
    BOOL contains = CFDictionaryContainsKey(_lru->_dic, (__bridge const void *)(key));
    pthread_mutex_unlock(&_lock);
    return contains;
}

- (id)objectForKey:(id)key {
    
    if (!key) return nil;
    
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
        //如果节点存在，则更新它的时间信息（最后一次访问的时间）
        node->_time = CACurrentMediaTime();
        [_lru bringNodeToHead:node];
    }
    pthread_mutex_unlock(&_lock);
    
    return node ? node->_value : nil;
}

```

而且需要在dealloc方法中销毁这个锁头：

```objc
- (void)dealloc {
    
    ...
    
    //销毁互斥锁
    pthread_mutex_destroy(&_lock);
}
```


### 磁盘缓存使用信号量来代替锁


框架作者采用了信号量的方式来给
首先在初始化的时候实例化了一个信号量：

```objc
- (instancetype)initWithPath:(NSString *)path
             inlineThreshold:(NSUInteger)threshold {
    ...
    _lock = dispatch_semaphore_create(1);
    _queue = dispatch_queue_create("com.ibireme.cache.disk", DISPATCH_QUEUE_CONCURRENT);
    ...
```

然后使用了宏来代替加锁解锁的代码：

```objc
#define Lock() dispatch_semaphore_wait(self->_lock, DISPATCH_TIME_FOREVER)
#define Unlock() dispatch_semaphore_signal(self->_lock)
```

简单说一下信号量：

dispatch_semaphore是GCD用来同步的一种方式，与他相关的共有三个函数，分别是

- dispatch_semaphore_create：定义信号量
- dispatch_semaphore_signal：使信号量+1
- dispatch_semaphore_wait：使信号量-1

当信号量为0时，就会做等待处理，这是其他线程如果访问的话就会让其等待。所以如果信号量在最开始的的时候被设置为1，那么就可以实现“锁”的功能：
- 执行某段代码之前，执行dispatch_semaphore_wait函数，让信号量-1变为0，执行这段代码。
- 此时如果其他线程过来访问这段代码，就要让其等待。
- 当这段代码在当前线程结束以后，执行dispatch_semaphore_signal函数，令信号量再次+1，那么如果有正在等待的线程就可以访问了。

需要注意的是：如果有多个线程等待，那么后来信号量恢复以后访问的顺序就是线程遇到dispatch_semaphore_wait的顺序。

这也就是信号量和互斥锁的一个区别：互斥量用于线程的互斥，信号线用于线程的同步。

- 互斥：是指某一资源同时只允许一个访问者对其进行访问，具有唯一性和排它性。但**互斥无法限制访问者对资源的访问顺序，即访问是无序的**。

- 同步：是指在互斥的基础上（大多数情况），通过其它机制实现访问者对资源的有序访问。在大多数情况下，同步已经实现了互斥，特别是所有写入资源的情况必定是互斥的。也就是说使用信号量可以使多个线程有序访问某个资源。


那么问题来了：为什么内存缓存使用的是互斥锁（pthread_mutex），而磁盘缓存使用的就是信号量（dispatch_semaphore）呢？

答案在框架作者的文章[YYCache 设计思路](https://blog.ibireme.com/2015/10/26/yycache/)里可以找到:


为什么内存缓存使用互斥锁（pthread_mutex）？

框架作者在最初使用的是自旋锁(OSSpinLock)作为内存缓存的线程锁，但是后来得知其不够安全，所以退而求其次，使用了pthread_mutex。


为什么磁盘缓存使用的是信号量（dispatch_semaphore）？


>dispatch_semaphore 是信号量，但当信号总量设为 1 时也可以当作锁来。在没有等待情况出现时，它的性能比 pthread_mutex 还要高，但一旦有等待情况出现时，性能就会下降许多。相对于 OSSpinLock 来说，它的优势在于等待时不会消耗 CPU 资源。对磁盘缓存来说，它比较合适。

因为YYDiskCache在写入比较大的缓存时，可能会有比较长的等待时间，而dispatch_semaphore在这个时候是不消耗CPU资源的，所以比较适合。


## 提高缓存性能的几个尝试


### 选择合适的线程锁

可以参考上一部分YYMemoryCache 和YYDiskCache使用的不同的锁以及原因。

### 选择合适的数据结构

在YYMemoryCache中，作者选择了双向链表来保存这些缓存节点。那么可以思考一下，为什么要用双向链表而不是单向链表或是数组呢？

- 为什么不选择单向链表：单链表的节点只知道它后面的节点（只有指向后一节点的指针），而不知道前面的。所以如果想移动其中一个节点的话，其前后的节点不好做衔接。

- 为什么不选择数组：数组中元素在内存的排列是连续的，对于寻址操作非常便利；但是对于插入，删除操作很不方便，需要整体移动，移动的元素个数越多，代价越大。而链表恰恰相反，因为其节点的关联仅仅是靠指针，所以对于插入和删除操作会很便利，而寻址操作缺比较费时。由于在LRU策略中会有非常多的移动，插入和删除节点的操作，所以使用双向链表是比较有优势的。


### 选择合适的线程来操作不同的任务

无论缓存的自动清理和释放，作者默认把这些任务放到子线程去做：

看一下释放所有内存缓存的操作：

```objc
- (void)removeAll {
    
    //将开销，缓存数量置为0
    _totalCost = 0;
    _totalCount = 0;
    
    //将链表的头尾节点置空
    _head = nil;
    _tail = nil;
    
    if (CFDictionaryGetCount(_dic) > 0) {
        
        CFMutableDictionaryRef holder = _dic;
        _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
        
        //是否在子线程操作
        if (_releaseAsynchronously) {
            dispatch_queue_t queue = _releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else if (_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else {
            CFRelease(holder);
        }
    }
}
```

这里的``YYMemoryCacheGetReleaseQueue()``使用了内联函数，返回了低优先级的并发队列。

```objc
//内联函数，返回优先级最低的全局并发队列
static inline dispatch_queue_t YYMemoryCacheGetReleaseQueue() {
    return dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0);
}
```


### 选择底层的类

同样是字典实现，但是作者使用了更底层且快速的CFDictionary而没有用NSDictionary来实现。


## 其他知识点


### 禁用原生初始化方法并标明新定义的指定初始化方法

YYCache有4个供外部调用的初始化接口，无论是对象方法还是类方法都需要传入一个字符串（名称或路径）。

而两个原生的初始化方法被框架作者禁掉了：

```objc
- (instancetype)init UNAVAILABLE_ATTRIBUTE;
+ (instancetype)new UNAVAILABLE_ATTRIBUTE;
```

如果用户使用了上面两个初始化方法就会在编译期报错。

而剩下的四个可以使用的初始化方法中，有一个是指定初始化方法，被作者用``NS_DESIGNATED_INITIALIZER``标记了。

```objc
- (nullable instancetype)initWithName:(NSString *)name;
- (nullable instancetype)initWithPath:(NSString *)path NS_DESIGNATED_INITIALIZER;

+ (nullable instancetype)cacheWithName:(NSString *)name;
+ (nullable instancetype)cacheWithPath:(NSString *)path;
```

指定初始化方法就是所有可使用的初始化方法都必须调用的方法。更详细的介绍可以参考我的下面两篇文章：

- [iOS 代码规范](https://juejin.im/post/5940c8befe88c2006a468ea6)中讲解“类”的这一部分。
- [《Effective objc》干货三部曲（三）：技巧篇](https://juejin.im/post/5a4f3710f265da3e4d728239)中的第16条。


### 异步释放对象的技巧

为了异步将某个对象释放掉，可以通过在GCD的block里面给它发个消息来实现。这个技巧在该框架中很常见，举一个删除一个内存缓存的例子：

首先将这个缓存的node类取出，然后异步将其释放掉。

```objc
- (void)removeObjectForKey:(id)key {
    
    if (!key) return;
    
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
        [_lru removeNode:node];
        if (_lru->_releaseAsynchronously) {
            dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                [node class]; //hold and release in queue
            });
        } else if (_lru->_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [node class]; //hold and release in queue
            });
        }
    }
    pthread_mutex_unlock(&_lock);
}
```

为了释放掉这个node对象，在一个异步执行的（主队列或自定义队列里）block里给其发送了``class``这个消息。不需要纠结这个消息具体是什么，他的目的是为了避免编译错误，因为我们无法在block里面硬生生地将某个对象写进去。

其实关于上面这一点我自己也有点拿不准，希望理解得比较透彻的同学能在下面留个言~ ^^





### 内存警告和进入后台的监听



YYCache默认在收到内存警告和进入后台时，自动清除所有内存缓存。所以在YYMemoryCache的初始化方法里，我们可以看到这两个监听的动作：

```objc
//YYMemoryCache.m

- (instancetype)init{
    
    ...
      
    //监听app生命周期
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidReceiveMemoryWarningNotification) name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidEnterBackgroundNotification) name:UIApplicationDidEnterBackgroundNotification object:nil];
   
    ...
}
```



然后实现监听到消息后的处理方法：

```objc
//内存警告时，删除所有内存缓存
- (void)_appDidReceiveMemoryWarningNotification {
    if (self.didReceiveMemoryWarningBlock) {
        self.didReceiveMemoryWarningBlock(self);
    }
    if (self.shouldRemoveAllObjectsOnMemoryWarning) {
        [self removeAllObjects];
    }
}


//进入后台时，删除所有内存缓存
- (void)_appDidEnterBackgroundNotification {
    if (self.didEnterBackgroundBlock) {
        self.didEnterBackgroundBlock(self);
    }
    if (self.shouldRemoveAllObjectsWhenEnteringBackground) {
        [self removeAllObjects];
    }
}

```






### 判断头文件的导入

```objc
#if __has_include(<YYCache/YYCache.h>)
#import <YYCache/YYMemoryCache.h>
#import <YYCache/YYDiskCache.h>
#import <YYCache/YYKVStorage.h>
#elif __has_include(<YYWebImage/YYCache.h>)
#import <YYWebImage/YYMemoryCache.h>
#import <YYWebImage/YYDiskCache.h>
#import <YYWebImage/YYKVStorage.h>
#else
#import "YYMemoryCache.h"
#import "YYDiskCache.h"
#import "YYKVStorage.h"
#endif
```

在这里作者使用__has_include来检查Frameworks是否引入某个类。
因为YYWebImage已经集成YYCache,所以如果导入过YYWebImage的话就无需重再导入YYCache了。


# 最后的话


通过看该组件的源码，我收获的不仅有缓存设计的思路，还有：

- 双向链表的概念以及相关操作
- 数据库的使用
- 互斥锁，信号量的使用
- 实现线程安全的方案
- 变量，方法的命名以及接口的设计

相信读过这篇文章的你也会有一些收获~ 
如果能趁热打铁，下载一个[YYCache](https://github.com/ibireme/YYCache)源码看就更好啦~









