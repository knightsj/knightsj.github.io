---
title: 斯坦福大学iOS开发公开课总结（十）：多线程，UIScrollView，图片浏览器Demo
tags: [iOS,Objective-C]
categories: iOS
---

本节课讲授了多线程编程和UIScrollview控件，最后通过一个显示图片的Demo综合了本节课讲解的知识。通过本节课的学习，我们可以初步了解该如何处理耗时的任务来提高系统性能的方法以及通过UIScrollview控件来显示超出屏幕大小的图片并实现滚动和缩放的效果。



# 多线程
-----
实现多线程编程（将不同的任务放在主线程和子线程工作），可以有效利用系统硬件优势提高系统性能。
首先，先介绍几个概念：

## 队列
队列：在队列中放入用来执行任务的block。这些block按照队列的性质被取出到应该工作的线程(主线程，子线程)。

队列分为串行队列和并行队列。
- 放入串行队列的任务将会在主线程执行，执行顺序是按照顺序执行。
- 放入并行队列的任务会在子线程执行，执行顺序是并行执行。

那么什么样的任务会放在主线程或子线程执行呢？

## 主线程&子线程
主线程：负责执行UI活动，绝大部分的UI活动都要在这里调用，不能让其阻塞，要将耗时的任务放到子线程来做。
子线程：负责执行耗时的运算，网络请求等不能放在主线程的任务。

系统为我们提供了共用的主队列(Main Dispatch Queue)和全局并行队列(Global Dispatch Queue)。我们只需将需要执行的任务放入到这两类队列里就可以实现多线程编程。

<!-- more -->

## 得到主队列
```
dispatch_queue_t mainQ = dispatch_get_main_queue();
NSOperationQueue *mainQ = [NSOperationQueue mainQueue];
```

## 得到主队列并布置任务
```
//NSThread
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;

//GCD
 dispatch_async(dispatch_get_main_queue(), ^{                   

                        [doSomething];

                    });
```

## 得到全局并行队列
```
dispatch queue_t globalDispatchQueueDefault = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
```

## 得到全局并行队列并布置任务

```
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
 
           [doSomething];
  });

```

## 使用多线程的例子：线程之间通信
很多情况下，我们需要在子线程进行下载任务，下载完成后在主线程更新UI，这时候就需要线程之间的通信：
```
  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      
              //下载图片
              NSData *dataFromURL = [NSData dataWithContentsOfURL:imageURL];
              UIImage *imageFromData = [UIImage imageWithData:dataFromURL];

      dispatch_async(dispatch_get_main_queue(), ^{
          
              //加载完成更新view
              UIImageView *imageView = [[UIImageView alloc] initWithImage:imageFromData];
          
      });
      
  });

```
>在这里，我们在全局并行队列的回调block里调用了主线程，并在主线程里执行了UI操作。

## 使用多线程的例子：通过NSURLSession下载
```
- (void)mainQueueCallBack
{
    NSURLRequest *request = [NSURLRequest requestWithURL:self.imageURL];
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:nil delegateQueue:[NSOperationQueue mainQueue]];
    NSURLSessionDownloadTask *task = [session downloadTaskWithRequest:request completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {
         //这里是主队列，可以更新UI        
    }];

    [task resume];
}
```
>在这里，``delegateQueue``的参数是主线程，所以``downloadTaskWithRequest::``方法的回调函数是在主线程，我们就可以在那里作更新UI的操作。

如果没有``delegateQueue``呢？我们需要自己获取主线程
```
- (void)noDelegateQueueRequest
{
    NSURLRequest *request = [NSURLRequest requestWithURL:self.imageURL];
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
    NSURLSessionDownloadTask *task = [session downloadTaskWithRequest:request completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {

     //获取主线程-通过NSThread
     [self performSelectorOnMainThread:(doUIThings) withObject:nil waitUntilDone:NO];
     //获取主线程-通过GCD
    dispatch_async(dispatch_get_main_queue(), ^{                   
                        [doUIThings]
                    });         
    }];    
    [task resume];
}
```

# UIScrollView
-----
UIScrollView是滚动视图，可以实现滚动和缩放的功能。

## 几个比较重要的属性：
视图要滚动的区域：``contentSize``
目前滚动的位置：``contentOffset``
滚动窗口的大小：``scrollView.bounds``

## 几个比较重要的方法：

获取当前显示的部分：
```
CGRect visibleRect = [scrollView convertRect：scrollView.bounds toView:subview];
```

用代码滚动视图：
```
- (void)scrollRectToVisible :(CGRect)aRect animated:(BOOL)animated;
```
代码实现缩放：
```
@property(nonatomic) CGFloat zoomScale;  
- (void)setZoomScale:(CGFloat)scale animated:(BOOL)animated);
- (void)zoomToRect:(CGRect)rect animated:(BOOL)animated;
```
告诉要缩放哪个```UIView```:
```
- (nullable UIView *)viewForZoomingInScrollView:(UIScrollView *)
```

## 缩放
#### 设置缩放极限
```
scrollView.minimumZoomSize = 0.5;
scrollView.maximumZoomSize = 2.0;
```

# Demo
------
## Demo需求
- 第一个页面显示三个按钮，在跳转后分别下载并显示不同图片。
- 在图片的下载过程中给予提示。
- 图片显示出来后可以移动，缩放。

## 效果图


![左：第一页 | 右：第二页](http://upload-images.jianshu.io/upload_images/859001-30b1dbbc035c8e1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




## 重要代码段

#### 1. 在跳转之前将图片下载的URL传给第二个页面
```
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{
     //通过内省判断跳转的页面类
     if ([segue.destinationViewController isKindOfClass:[ImageViewController class]]) {       

        //告诉编译期，即将跳转的页面类
        ImageViewController *imageVC = (ImageViewController *)segue.destinationViewController;        
        //初始化指针，将其设为nil 
        NSString *string = nil;
        //通过identifier判断跳转界面
        if ([segue.identifier isEqualToString:@"paint"]) {            
            //这张图貌似得翻墙，而且图片很大，建议换一张
            string = @"https://lh6.ggpht.com/ZoD88QrTxZbZnhpJgQbo9SPuosryX9ujjdRaHvjjvbUGeZcI-9C4AFQsWQm7-pVDv1E=h900";       

        }else if ([segue.identifier isEqualToString:@"earth"]) {
            //这张图不是很大，可以不用花很久就能显示
            string = @"http://news.nationalgeographic.com/content/dam/news/2016/02/12/01asteroidearth.jpg";           

        }else if ([segue.identifier isEqualToString:@"night"])  {
            //这张图貌似得翻墙，而且图片很大，建议换一张
            string = @"https://lh5.ggpht.com/j4C_pXnbRc5FnxNO90wIqodn4QA3f_6rB0cyu2sVnCeSwLDmyZf-xSrC9L8c3oxr6NE=h900";

        }        

        imageVC.imageURL = [NSURL URLWithString:string];

         //设置导航栏的标题
        imageVC.title = segue.identifier;

    }
}
```

#### 2. 使UIScrollView控件能够拖动
```
/**
 *  设置图片后，重新imageView的图片和自己的大小，并设置contentSize
 *
 *  @param image <#image description#>
 */
- (void)setImage:(UIImage *)image
{
    self.imageView.image = image;
    //根据图片大小设置imageview的大小

    [self.imageView sizeToFit];
    //保护机制：有图片设置size，否则size=0

    self.scrollView.contentSize = self.image? self.image.size : CGSizeZero;

}
```
>为了使UIScrollView控件能够拖动，**必须**要设置它的contentSize大小，否则无法滚动！

#### 3. 设置UIScrollView伸缩
```

- (void)setScrollView:(UIScrollView *)scrollView
{
    _scrollView = scrollView;
    _scrollView.minimumZoomScale = 0.2;
    _scrollView.maximumZoomScale = 2.0;
    _scrollView.delegate = self;
    //设置两次contSize的原因是我们不确保这两个方法哪个是先被调用的

    self.scrollView.contentSize = self.image? self.image.size : CGSizeZero;

}
```

效果图：


![实现伸缩效果](http://upload-images.jianshu.io/upload_images/859001-2d0da3c643a5b7d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 4. 下载图片
错误做法：阻塞主线程
```
- (void)setImageURL:(NSURL *)imageURL
{
    _imageURL = imageURL;   
    self.image = [UIImage imageWithData:[NSData dataWithContentsOfURL:self.imageURL]];
}
```
>永远不要在主线程调用下载的方法！主线程负责UI相应，如果调用耗时的方法会使得其下一项任务在下载完成之前无法执行（主线程是串行队列），造成卡死的情况。
>所以，我们应该另外开一个子线程让其负责下载：

```
- (void)startDownloading
{
    //先清空现有图片
    self.image = nil;   

    if (self.imageURL) {        

        //转动的小动画，提示正在下载
       [self.spinner startAnimating];
        NSURLRequest *request = [NSURLRequest requestWithURL:self.imageURL];
        NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
        NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
        NSURLSessionDownloadTask *task = [session downloadTaskWithRequest:request completionHandler:^(NSURL * _Nullable location, NSURLResponse * _Nullable response, NSError * _Nullable error) {            

            if(!error){
               
                //判断URL是否被更改，因为这是一个异步操作，无法保证在下载过程中一定能保持原来的数据
                if ([request.URL isEqual:self.imageURL])
                {
                    //下载完成，拿到本地的路径
                    UIImage *image = [UIImage imageWithData:[NSData dataWithContentsOfURL:location]];                    
                    //获得主队列
                    dispatch_async(dispatch_get_main_queue(), ^{

                        //在主队列更新UI
                        self.image = image;

                    });
                }
            }
        }];        

        [task resume];
    }
}
```

# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，在评论里留言即可。

笔者这两天会总结一下这一系列的Demo，发布到我的个人GitHub账号上去，以后就可以方便很多了~

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~

# 2016.7.12日更新：
---

笔者已经把目前为止整理的所有Demo(第二课到第十课)放入到了我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)仓库里。分为英文注释版和中文注释版(英文注释要少一点，嘿嘿)想要的小伙伴可以果断下载~ 如果有不知道怎么下载的小伙伴请联系我~




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



