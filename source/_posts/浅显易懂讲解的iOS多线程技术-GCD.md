---
title: 浅显易懂讲解iOS多线程技术-GCD
tags: [iOS,Objective-C]
categories: iOS
---


励志打造一篇浅显易懂地介绍iOS中GCD的文章！
笔者见过很多其他讲解GCD的博客，有些写得非常详细非常专业，几乎涵盖了GCD大大小小的全部知识，细致庞杂的内容容易让人摸不清主次，笔者觉得这类文章**并不适合初学者学习**，于是决定写一篇针对一些只是听过，但是对GCD还不了解的童鞋们。

本文排除了一些细枝末节，扰乱人头绪的东西，着重讲解了GCD中重要的知识点，并在最后展示了GCD中**经常使用的函数**并附上结果图和讲解，简单明了。

<!-- more -->


# 进程与线程

-----

在了解多线程之前，需要弄清进程和线程的概念和他们之间的区别。



### 进程：

系统中正在运行的一个程序，进程之间是相互独立的，每个进程都有属于自己的内存空间。比如手机中的**微信**应用和**印象笔记**应用，他们都是iOS系统中独立的进程，有着自己的内存空间。



### 线程：

进程内部执行任务所需要的执行路径。进程若想执行任务，则必须得在线程下执行。也就是说进程至少有一个线程才能执行任务。但是，我们使用软件的时候，很少有只让它做一件事的时候：

举个**印象笔记**的🌰 ： 当你正在编辑一则笔记的时候点击了同步按钮，那么编辑任务（线程）和同步任务（线程）一定是不能按照顺序执行的。因为同步任务的完成时间是不可控的，如果在同步的过程中无法进行别的任务（线程）那就太糟糕了！

因此，我们需要让一些任务可以同时进行。既然任务是在线程上执行的，那么多任务的执行就意味着需要多线程的开启和使用。

来一张图直观地展示一下内存，进程和线程的关系：

![内存，进程和线程](http://upload-images.jianshu.io/upload_images/859001-a1e6c65eda0d3aaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





# 多线程概述
-----

多线程的实现原理：虽然在同一时刻，CPU只能处理1条线程，但是CPU可以快速地在多条线程之间调度（切换），造成了多线程并发执行的假象。



## 1. 多线程的优点

- 能适当提高程序的执行效率。
- 能适当提高资源利用率（CPU、内存利用率）。



## 2. 多线程的缺点

- 创建线程是需要成本的：iOS下主要成本包括：在栈空间的子线程512KB、主线程1MB，创建线程大约需要90毫秒的创建时间。
- 线程越多，CPU在调度线程上的开销就越大。
- 线程越多，程序设计就越复杂：因为要考虑到线程之间的通信，多线程的数据共享。


# 多线程在iOS开发中的应
------





## 1. iOS的主线程

一个iOS程序运行后，默认会开启1条线程，称为“主线程”或“UI线程”



#### 主线程的作用:

- 显示\刷新UI界面
- 处理UI事件（比如点击事件、滚动事件、拖拽事件等）

>主线程的使用**注意事项**:
>不能把比较耗时的操作放到主线程中，，严重影响UI的流畅度，给用户一种程序“卡顿”的体验。
>因此，要将耗时的操作放在子线程中异步执行。这样一来，及时开始执行了耗时的操作，也不会影响主线程中UI交互的体验。



## 2. iOS的子线程
子线程是异步执行的，不影响主线程。在iOS开发中，我们需要将耗时的任务（网络请求，复杂的运算）放在子线程进行，不让其影响UI的交互体验。

## 3. 多线程安全 

当多个线程访问同一块资源时，很容易引发数据错乱和数据安全问题。就好比好几个人在同时修改同一个表格，造成数据的错乱。

#### 3.1 资源抢夺的解决方案

我们需要给数据添加**互斥锁**。也就是说，当某线程访问一个数据之前就要给数据加锁，让其不被其他的线程所修改。就好比一个人修改表格的时候给表格设置了密码，那么其他人就无法访问文件了。当他修改文件之后，再讲密码撤销，第二个人就可以访问该文件了。


>**注意**：
>这里的线程都为子线程，如果给数据加了锁，就等于将这些异步的子线程变成同步的了，这也叫做线程同步技术。


#### 3.2 互斥锁使用：

```objc
@synchronized(锁对象) { // 需要锁定的代码  };
```



#### 3.3 互斥锁的优缺点

优点：能有效防止因多线程抢夺资源造成的数据安全问题
缺点：需要消耗大量的CPU资源

互斥锁的使用前提：多条线程抢夺同一块资源的时候使用。


#### 3.4互斥锁在iOS开发中的使用

OC在定义属性时有``nonatomic``和``atomic``两种选择

- atomic：原子属性，为setter方法加锁（默认就是atomic）
- nonatomic：非原子属性，不会为setter方法加锁


#### 3.5 nonatomic和atomic对比

atomic：线程安全，需要消耗大量的资源
nonatomic：非线程安全，适合内存小的移动设备


>**建议：**
>所有属性都声明为nonatomic，尽量避免多线程抢夺同一块资源，将加锁、资源抢夺的业务逻辑交给服务器端处理，减小移动客户端的压力。



# 多线程在iOS中的应用：GCD 
----


GCD，全称为 Grand Central Dispatch ，是iOS用来管理线程的技术。 纯C语言，提供了非常多强大的函数。


## 1. GCD的优势

GCD会自动利用更多的CPU内核（比如双核、四核）。
GCD会自动管理线程的生命周期（创建线程、调度任务、销毁线程）。
程序员只需要告诉GCD想要执行什么任务，不需要编写任何线程管理代码。



## 2. 为什么要用GCD？

为了要提高软件性能，应该异步执行耗时任务(加载图片)，以防止影响主线程任务的执行(UI相应)。


>举个🌰 ：
>从网络加载一张图片，如果将此任务放到主线程，那么在下载完成的时间里，软件是无法相应用户的任何操作的。特别地，如果当前是在可以滚动的页面，就会造成无法滚动这种体验非常糟的情况。

所以：应该将网络加载放在异步执行，执行成功后，再回到主线程显示加载后的图片(详细做法马上就会讲到)。



## 3. GCD的使用步骤

1. 由开发者定制将要执行的任务。
- 将任务添加到队列中，GCD会自动将队列中的任务取出，放到对应的线程中执行。

>**注意：**
>任务的取出遵循队列的FIFO原则：先进先出，后进后出。



## 4. 什么是队列？

队列是用来存放任务的，由GCD将这些任务从队列中取出并放到相应的线程中执行。

### GCD的队列可以分为2大类型：

#### 1. 并发队列（Concurrent Dispatch Queue）

可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务），并发功能只有在异步（dispatch_async）函数下才有效



#### 2. 串行队列（Serial Dispatch Queue）

让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）。



那么队列和线程又有什么区别？

简单来说，队列就是用来存放任务的“暂存区”，而线程是执行任务的路径，GCD将这些存在于队列的任务取出来放到相应的线程上去执行，而队列的性质决定了在其中的任务在哪种线程上执行。

下面由一张图来直观地展示任务，队列和线程的关系：


![任务，队列和线程](http://upload-images.jianshu.io/upload_images/859001-6da601dd550b8390.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







>在这里，我们可以看到，放入串行队列的任务会一个一个地执行。而放入并行队列的任务，会在多个线程并发地执行。



## 5. 队列的创建


### 5.1 串行队列的创建：


GCD中获得串行有2种途径：

1.使用``dispatch_queue_create``函数创建串行队列

```objc
// 创建串行队列（队列类型传递NULL或者DISPATCH_QUEUE_SERIAL）
dispatch_queue_t queue = dispatch_queue_create("serial_queue", NULL); 
```



2.使用主队列（跟主线程相关联的队列）

主队列是GCD自带的一种特殊的串行队列：放在主队列中的任务，都会放到主线程中执行。
可以使用dispatch_get_main_queue()获得系统提供的主队列：

```objc
dispatch_queue_t queue = dispatch_get_main_queue();
```

### 5.2 并发队列的创建：

1.使用``dispatch_queue_create``函数创建并发队列。

```objc
dispatch_queue_t queue = dispatch_queue_create("concurrent.queue", DISPATCH_QUEUE_CONCURRENT);
```

2.使用``dispatch_get_global_queue``获得全局并发队列。

GCD默认已经提供了全局的并发队列，供整个应用使用，可以无需手动创建。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 
```



## 6. GCD的几种重要的应用

### 6.1 子线程与主线程的通信

需求点:我们有时需要在子线程处理一个耗时比较长的任务，而且此任务完成后，要在主线程执行另一个任务。
例子：从网络加载图片（在子线程），加载完成就更新UIView（在主线程）。

为了实现这个需求，我们需要首先拿到全局并发队列（或自己开启一个子线程）来执行耗时的操作，然后在其完成block中拿到全局串行队列来执行UI刷新的任务。

```objc
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

              //加载图片
              NSData *dataFromURL = [NSData dataWithContentsOfURL:imageURL];
              UIImage *imageFromData = [UIImage imageWithData:dataFromURL];

      dispatch_async(dispatch_get_main_queue(), ^{
          
              //加载完成更新view
              UIImageView *imageView = [[UIImageView alloc] initWithImage:imageFromData];          
      });      
  });
```

>以笔者的拙见，除了复杂的算法，网络请求以外，大多数``dataWithContentsOf。。。``函数可能也会比较耗时，所以以后遇到与NSData交互的操作时，尽量将其放在子线程执行。



#### 6.2 dispatch_once

需求点：用于在程序启动到终止，只执行一次的代码。此代码被执行后，相当于自身全部被加上了注释，不会再执行了。
为了实现这个需求，我们需要使用``dispatch_once``让代码在运行一次后即刻被“雪藏”。

```objc
//使用dispatch_once函数能保证某段代码在程序运行过程中只被执行1次
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    // 只执行1次的代码，这里默认是线程安全的：不会有其他线程可以访问到这里
});
```



#### 6.3 dispatch_group


需求点：执行多个耗时的异步任务，但是只能等到这些任务都执行完毕后，才能在主线程执行某个任务。
为了实现这个需求，我们需要让将这些异步执行的操作放在``dispatch_group_async``函数中执行，最后再调用``dispatch_group_notify``来执行最后执行的任务。

```objc
  dispatch_group_t group =  dispatch_group_create();
  dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      // 执行1个耗时的异步操作
  });
  dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      // 执行1个耗时的异步操作
  });
  dispatch_group_notify(group, dispatch_get_main_queue(), ^{
      // 等前面的异步操作都执行完毕后，回到主线程...
  });
```



让我们看一下示例代码和运行结果：


示例代码：

为了使对比明显，笔者多开了几条线程，这样更容易看清问题。
```objc
    dispatch_group_t group =  dispatch_group_create();    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        // 执行1个耗时的异步操作
        for (NSInteger index = 0; index < 10000; index ++) {
        }
        NSLog(@"完成了任务1");                
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        // 执行1个耗时的异步操作
        for (NSInteger index = 0; index < 20000; index ++) {
        }
        NSLog(@"完成了任务2");        
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        // 执行1个耗时的异步操作
        for (NSInteger index = 0; index < 200000; index ++) {
        }
        NSLog(@"完成了任务3");
        
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        // 执行1个耗时的异步操作
        for (NSInteger index = 0; index < 400000; index ++) {
        }
        NSLog(@"完成了任务4");
        
    });
    
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        
        // 执行1个耗时的异步操作
        for (NSInteger index = 0; index < 1000000; index ++) {
        }
        NSLog(@"完成了任务5");
        
    });
            
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        
        // 等前面的异步操作都执行完毕后，回到主线程...
        NSLog(@"都完成了");        
    });
```



运行结果:


![  dispatch_group 的使用运行结果](http://upload-images.jianshu.io/upload_images/859001-3c8d5886442ce7b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)









>从三次运行的结果来看:
1. 异步执行的任务1-5的最终完成时间是与其自身完成任务所需要的时间并无绝对关联。因为任务5是最耗时的，它在第一次运行结果里并不是最后才完成的。任务1是最不耗时的，但是它在第二次运行结果里也不是最先完成的。

>2. 异步执行的任务1-5无论完成顺序如何，只有当他们都完成后才会调用主线程的打印“都完成了”。



#### 6.4 dispatch_barrier

需求点：虽然我们有时要执行几个不同的异步任务，但是我们还是要将其分成两组：当第一组异步任务都执行完成后才执行第二组的异步任务。这里的组可以包含一个任务，也可以包含多个任务。

为了实现这个需求，我们需要使用``dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);``在两组任务之间形成“栅栏”，使其“下方”的异步任务在其“上方”的异步任务都完成之前是无法执行的。


```objc
    dispatch_queue_t queue = dispatch_queue_create("12312312", DISPATCH_QUEUE_CONCURRENT);

    dispatch_async(queue, ^{
        NSLog(@"----任务 1-----");
    });

    dispatch_async(queue, ^{
        NSLog(@"----任务 2-----");
    });    

    dispatch_barrier_async(queue, ^{
        NSLog(@"----barrier-----");
    });
   
    dispatch_async(queue, ^{
        NSLog(@"----任务 3-----");
    });

    dispatch_async(queue, ^{
        NSLog(@"----任务 4-----");
    });
```





示例代码：

```objc
    dispatch_queue_t queue = dispatch_queue_create("12312312", DISPATCH_QUEUE_CONCURRENT);
   
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 10000; index ++) {
        }
        NSLog(@"完成了任务1");       
    });
    
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 20000; index ++) {
        }
        NSLog(@"完成了任务2");        
    });
    
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 200000; index ++) {
        }
        NSLog(@"完成了任务3");        
    });
         
    
    dispatch_barrier_async(queue, ^{        
        NSLog(@"--------我是分割线--------");        
    });
    
        
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 400000; index ++) {
        }
        NSLog(@"完成了任务4");       
    });
    
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 1000000; index ++) {
        }
        NSLog(@"完成了任务5");        
    });
    
    dispatch_async(queue, ^{
        
        for (NSInteger index = 0; index < 1000; index ++) {
        }
        NSLog(@"完成了任务6");       
    });
```



运行结果：


![dispatch_barrier 的使用运行结果](http://upload-images.jianshu.io/upload_images/859001-ca923e54d3839b2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)






>从这三次运行结果来看:
1. 无论任务1-3内部的执行顺序如何，只有当三者都完成了才会执行任务4-6。
2. 1-3内部的执行顺序和4-6内部的完成顺序都是不可控的，同上一个知识点类似。

本文介绍了需要了解GCD所需的最重要的知识，因为怕打断读者思路，并没有涵盖所有细节。以后有机会会再写一篇深入介绍GCD的文章，查缺补漏。


