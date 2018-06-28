---
title: 斯坦福大学iOS开发公开课总结（十六）：模态视图，UITextField，UImagePickerController，在Flickr添加摄影师照片Demo
tags: [iOS,Objective-C]
categories: iOS
---

本节课讲解了模态视图，文本框，UImagePickerController的相关知识，并延续了上一节课的Demo，添加了照相并存储照片的功能。


# 模态视图
------

模态视图不同于左右滑入的视图，它是从下往上，覆盖整个屏幕的视图。每次滑入都会重新新建一个控制器。通常用于修改信息等操作。

显示模态视图：

```
  [self presentViewController:(nonnull UIViewController *) animated:(BOOL) completion:^{}];
```

让模态视图消失：
```
- (void)dissmissViewControllerAnimated:(BOOL)animated completion:(void(^))block;
```
>注意:该消息是发送给present该模态视图的控制器，而不是该模态视图本身。因此，调用方法应该是：

```
[self.presentingViewController dissmissViewControllerAnimated:YES ...];
```

<!-- more -->

# UITextField
------

UITextField是文本框，可以用来输入文字，类似于UILabel。

### 让键盘出现和消失：
```
[textField becomeFirstResponder];
[textField resignFirstResponder];
```

## 代理方法：

#### 1. 当点击了确定按键,让键盘消失
```
- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    [textField resignFirstResponder];
    return YES;
}
```

#### 2. 当resignFirstResponder完成后执行：
```
- (void)textFieldDidEndEditing:(UITextField *)textField;
```

#### 3. 内容发生变化时收到通知：

注册这个广播，就可以收到该通知：
```
UIKIT_EXTERN NSString *const UITextFieldTextDidChangeNotification;
```

#### 4. 键盘
设置键盘的类型需要通过给实现``UITextInputTraits``的协议方法，通常是``UITextField``

```
@protocol UITextInputTraits <NSObject>

@optional
@property(nonatomic) UITextAutocapitalizationType autocapitalizationType; // 自动大写
@property(nonatomic) UIKeyboardType keyboardType;                         // default is UIKeyboardTypeDefault
@property(nonatomic) UIReturnKeyType returnKeyType;                       // 回车键，返回键，搜索键盘
@property(nonatomic,getter=isSecureTextEntry) BOOL secureTextEntry;       // 密码
@end
```

#### 5. 监听键盘的高度：
通过注册``UIKeyboard{will,did} {show hide}``广播来计算键盘弹出后的高度。


# Alert

Alert是用来提醒用户某些消息的控件，它会出现在屏幕的正中央。我们可以自定义它的显示消息和按钮。也可以设定点击某个按钮执行的操作。

## 初始化：
```
- (instancetype)initWithTitle:(nullable NSString *)title delegate:(nullable id<UIActionSheetDelegate>)delegate cancelButtonTitle:(nullable NSString *)cancelButtonTitle destructiveButtonTitle:(nullable NSString *)destructiveButtonTitle otherButtonTitles:(nullable NSString *)otherButtonTitles, ... NS_REQUIRES_NIL_TERMINATION NS_EXTENSION_UNAVAILABLE_IOS("Use UIAlertController instead.");
```

## 增加按钮
```
- (NSInteger)addButtonWithTitle:(nullable NSString *)title; 
```

## 显示在屏幕上:
```
- (void)showFromRect:(CGRect)rect inView:(UIView *)view animated:(BOOL)animated NS_AVAILABLE_IOS(3_2);
- (void)showInView:(UIView *)view;
```

## 处理点击事件：
```
- (void)actionSheet:(UIActionSheet *)actionSheet didDismissWithButtonIndex:(NSInteger)buttonIndex NS_DEPRECATED_IOS(2_0, 8_3) __TVOS_PROHIBITED;  // after animation
```

## 手动让其消失
```
- (void)dismissWithClickedButtonIndex:(NSInteger)buttonIndex animated:(BOOL)animated;
```

# UImagePickerController
-------------

UImagePickerController是用来选取图片，视频资源的控制器，也可以进行拍照。

## 使用步骤：
1. alloc/init, set delegate
2. 配置摄像头，照片库，用户是否可以编辑相片
3. 显示
4. 实现代理方法，获取媒体

## 检查硬件设备：
```
+ (BOOL)isSourceTypeAvailable:(UIImagePickerControllerSourceType)sourceType;  

typedef NS_ENUM(NSInteger, UIImagePickerControllerSourceType) {
    UIImagePickerControllerSourceTypePhotoLibrary,
    UIImagePickerControllerSourceTypeCamera,
    UIImagePickerControllerSourceTypeSavedPhotosAlbum
}

```

## 检查是否可以摄像：
```
+ (nullable NSArray<NSString *> *)availableMediaTypesForSourceType:(UIImagePickerControllerSourceType)sourceType;

```

返回的数组是否有相应的字段：
kUTTypeImage
kUTTypeMovie

## 允许用户编辑（裁剪）：
```
@property BOOL allowEditing;
```

## 获取了媒体（照片）后的代理方法
```
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info;
```

## 点击了取消后的代理方法：
```
- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker
{
    [self dismissViewControllerAnimated:YES completion:NULL];
}

```

# Demo 
------------

## Demo需求：
- 在原摄影师列表的第一行添加“我的照片”。
- 点击“我的照片”后，显示地图上“我”所照照片的地点。
- 点击大头针，显示相应照片详情。
- 在导航栏右上角显示照相机按钮。
- 点击照相机按钮，从底部弹出添加照片的页面。
- 点击“Take Photo”,启动相机，照相并可以裁剪。
- 裁剪后，回到添加剂照片的页面。设置标题和副标题后，保存照片。
- 回到地图页面，自动添加刚才所增加照片的大头针。

## Demo效果图

![左：添加照片 | 中：在地图上标注位置 | 右：点击查看大图](http://upload-images.jianshu.io/upload_images/859001-b1f9ecf0f562f5c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 重要知识点和代码段

#### 1. 在添加照片页面显示后判断是否可以拍照
```

- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    
    if (![[self class] canAddPhoto]) {
        [self fatalAlert:@"Sorry, this device cannot add a photo."];
    } else {
        [self.locationManager startUpdatingLocation];//开始更新位置
    }
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    //页面消失前，停止更新位置，避免耗能    
    [self.locationManager stopUpdatingLocation];
}

/**
 *  判断当前设备是否可以添加照片
 *
 *  @return 是否
 */

+ (BOOL)canAddPhoto
{
    //判断相机是否可用
    if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
        //返回支持媒体类型的数组
        NSArray *availableMediaTypes = [UIImagePickerController availableMediaTypesForSourceType:UIImagePickerControllerSourceTypeCamera];

        //判断数组里有无照片类型
        if ([availableMediaTypes containsObject:(NSString *)kUTTypeImage]) {

            //判断可以支持照片类型后，判断当前设备是否可以获取位置信息
            if ([CLLocationManager authorizationStatus] != kCLAuthorizationStatusRestricted) {

                return YES;
            }
        }
    }
    return NO;
}

- (void)fatalAlert:(NSString *)msg
{
    [[[UIAlertView alloc] initWithTitle:@"Add Photo"
                                message:msg
                               delegate:self
                      cancelButtonTitle:nil
                      otherButtonTitles:@"Cancel", nil] show];
}

```

#### 2. CLLocationManager的初始化和使用

```
- (CLLocationManager *)locationManager
{
    if (!_locationManager) {

        //1. 初始化
        CLLocationManager *locationManager = [[CLLocationManager alloc] init];

        //2. 设置代理
        locationManager.delegate = self;

        //3. 设置精度
        locationManager.desiredAccuracy = kCLLocationAccuracyBest;
        _locationManager = locationManager;
        //4. iOS8以上要调用，否则无法监听位置！
        [_locationManager requestAlwaysAuthorization];
        [_locationManager requestWhenInUseAuthorization];
    }
    return _locationManager;
}

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
{
    //获取最后得到的位置信息（最准确）
    self.location = [locations lastObject];
}

- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error
{
    //获取错误码
    self.locationErrorCode = error.code;
}
```

#### 3. UIImagePickerController的初始化和使用

```

/**
 *  点击了“Take Photo 添加照片”
 */
- (IBAction)takePhoto
{
    //1. 初始化
    UIImagePickerController *uiipc = [[UIImagePickerController alloc] init];

    //2. 设置代理
    uiipc.delegate = self;

    //3. 获取图片媒体
    uiipc.mediaTypes = @[(NSString *)kUTTypeImage];
    uiipc.sourceType = UIImagePickerControllerSourceTypeCamera | UIImagePickerControllerSourceTypePhotoLibrary;

    //4. 允许裁剪
    uiipc.allowsEditing = YES;

    //5. 弹出UIImagePickerController
    [self presentViewController:uiipc animated:YES completion:NULL];

}

/**
 *  拍照成功
 *
 *  @param picker 当前的UIImagePickerController
 *  @param info   获取的照片信息
 */

- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
{
    //获取裁剪后的图片
    UIImage *image = info[UIImagePickerControllerEditedImage];

    //如果无法获取裁剪后的图片，获取原图
    if (!image) image = info[UIImagePickerControllerOriginalImage];

    //更新当前的image属性
    self.image = image;

    [self dismissViewControllerAnimated:YES completion:NULL];
}

/**
 *  点击了取消
 *
 *  @param picker 当前的UIImagePickerController
 */
- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker
{
    [self dismissViewControllerAnimated:YES completion:NULL];
}
```

#### 4. 判断是否执行某个Segue的Identifier

```
- (BOOL)shouldPerformSegueWithIdentifier:(NSString *)identifier sender:(id)sender
{

    if ([identifier isEqualToString:@"Do Add Photo"]) {
        if (!self.image) {
            //无照片
            [self alert:@"No photo taken!"];
            return NO;
            
        } else if (![self.titleTextField.text length]) {

            //无标题
            [self alert:@"Title required!"];
            return NO;
       
        } else if (!self.location) {
            
            //没有获取到位置信息
            switch (self.locationErrorCode) {

                case kCLErrorLocationUnknown:
                    [self alert:@"Couldn't figure out where this photo was taken (yet)."]; break;

                case kCLErrorDenied:
                    [self alert:@"Location Services disabled under Privacy in Settings application."]; break;

                case kCLErrorNetwork:
                    [self alert:@"Can't figure out where this photo is being taken.  Verify your connection to the network."]; break;

                default:
                    [self alert:@"Cant figure out where this photo is being taken, sorry."]; break;
            }

            return NO;

        } else {
            return YES;
        }

    } else {

        return [super shouldPerformSegueWithIdentifier:identifier sender:sender];
    }
}

- (void)alert:(NSString *)msg
{

    [[[UIAlertView alloc] initWithTitle:@"Add Photo"
                                message:msg
                               delegate:nil
                      cancelButtonTitle:nil
                      otherButtonTitles:@"Cancel", nil] show];
}

```
>在添加照片后，我们需要将该页面取消并储存相应的数据。但由于业务需求，如果想要储存数据的前提下取消页面的话，那么就需要在取消页面之前来判断当前的数据是否满足储存的条件：是否有照片；是否设置了标题；是否获取了位置信息等。

#### 5. 确定页面可以被取消后，在页面被取消前储存数据：

```
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender
{

    if ([segue.identifier isEqualToString:@"Do Add Photo"]) {

        NSManagedObjectContext *context = self.photographerTakingPhoto.managedObjectContext;

        if (context) {

            Photo *photo = [NSEntityDescription insertNewObjectForEntityForName:@"Photo"  inManagedObjectContext:context];

            photo.title = self.titleTextField.text;
            photo.subtittle = self.subtitleTextField.text;
            photo.whoTook = self.photographerTakingPhoto;
            photo.latitude = @(self.location.coordinate.latitude);
            photo.longitude = @(self.location.coordinate.longitude);
            photo.imageURL = [self.imageURL absoluteString];
            photo.thumbnailURL = [self.thumbnailURL absoluteString];            

            self.addedPhoto = photo;         
            self.imageURL = nil;
            self.thumbnailURL = nil;
        }
    }
}

```

#### 6. 获取新增图片的本地路径

```
/**
 *  获取图片的本地URL
 *
 *  @return 图片的本地URL
 */

- (NSURL *)imageURL
{
    if (!_imageURL && self.image) {
        NSURL *url = [self uniqueDocumentURL];
        if (url) {
            //UIImage -> NSData
            NSData *imageData = UIImageJPEGRepresentation(self.image, 1.0);
            //将data写入url
            if ([imageData writeToURL:url atomically:YES]) {
                //如果写入成功，更新imageURL属性
                _imageURL = url;
            }
        }
    }
    return _imageURL;
}

/**
 *  获取图片缩略图的本地URL
 *
 *  @return 缩略图的本地URL
 */

- (NSURL *)thumbnailURL
{

    NSURL *url = [self.imageURL URLByAppendingPathExtension:@"thumbnail"];
    if (![_thumbnailURL isEqual:url]) {

        _thumbnailURL = nil;
        if (url) {
            //以某Size压缩图片（详情请看本Demo添加的image分类）
            UIImage *thumbnail = [self.image imageByScalingToSize:CGSizeMake(75, 75)];

            //0.5倍压缩
            NSData *imageData = UIImageJPEGRepresentation(thumbnail, 0.5);

            if ([imageData writeToURL:url atomically:YES]) {
                _thumbnailURL = url;
            }
        }
    }
    return _thumbnailURL;
}

/**
 *  以时间来生成唯一本地路径
 *
 *  @return 本地路径
 */

- (NSURL *)uniqueDocumentURL
{

    NSArray *documentDirectories = [[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask];

    NSString *unique = [NSString stringWithFormat:@"%.0f", floor([NSDate timeIntervalSinceReferenceDate])];
    return [[documentDirectories firstObject] URLByAppendingPathComponent:unique];

}

```

>在这里，我们用当前的时间来拼接Document路径，获得了图片的唯一地址。

笔者今天在公司附近拍了一张照片来验证效果：一张南京东路苹果旗舰店的照片，不过定位比较不准。可能是由于周围高楼比较多，而且定位时间不够长的关系。

# 最后的话
----
如果哪位小伙伴想拿到本文Demo的代码请不要客气，可以进入我的[GitHub](https://github.com/Shijie0111/Stanford_iOS_Lecture_DemoBundle)下载哦~    这一系列到现在为止的所有Demo都在里面，分为英文注释版本和中文注释版本两种。

如果嫌麻烦的童鞋可以在留言留下邮箱，笔者会将Demo包发给你~

十分欢迎给笔者的代码和文笔抛出宝贵的意见和建议~

本文为笔者原创，如需转载，请事先与笔者交涉~


