---
title: 开源一个封装AFNetworking的网络框架 - SJNetwork
tags: [iOS,Objective-C]
categories: Production
---





# 介绍

该框架是一个通用的网络层，可以供给不同app的业务层调用。该框架封装了AFNetworking，而且有些地方借鉴了YTKNetwork的设计思路：以对象的形式封装并管理请求。


它在功能上支持：

- 发送请求方法为GET，POST，PUT，DELETE的普通网络请求的功能
- 上传图片功能（单张与多张上传，指定上传前的图片压缩比率）
- 下载功能（支持断点续传和后台下载）
- 缓存管理（写入，读取，清除缓存，计算大小）功能
- 请求管理（查看正在进行的请求的状态，请求的单个与批量取消）功能
- 设置请求体里的默认键值对（eg.需要添加版本号进行版本控制）
- 添加请求头（eg.针对一些需要使用token的服务）
- 设置服务器地址
- 设置debug模式（调试时打印出便于调试的log，比如读取缓存失败的具体原因）


GitHub链接：[SJNetwork](https://github.com/knightsj/SJNetwork)

>里面附有demo

<!-- more -->


# 架构

在看架构图之前，先简单介绍一下该框架里每个类的职责:



## 职责划分

| 类名                              | 职责                                       |
| :------------------------------ | :--------------------------------------- |
| SJNetwork                       | 总头文件，只需要引入该文件即可使用该框架所有功能                 |
| SJNetworkProtocol               | 定制了请求结束后的处理方法，今后可能还会扩展                   |
| SJNetworkHeader                 | 定义了回调block和一些枚举类型                        |
| SJNetworkManager                | 与业务层直接对接的类，包含了除配置接口外所有关于网络请求功能的接口        |
| SJNetworkBaseEngine             | 所有负责发送请求类的基类                             |
| SJNetworkRequestEngine          | 发送（GET,POST,PUT,DELETE）请求的类：支持设置缓存有效期，读，写和清理缓存 |
| SJNetworkUploadEngine           | 发送上传请求的类：支持设置图片类型和压缩上传，批量上传              |
| SJNetworkDownloadEngine         | 发送下载请求的类：支持断点续传和后台下载                     |
| SJNetworkRequestModel           | 请求对象类：持有某个网络请求的一些数据；比如请求url，请求体等）        |
| SJNetworkCacheManager           | 缓存处理类：缓存的写入，读取，删除                        |
| SJNetworkConfig                 | 配置类：配置服务器地址，debug模式等                     |
| SJNetworkUtils                  | 工具类：可以用于生成缓存路径，app版本号等                   |
| SJNetworkRequestPool            | 请求对象池：用于存放正在进行的请求对象                      |
| SJNetworkCacheInfo              | 缓存元数据：记录其对应缓存数据的信息（版本号，缓存过期时间）           |
| SJNetworkDownloadResumeDataInfo | 未下载完成数据的元数据：记录未下载完成数据的信息（已经下载的比例，下载数据的总长度，已经下载数据的长度） |



## 架构图

![architecture](https://user-gold-cdn.xitu.io/2017/12/26/1609352038129d87?w=839&h=539&f=png&s=45872)

>从架构图中可以看出：
- 业务方调用``SJNetworkManager``的接口来发送请求（或进行操作请求等操作），而实际进行工作的类其实是``SJNetworkRequestEngine``,``SJNetworkUploadEngine``,``SJNetworkDownloadEngine``，``SJNetworkCacheManager``这些类。
- 所有的请求都会被封装成一个``SJNetworkRequestModel``实例来管理，其管理者为``SJNetworkRequestPool``。
- 在该框架内，``SJNetworkConfig``和``SJNetworkUtils``是可以在任意地方调用的，因为要经常获取用户所做的一些配置以及调用一些常用的工具类方法。



# 使用方法



**Step1:下载与导入框架**



通过Pod:

```pod 'SJNetwork'```



或者 手动将 ``SJNetwork``文件夹拖入到工程里面。

>因为代码刚上传到Cocoapods，所以很可能还没有审核通过。



**Step2:引入头文件：**

```objective-c
#import "SJNetwork.h"
```





# 功能介绍



## 基本的配置



因为配置对象是一个单例（``SJNetworkConfig``），所以可以在项目任何地方来使用。看一下该框架支持哪些配置项：



### 服务器地址：

```objective-c
[SJNetworkConfig sharedConfig].baseUrl = @"http://v.juhe.cn";
```





### 默认参数:

```objective-c
[SJNetworkConfig sharedConfig].defailtParameters = @{@"app_version":[SJNetworkUtils appVersionStr],
                                                        @"platform":@"iOS"};
```

> 默认参数会拼接在所有请求的请求体中；
>
> 如果是GET请求，则拼接在url里面。





### 超时时间：

```objective-c
[SJNetworkConfig sharedConfig].timeoutSeconds = 30;
```

> 超时时间默认为20s。



### Debug模式：

```objective-c
[SJNetworkConfig sharedConfig].debugMode = YES;//默认为NO
```

> 如果设置debug模式为YES，则会打印出很多详细的log，便于调试。
>
> 如果设置为NO，则没有log。
>



### 添加请求头键值对：

```objective-c
[[SJNetworkConfig sharedConfig] addCustomHeader:@{@"token":@"2j4jd9s74bfm9sn3"}];
```

或者

```objective-c
[[SJNetworkManager sharedManager] addCustomHeader:@{@"token":@"2j4jd9s74bfm9sn3"}];//实际上调用了SJNetworkConfig的addCustomHeader方法
```



> 添加的请求头键值对会自动添加到所有的请求头中；
>
> 如果键值对原来不存在，则添加；如果原来存在，则替换原有的。





## 普通网络请求



在这里定义GET，POST，PUT，DELETE请求为普通的网络请求，是由``SJNetworkRequestManager``实现的。所有的这些普通的网络请求都支持写入和读取缓存，但是默认是不支持的，由用户来决定是否写入，读取缓存。



发送一个不支持写入和读取缓存的POST请求：

```objective-c
[[SJNetworkManager sharedManager] sendPostRequest:@"toutiao/index"
                                       parameters:@{@"type":@"top",
                                                    @"key" :@"0c60"}
                                          success:^(id responseObject) {

      NSLog(@"request succeed:%@",responseObject);

  } failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode) {

      NSLog(@"request failed:%@",error);
  }];
```



发送一个支持写入有效时间为180秒并在缓存有效时读取缓存的POST请求：

```objective-c
[[SJNetworkManager sharedManager] sendPostRequest:@"toutiao/index"
                                       parameters:@{@"type":@"top",
                                                    @"key" :@"0c60"}
                                        loadCache:YES
                                    cacheDuration:180
                                          success:^(id responseObject) {

     NSLog(@"request succeed:%@",responseObject);

 } failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode) {

     NSLog(@"request failed:%@",error);
 }];
```



> cacheDuration：缓存有效的时间，单位为秒。
>
> - 如果大于0，则进行缓存。
> - 如果小于等于0，则不进行缓存。
>
> loadCache：如果设置为YES，则在发起请求前，先查看是否缓存有效（如果设置为NO，则无论有没有缓存，都进行网络请求）：
>
> - 如果缓存存在并有效，则返回缓存，不进行网络请求；
> - 如果缓存不存在，或者存在但失效（时间过期）则删除缓存（如果缓存存在）并进行网络请求。





完整的带有缓存判断的普通网络请求的流程图：

![request](https://user-gold-cdn.xitu.io/2017/12/24/160873e31e97cd9b?w=1037&h=617&f=png&s=64142)



## 缓存管理



缓存管理是由``SJNetworkCacheManager``的单例来实现的，功能分为缓存的读取，删除和计算。先来看一下缓存的读取：



### 缓存的读取



该框架支持单个缓存的读取和多个缓存的读取：

- 单个缓存的读取只返回某个缓存对象（字典，或数组）或nil。
- 多个缓存的读取返回的是一个数组或nil。

#### 单个缓存的读取：



如果知道这个缓存对应的请求url，method，请求体，就能尝试获取它所对应的缓存对象：

举个例子，如果想获取上面有写入缓存的网络请求的缓存，就可以用如下API：

```objective-c
[[SJNetworkManager sharedManager] loadCacheWithUrl:@"toutiao/index"
                                            method:@"POST"
                                        parameters:@{@"type":@"top",
                                                   @"key" :@"0c60"}
                                   completionBlock:^(id  _Nullable cacheObject) {
                               
    NSLog(@"%@",cacheObject);
                               
}];
```

>注意，在缓存的读取过程中会有以下几种情况： 
>- 如果这个请求对应的缓存不存在，则会从block里传过来nil。
>
> - 如果这个请求对应的缓存存在，但是失效了（有效期过了），则这个缓存就会被清除掉，并会在block里传过来nil。
>
> - 如果这个请求对应的缓存存在并有效，则会从block里传过来缓存对象。



#### 多个缓存的读取：



如果有些请求使用的是同一个url（但是不同的请求方法或者参数）并做了缓存，那么通过如下方法可以获取它们的缓存：

```objective-c
[[SJNetworkManager sharedManager] loadCacheWithUrl:@"toutiao/index"
                                   completionBlock:^(NSArray * _Nullable cacheArr) {
    NSLog(@"%@",cacheArr);
}];
```



如果有些请求使用的是同一个url以及请求方法，但是请求参数不同，那么通过如下方法可以获取它们的缓存（用数组保存）：

```objective-c
[[SJNetworkManager sharedManager] loadCacheWithUrl:@"toutiao/index"
                                            method:@"POST"
                                   completionBlock:^(NSArray * _Nullable cacheArr) {
     NSLog(@"%@",cacheArr);
}];
```



现在我们知道这个框架在缓存的读取上支持单个与批量读取，接下来看一下缓存的删除：





### 缓存的删除



同样地，该框架也支持缓存的单个与批量删除。


如果你想删除属于某个特定url，method，请求参数的请求的缓存，可以使用下面这个API：

```objective-c
[[SJNetworkManager sharedManager] clearCacheWithUrl:@"toutiao/index"
                                             method:@"POST"
                                         parameters:@{@"type":@"top",
                                                      @"key" :@"0c60"}
                                    completionBlock:^(BOOL isSuccess) {

     if (isSuccess) {
       NSLog(@"Clearing cache successfully!");
     }
}];
```



如果你想删除使用的是同一个url（但是不同的请求方法或者参数）的请求的缓存，可以使用下面这个API：

```objective-c
[[SJNetworkManager sharedManager] clearCacheWithUrl:@"toutiao/index"
                                    completionBlock:^(BOOL isSuccess) {

     if (isSuccess) {
       NSLog(@"Clearing cache successfully!");
     }
}];
```



如果你想删除使用同一个url和method，但是不同请求参数的的请求的缓存，可以使用下面这个API：

```objective-c
[[SJNetworkManager sharedManager] clearCacheWithUrl:@"toutiao/index"
                                             method:@"POST"
                                withCompletionBlock:^(BOOL isSuccess) {
     if (isSuccess) {
        NSLog(@"Clearing cache successfully!");
     }
  }];
```



看完了缓存的读取和删除，我们来看一下缓存的计算:



### 缓存的计算



缓存的计算只提供了一个接口，在block回调的时候会回传一个文件个数，所有缓存的大小，以及带有KB或MB的字符串：

```objective-c
[[SJNetworkManager sharedManager] calculateCacheSizeWithCompletionBlock:^(NSUInteger fileCount, NSUInteger totalSize, NSString *totalSizeString) {
        
        NSLog(@"file count :%lu and total size:%lu total size string:%@",(unsigned long)fileCount,(unsigned long)totalSize, totalSizeString);
 }];
```



> fileCount:缓存文件的个数，为整数
>
> total size：单位为字节
>
> totalSizeString：带有KB和MB转化的字符串：在1024*1024字节以内以KB为单位；以外以MB为单位。例如：```file count :5 and total size:1298609 total size string:1.2385 MB```
>
> **注意：**所计算的缓存包括所有普通请求的缓存以及未下载完成，以后需要继续下载的数据。



## 上传功能



上传图片的功能是由``SJNetworkUploadManager`类的单例实现的：支持上传单个与多个``UIImage``对象，可以设置压缩比率（不设置时默认为1，不压缩）。



### 单张图片，原图上传

上传单个UIImage对象，上传前不对图片进行压缩：



```objective-c
[[SJNetworkManager sharedManager]  sendUploadImageRequest:@"api"
                                               parameters:nil
                                                    image:image_1
                                                     name:@"color"
                                                 mimeType:@"png"
                                                 progress:^(NSProgress *uploadProgress) 
{

    self.progressView.observedProgress = uploadProgress;

} success:^(id responseObject) {

    NSLog(@"upload succeed");

} failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode, NSArray<UIImage *> *uploadFailedImages) {

    NSLog(@"upload failed, failed images:%@",uploadFailedImages);

}];
```





### 多张图片，压缩一半

上传多个UIImage对象，压缩比率为0.5：

```objective-c
[[SJNetworkManager sharedManager]  sendUploadImagesRequest:@"api"
                                                parameters:nil
                                                    images:@[image_1,image_2]
                                             compressRatio:0.5
                                                      name:@"images"
                                                  mimeType:@"jpg"
                                                  progress:^(NSProgress *uploadProgress) 
{

    self.progressView.observedProgress = uploadProgress;

} success:^(id responseObject) {

    NSLog(@"upload succeed");

} failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode, NSArray<UIImage *> *uploadFailedImages) {

    NSLog(@"upload failed, failed images:%@",uploadFailedImages);
}];
```
> 这里的mimeType可以设置为jpg/JPG, png/PNG, jpeg/JPEG，作为图片上传到服务器时的类型。需要注意的是，如果mimeType为png/PNG的时候，设置的压缩比率就是无效的，将会一定以原图大小上传。


### 忽略设置过的BaseUrl

考虑到上传图片的服务器可能与普通请求的服务器不同，特意增加了一个参数：``ignoreBaseUrl``。如果该布尔值设置为YES，则在```SJNetworkConfig```单例里面设置的``baseUrl``就会被忽略掉，用户需要在请求的第一个参数里面将完整的请求url写进去：

```objective-c
[[SJNetworkManager sharedManager]  sendUploadImagesRequest:@"http://uploads.im/api"
                                               ignoreBaseUrl:YES
                                                  parameters:nil
                                                      images:@[image_1,image_2]
                                               compressRatio:0.5
                                                        name:@"images"
                                                    mimeType:@"jpg"
                                                    progress:^(NSProgress *uploadProgress) 
  {

      self.progressView.observedProgress = uploadProgress;

  } success:^(id responseObject) {

      NSLog(@"upload succeed");

  } failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode, NSArray<UIImage *> *uploadFailedImages) {

      NSLog(@"upload failed, failed images:%@",uploadFailedImages);

  }];
```



还有一个方法，就是强制更改```SJNetworkConfig```单例设置的``baseUrl``:

```objective-c
[SJNetworkConfig sharedConfig].baseUrl = @"http://uploads.im";
[[SJNetworkManager sharedManager]  sendUploadImagesRequest:@"api"
                                             ignoreBaseUrl:NO
                                                parameters:nil
                                                    images:@[image_3,image_4]
                                             compressRatio:0.5
                                                      name:@"color"
                                                  mimeType:@"png"
                                                  progress:^(NSProgress *uploadProgress) 
{

    self.progressView.observedProgress = uploadProgress;

} success:^(id responseObject) {

    NSLog(@"upload succeed");

} failure:^(NSURLSessionTask *task, NSError *error, NSInteger statusCode, NSArray<UIImage *> *uploadFailedImages) {

    NSLog(@"upload failed, failed images:%@",uploadFailedImages);

}];
```



虽然看上去不是很优雅，但却也是可行的：在请求所有普通网络请求之前将baseUrl再次改回去即可。

暂时该框架还无法支持多个baseUrl的功能，以后如果有研究到的话就会添加上去。







## 下载功能



下载功能是由``SJNetworkDownloadManager``的单例来实现的，支持断点续传以及后台下载。



- 如果设置为支持后台下载，则在内部生成的task类为：``NSURLSessionDownloadTask``。在手机退出前台，进入后台后后仍然可以下载。
- 如果设置为不支持后台下载，则在内部生成的task类为：``NSURLSessionDataTask``。在手机退出前台后无法继续下载，但是通过框架内部的**自动恢复下载机制**，在回到前台后就会继续之前的下载。而且结合了``NSOutputStream``实例，将下载下来的数据一点一点的写在沙盒里面，减少了内存的压力，也就是支持大文件下载。
- 如果支持断点续传，则由于断网或者取消请求等造成的下载失败后会保存未下载完成的数据。在后来启动该下载任务后，会继续下载。
- 如果不支持断点续传，则不会保留未下载完成的数据。在后来启动该下载后只能从头开始下载。

综上会是由四种情况：


|         | 支持断点续传 | 不支持断点续传 |
| ------- | ------ | ------- |
| 支持后台下载  | ✅      | ✅       |
| 不支持后台下载 | ✅      | ✅       |




**默认配置**为：支持断点续传，不支持后台下载（因为除非是音乐视频类等特殊app，后台下载的操作可能会被Apple拒掉）。



### 下载接口



#### 默认的下载功能（支持断点续传，不支持后台下载）：

```objective-c
[[SJNetworkManager sharedManager] sendDownloadRequest:@"wallpaper.jpg"
                                     downloadFilePath:_imageFileLocalPath
                                             progress:^(NSInteger receivedSize, NSInteger expectedSize, CGFloat progress)
{
       self.progressView.progress = progress;

} success:^(id responseObject) {

      NSLog(@"Download succeed!");

} failure:^(NSURLSessionTask *task, NSError *error, NSString *resumableDataPath) {

      NSLog(@"Download failed!");

}];
```



> 如果支持断点续传，在返回失败的回调里面会传过来未下载完成数据的路径：``resumableDataPath``。



#### 不支持断点续传，不支持后台下载：

```objective-c
[[SJNetworkManager sharedManager] sendDownloadRequest:@"half-eatch.jpg"
                                     downloadFilePath:_imageFileLocalPath
                                            resumable:NO
                                    backgroundSupport:NO
                                             progress:^(NSInteger receivedSize, NSInteger expectedSize, CGFloat progress) 
{

    self.progressView.progress = progress;

} success:^(id responseObject) {

    NSLog(@"Download succeed!");

} failure:^(NSURLSessionTask *task, NSError *error, NSString *resumableDataPath) {

    NSLog(@"Download failed!");
    
}];
```



#### 支持断点续传，支持后台下载：

```objective-c
[[SJNetworkManager sharedManager] sendDownloadRequest:@"universe.jpg"
                                     downloadFilePath:_imageFileLocalPath
                                            resumable:YES
                                    backgroundSupport:YES
                                             progress:^(NSInteger receivedSize, NSInteger expectedSize, CGFloat progress)
{

    self.progressView.progress = progress;

} success:^(id responseObject) {

     NSLog(@"Download succeed!");

} failure:^(NSURLSessionTask *task, NSError *error, NSString *resumableDataPath) {

    NSLog(@"Download failed!");

}];
```



#### 不支持断点续传，支持后台下载：

```objective-c
[[SJNetworkManager sharedManager] sendDownloadRequest:@"iceberg.jpg"
                                     downloadFilePath:_imageFileLocalPath
                                            resumable:NO
                                    backgroundSupport:YES
                                             progress:^(NSInteger receivedSize, NSInteger expectedSize, CGFloat progress)
{

    self.progressView.progress = progress;

 } success:^(id responseObject) {

     NSLog(@"Download succeed!");

 } failure:^(NSURLSessionTask *task, NSError *error, NSString *resumableDataPath) {

      NSLog(@"Download failed!");

 }];
```



和上传一样，下载接口也都支持是否忽略baseUrl：



```objective-c
[[SJNetworkManager sharedManager] sendDownloadRequest:@"http://oih3a9o4n.bkt.clouddn.com/wallpaper.jpg"
                                        ignoreBaseUrl:YES
                                     downloadFilePath:_imageFileLocalPath
                                             progress:^(NSInteger receivedSize, NSInteger expectedSize, CGFloat progress)
{
      self.progressView.progress = progress;

} success:^(id responseObject) {

      NSLog(@"Download succeed!");

} failure:^(NSURLSessionTask *task, NSError *error, NSString *resumableDataPath) {

      NSLog(@"Download failed!");

}];
```





### 下载的暂停，恢复和取消



所有的下载请求都支持暂停，恢复和取消操作。并且这些操作都支持单独与批量操作：



#### 下载的暂停



暂停单独的下载请求：

```objective-c
[[SJNetworkManager sharedManager] suspendDownloadRequest:@"universe.jpg"];
```



暂停多个下载请求：

```objective-c
[[SJNetworkManager sharedManager] suspendDownloadRequests:@[@"universe.jpg",@"wallpaper.jpg"]];
```



暂停所有下载请求：

```objective-c
[[SJNetworkManager sharedManager] suspendAllDownloadRequests];
```



#### 下载的恢复



恢复单独的正在暂停的下载请求：

```objective-c
[[SJNetworkManager sharedManager] resumeDownloadReqeust:@"universe.jpg"];
```



恢复多个正在暂停的下载请求：

```objective-c
[[SJNetworkManager sharedManager] resumeDownloadReqeusts:@[@"universe.jpg",@"wallpaper.jpg"]];
```



恢复所有正在暂停的下载请求：

```objective-c
[[SJNetworkManager sharedManager] resumeAllDownloadRequests];
```





#### 下载的取消



取消单独的下载请求：

```objective-c
[[SJNetworkManager sharedManager] cancelDownloadRequest:@"universe.jpg"];
```



取消多个下载请求：

```objective-c
[[SJNetworkManager sharedManager] cancelDownloadRequests:@[@"universe.jpg",@"wallpaper.jpg"]];
```



取消所有下载请求：

```objective-c
[[SJNetworkManager sharedManager] cancelAllDownloadRequests];
```







## 请求的管理



在该框架中，无论是普通的下载请求，上传请求和下载请求，在发送请求之前都将用户传入的参数保存在专门的请求对象``SJNetworkRequestModel``的实例里面。而这些实例的管理工作交给了``SJNetworkRequestPool``类的单例：

- 在请求开始之前，将请求实例放入其中的一个字典里管理。
- 当请求结束以后，将所对应的请求实例移除。

除了添加和移除请求对象以外，``SJNetworkRequestPool``对请求的管理还包括：

- 对正在进行的请求状况的查询
- 对正在进行的请求的取消。



### 请求状况的查询



#### 是否仍然有正在进行的请求：

```objective-c
BOOL remaining =  [[SJNetworkManager sharedManager] remainingCurrentRequests];
if (remaining) {
    NSLog(@"There is remaining request");
}
```



#### 正在进行的请求个数：

```objective-c
NSUInteger count = [[SJNetworkManager sharedManager] currentRequestCount];
if (count > 0) {
    NSLog(@"There is %lu requests",(unsigned long)count);
}
```



#### 打印所有正在进行的请求对象：

```objective-c
[[SJNetworkManager sharedManager] logAllCurrentRequests];
```







### 请求的取消



请求的取消也分为单个和批量的取消：



#### 取消某个请求：

```objective-c
[[SJNetworkManager sharedManager] cancelCurrentRequestWithUrl:@"toutiao/index"
                                                           method:@"POST"
                                                       parameters:@{@"type":@"top",
                                                                    @"key" :@"0c60"}];
```



#### 取消相同url的请求：



```objective-c
[[SJNetworkManager sharedManager] cancelCurrentRequestWithUrl:@"toutiao/index"];
```



#### 取消多个指定url的请求：

```objective-c
[[SJNetworkManager sharedManager] cancelDownloadRequests:@[@"toutiao/index",@"weixin/query"]];
```



#### 取消所有正在进行的请求：

```objective-c
[[SJNetworkManager sharedManager] cancelAllCurrentRequests];
```

 	



## Log输出



如果将debug模式设置为YES，则会打印出很多便于调试的log：

```objective-c
[SJNetworkConfig sharedConfig].debugMode = YES;
```



### 请求对象的log



由于重写了``SJNetworkRequestModel``的``description``方法，所以在打印该对象的时候，普通的网络请求，上传请求，下载请求都有属于自己的log，在这里举一个普通请求的log：



```objective-c
{
   <SJNetworkRequestModel: 0x6040001fc100>
   type:             ordinary request
   method:          GET
   url:             http://v.juhe.cn/toutiao/index
   parameters:      {
    "app_version" = "1.0";
    key = 0c60;
    platform = iOS;
    type = top;
}
   loadCache:       YES
   cacheDuration:   5 seconds
   requestIdentifer:b4b36793efabad54a14389cf09bc8133_a6a72ddee1dd86825cb5707c500784f5_7b65261ff298c6a386c89a632bd17b39_30c9b994c268547f38a2f9af6f8c171f
   task:            <__NSCFLocalDataTask: 0x7f8e075320a0>{ taskIdentifier: 1 } { completed }
} 
```





### 请求和缓存的log



举一个需要获取缓存的网络请求但是遇到缓存过期的情况：

```objective-c
=========== Load cache info failed, reason:Cache is expired, begin to clear cache...
=========== Load cache failed: Cache info is invalid 
=========== Faild to load cache, start to sending network request...
=========== Start requesting...
=========== url:http://v.juhe.cn/toutiao/index
=========== method:GET
=========== parameters:{
    "app_version" = "1.0";
    key = 0c60;
    platform = iOS;
    type = top;
}
=========== Request succeed! 
=========== Request url:http://v.juhe.cn/toutiao/index
=========== Response object:{
  code = 200,
  msg = "",
  data = {}
}
=========== Write cache succeed!
=========== cache object: {
  code = 200,
  msg = "",
  data = {}
}
=========== Cache path: /Users/****/***/***/*******.cacheData
=========== Available duration: 180 seconds
```





# 感谢

在调试这个框架的时候使用了很多网络资源，也看了好多文章，获得了很多帮助，所以不得不提一下：


## API服务

- 免费的接口服务：[聚合数据](https://www.juhe.cn/)
- 上传图片的接口服务：[Uploads](http://uploads.im)
- 下载图片的图床：[七牛云开发者平台](https://portal.qiniu.com/)


## 使用和参考的框架

- [AFNetworking](https://github.com/AFNetworking/AFNetworking)
- [YTKNetwork](https://github.com/yuantiku/YTKNetwork)



## 参考文章



- [iOS开发网络篇之文件下载、大文件下载、断点下载](http://www.jianshu.com/p/f65e32012f07)
- [iOS使用NSURLSession进行下载（包括后台下载，断点下载）](http://www.jianshu.com/p/1211cf99dfc3)
- [iOS开发之网络编程 --2. NSURLSessionDownloadTask文件下载](https://www.cnblogs.com/goodboy-heyang/p/5195806.html)
- [MCDownloadManager ios文件下载管理器](http://www.cnblogs.com/machao/p/5864251.html)
- [华山论剑之浅谈iOS的文件下载,断点下载(基于NSURLSession的网络请求)](http://www.jianshu.com/p/5e6630e999fa?utm_campaign=hugo&utm_medium=reader_share&utm_content=note&utm_source=weixin-friends)



# 框架传送门

GitHub地址：[SJNetwork](https://github.com/knightsj/SJNetwork)

>里面附有demo


# 最后的话



现在社区里二次封装AFNetworking，上传图片，以及下载器的框架有很多，但是因为总是想写一个**属于自己代码风格的框架**，而且网络层对我自己还是有些挑战的，所以想试一试。



除去中间间隔的时间，整个框架基本成型的时间用了有一个多月，但是写全英文的注释，最后的重构（修改了架构，分离了一些类），命名的规范，修改bug，优化等事情又花去了半个月。特别是由于自己对下载这一块不熟，尤其是断点续传，后台下载这两方面更是没有实战经验，在写的时候也花了不少时间。



希望能多给出宝贵意见和建议，我自己发现有不足的地方也会更新~



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



