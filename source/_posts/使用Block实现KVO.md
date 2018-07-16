---
title: 使用Block实现KVO
tags: [iOS,Objective-C]
categories: Production
---




在iOS开发中，我们可以通过KVO机制来监听某个对象的某个属性的变化。

用过KVO的同学都应该知道，KVO的回调是以代理的形式实现的：在给某个对象添加观察以后，需要在另外一个地方实现回调代理方法。这种设计给人感觉比较分散，因此突然想试试用Block来实现KVO，将添加观察的代码和回调处理的代码写在一起。在学习了[ImplementKVO](https://github.com/okcomp/ImplementKVO)的实现以后，自己也写了一个：[SJKVOController](https://github.com/knightsj/SJKVOController)

![使用Block来实现KVO](http://oih3a9o4n.bkt.clouddn.com/sjkvocontrollerblogimage.png)

# SJKVOController的用法

只需要引入``NSObject+SJKVOController.h``头文件就可以使用SJKVOController。
先看一下它的头文件：
```objc
#import <Foundation/Foundation.h>
#import "SJKVOHeader.h"

@interface NSObject (SJKVOController)


//============== add observer ===============//
- (void)sj_addObserver:(NSObject *)observer forKeys:(NSArray <NSString *>*)keys withBlock:(SJKVOBlock)block;
- (void)sj_addObserver:(NSObject *)observer forKey:(NSString *)key withBlock:(SJKVOBlock)block;


//============= remove observer =============//
- (void)sj_removeObserver:(NSObject *)observer forKeys:(NSArray <NSString *>*)keys;
- (void)sj_removeObserver:(NSObject *)observer forKey:(NSString *)key;
- (void)sj_removeObserver:(NSObject *)observer;
- (void)sj_removeAllObservers;


//============= list observers ===============//
- (void)sj_listAllObservers;

@end
```

<!-- more -->

>从上面的API可以看出，这个小轮子：
1. 支持一次观察同一对象的多个属性。
2. 可以一次只观察一个对象的一个属性。
3. 可以移除对某个对象对多个属性的观察。
4. 可以移除对某个对象对某个属性的观察。
5. 可以移除某个观察自己的对象。
6. 可以移除所有观察自己的对象。
6. 打印出所有观察自己的对象的信息，包括对象本身，观察的属性，setter方法。

下面来结合Demo讲解一下如何使用这个小轮子：


在点击上面两个按钮中的任意一个，增加观察：

一次性添加：
```objc
- (IBAction)addObserversTogether:(UIButton *)sender {
    
    NSArray *keys = @[@"number",@"color"];
    
    [self.model sj_addObserver:self forKeys:keys withBlock:^(id observedObject, NSString *key, id oldValue, id newValue) {
        
        if ([key isEqualToString:@"number"]) {
            
            dispatch_async(dispatch_get_main_queue(), ^{
                self.numberLabel.text = [NSString stringWithFormat:@"%@",newValue];
            });
            
        }else if ([key isEqualToString:@"color"]){
            
            dispatch_async(dispatch_get_main_queue(), ^{
                self.numberLabel.backgroundColor = newValue;
            });
        }
        
    }];
}
```

分两次添加：
```objc
- (IBAction)addObserverSeparatedly:(UIButton *)sender {
    
    [self.model sj_addObserver:self forKey:@"number" withBlock:^(id observedObject, NSString *key, id oldValue, id newValue) {
        
        dispatch_async(dispatch_get_main_queue(), ^{
            self.numberLabel.text = [NSString stringWithFormat:@"%@",newValue];
        });
        
    }];
    
    [self.model sj_addObserver:self forKey:@"color" withBlock:^(id observedObject, NSString *key, id oldValue, id newValue) {
        
        dispatch_async(dispatch_get_main_queue(), ^{
            self.numberLabel.backgroundColor = newValue;
        });
        
    }];
    
}
```

添加以后，点击最下面的按钮来显示所有的观察信息：

```objc
- (IBAction)showAllObservingItems:(UIButton *)sender {
    
    [self.model sj_listAllObservers];
}
```
输出：
```objc
SJKVOController[80499:4242749] SJKVOLog:==================== Start Listing All Observers: ==================== 
SJKVOController[80499:4242749] SJKVOLog:observer item:{observer: <ViewController: 0x7fa1577054f0> | key: color | setter: setColor:}
SJKVOController[80499:4242749] SJKVOLog:observer item:{observer: <ViewController: 0x7fa1577054f0> | key: number | setter: setNumber:}
```

>在这里我重写了description方法，打印出了每个观察的对象和key,以及setter方法。


现在点击更新按钮，则会更新model的number和color属性，从而触发KVO：
```objc
- (IBAction)updateNumber:(UIButton *)sender {
    
    //trigger KVO : number
    NSInteger newNumber = arc4random() % 100;
    self.model.number = [NSNumber numberWithInteger:newNumber];
    
    //trigger KVO : color
    NSArray *colors = @[[UIColor redColor],[UIColor yellowColor],[UIColor blueColor],[UIColor greenColor]];
    NSInteger colorIndex = arc4random() % 3;
    self.model.color = colors[colorIndex];
}
```

我们可以看到中间的Label上面显示的数字和背景色都在变化，成功实现了KVO：

![同时观察颜色和数字的变化](http://oih3a9o4n.bkt.clouddn.com/sjkvocontrolllergif1.gif)



现在我们移除观察，点击remove按钮

```objc
- (IBAction)removeAllObservingItems:(UIButton *)sender {
    [self.model sj_removeAllObservers];   
}
```

在移除了所有的观察者以后，则会打印出：
```objc
SJKVOController[80499:4242749] SJKVOLog:Removed all obserbing objects of object:<Model: 0x60000003b700>
```
而且如果在这个时候打印观察者list，则会输出：
```objc
SJKVOController[80499:4242749] SJKVOLog:There is no observers obserbing object:<Model: 0x60000003b700>
```

需要注意的是，这里的移除可以有多种选择：可以移某个对象的某个key，也可以移除某个对象的几个keys，为了验证，我们可以结合list方法来验证一下移除是否成功：

#### 验证1:在添加number和color的观察后，移除nunber的观察：
```objc
- (IBAction)removeAllObservingItems:(UIButton *)sender {
    [self.model sj_removeObserver:self forKey:@"number"];
}
```

在移除以后，我们调用list方法，输出：
```objc
SJKVOController[80850:4278383] SJKVOLog:==================== Start Listing All Observers: ====================
SJKVOController[80850:4278383] SJKVOLog:observer item:{observer: <ViewController: 0x7ffeec408560> | key: color | setter: setColor:}
```
现在只有color属性被观察了。看一下实际的效果：

![只观察颜色的变化](http://oih3a9o4n.bkt.clouddn.com/sjkvocontrolllergif2.gif)

我们可以看到，现在只有color在变，而数字没有变化了，验证此移除方法正确。



#### 验证2:在添加number和color的观察后，移除nunber和color的观察：

```objc
- (IBAction)removeAllObservingItems:(UIButton *)sender {
    
    [self.model sj_removeObserver:self forKeys:@[@"number",@"color"]];
}
```

在移除以后，我们调用list方法，输出：
```objc
SJKVOController[80901:4283311] SJKVOLog:There is no observers obserbing object:<Model: 0x600000220fa0>
```
现在color和number属性都不被观察了。看一下实际的效果：

![颜色和数字的变化都不再被观察](http://oih3a9o4n.bkt.clouddn.com/sjkvocontrolllergif3.gif)

我们可以看到，现在color和number都不变了，验证此移除方法正确。

OK，现在知道了怎么用SJKVOController，我下面给大家看一下代码：
# SJKVOController代码解析

先大致讲解一下SJKVOController的实现思路：
1. 为了减少侵入性，SJKVOController被设计为NSObject的一个分类。
2. SJKVOController仿照了KVO的实现思路，在添加观察以后在运行时动态生成当前类的子类，给这个子类添加被观察的属性的set方法并使用isa swizzle的方式将当前对象转换为当前类的子类的实现。
3. 同时，这个子类还使用了关联对象来保存一个“观察项”的set，每一个观察项封装了一次观察的行为（有去重机制）：包括观察自己的对象，自己被观察的属性，以及传进来的block。
4. 在当前类，也就是子类的set方法被调用的时候做三件事情：
    - 第一件事情是使用KVC来找出当前属性的旧值。
    - 第二件事情是调用父类（原来的类）的set方法（设新值）。
    - 第三件事是根据当前的观察对象和key，在观察项set里面找出对应的block并调用。

再来看一下这个小轮子的几个类：

- SJKVOController：实现KVO主要功能的类。
- SJKVOObserverItem：封装观察项的类。
- SJKVOTool：setter和getter的相互转换和相关运行时查询方法等。
- SJKVOError：封装错误类型。
- SJKVOHeader：引用了运行时的头文件。


 下面开始一个一个来讲解每个类的源码：


## SJKVOController

再看一下头文件：
```objc
#import <Foundation/Foundation.h>
#import "SJKVOHeader.h"

@interface NSObject (SJKVOController)

//============== add observer ===============//
- (void)sj_addObserver:(NSObject *)observer forKeys:(NSArray <NSString *>*)keys withBlock:(SJKVOBlock)block;
- (void)sj_addObserver:(NSObject *)observer forKey:(NSString *)key withBlock:(SJKVOBlock)block;


//============= remove observer =============//
- (void)sj_removeObserver:(NSObject *)observer forKeys:(NSArray <NSString *>*)keys;
- (void)sj_removeObserver:(NSObject *)observer forKey:(NSString *)key;
- (void)sj_removeObserver:(NSObject *)observer;
- (void)sj_removeAllObservers;

//============= list observers ===============//
- (void)sj_listAllObservers;

@end
```

每个方法的意思相信读者已经能看懂了，现在讲一下具体的实现。从``sj_addObserver:forKey withBlock:``开始：

### sj_addObserver:forKey withBlock:方法：
除去一些错误的判断，该方法作了下面几件事情：

#### 1.判断当前被观察的类是否存在与传入key对应的setter方法：

```objc
SEL setterSelector = NSSelectorFromString([SJKVOTool setterFromGetter:key]);
Method setterMethod = [SJKVOTool objc_methodFromClass:[self class] selector:setterSelector];
//error: no corresponding setter mothod
if (!setterMethod) {
     SJLog(@"%@",[SJKVOError errorNoMatchingSetterForKey:key]);
     return;
}
```
#### 2. 如果有，判断当前被观察到类是否已经是KVO类(在KVO机制中，如果某个对象一旦被观察，则这个对象就变成了带有包含KVO前缀的类的实例)。如果已经是KVO类，则将当前实例的isa指针指向其父类（最开始被观察的类）：

```objc
    //get original class(current class,may be KVO class)
    NSString *originalClassName = NSStringFromClass(OriginalClass);
    
    //如果当前的类是带有KVO前缀的类(也就是已经被观察到类),则需要将KVO前缀的类删除，并讲
    if ([originalClassName hasPrefix:SJKVOClassPrefix]) {
        //now,the OriginalClass is KVO class, we should destroy it and make new one
        Class CurrentKVOClass = OriginalClass;
        object_setClass(self, class_getSuperclass(OriginalClass));
        objc_disposeClassPair(CurrentKVOClass);
        originalClassName = [originalClassName substringFromIndex:(SJKVOClassPrefix.length)];
    }
```


#### 3. 如果不是KVO类（说明当前实例没有被观察），则创建一个带有KVO前缀的类，并将当前实例的isa指针指向这个新建的类：

```objc
    //create a KVO class
    Class KVOClass = [self createKVOClassFromOriginalClassName:originalClassName];
    
    //swizzle isa from self to KVO class
    object_setClass(self, KVOClass);
```

看一下如何新建一个新的类：
```objc
- (Class)createKVOClassFromOriginalClassName:(NSString *)originalClassName
{
    NSString *kvoClassName = [SJKVOClassPrefix stringByAppendingString:originalClassName];
    Class KVOClass = NSClassFromString(kvoClassName);
    
    // KVO class already exists
    if (KVOClass) {
        return KVOClass;
    }
    
    // if there is no KVO class, then create one
    KVOClass = objc_allocateClassPair(OriginalClass, kvoClassName.UTF8String, 0);//OriginalClass is super class
    
    // pretending to be the original class:return the super class in class method
    Method clazzMethod = class_getInstanceMethod(OriginalClass, @selector(class));
    class_addMethod(KVOClass, @selector(class), (IMP)return_original_class, method_getTypeEncoding(clazzMethod));
    
    // finally, register this new KVO class
    objc_registerClassPair(KVOClass);
    
    return KVOClass;
}
```

#### 4. 查看观察项set，如果这个set里面有已经保存的观察项，则需要新建一个空的观察项set，将已经保存的观察项放入这个新建的set里面：

```objc
    //if we already have some history observer items, we should add them into new KVO class
    NSMutableSet* observers = objc_getAssociatedObject(self, &SJKVOObservers);
    if (observers.count > 0) {
        
        NSMutableSet *newObservers = [[NSMutableSet alloc] initWithCapacity:5];
        objc_setAssociatedObject(self, &SJKVOObservers, newObservers, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        
        for (SJKVOObserverItem *item in observers) {
            [self KVOConfigurationWithObserver:item.observer key:item.key block:item.block kvoClass:KVOClass setterSelector:item.setterSelector setterMethod:setterMethod];
        }    
    }
```

看一下如何保存观察项的：
```objc
- (void)KVOConfigurationWithObserver:(NSObject *)observer key:(NSString *)key block:(SJKVOBlock)block kvoClass:(Class)kvoClass setterSelector:(SEL)setterSelector setterMethod:(Method)setterMethod
{
    //add setter method in KVO Class
    if(![SJKVOTool detectClass:OriginalClass hasSelector:setterSelector]){
        class_addMethod(kvoClass, setterSelector, (IMP)kvo_setter_implementation, method_getTypeEncoding(setterMethod));
    }
    
    //add item of this observer&&key pair
    [self addObserverItem:observer key:key setterSelector:setterSelector setterMethod:setterMethod block:block];
}
```

这里首先给KVO类增加了setter方法：

```objc
//implementation of KVO setter method
void kvo_setter_implementation(id self, SEL _cmd, id newValue)
{
    
    NSString *setterName = NSStringFromSelector(_cmd);
    NSString *getterName = [SJKVOTool getterFromSetter:setterName];
    

    if (!getterName) {
        SJLog(@"%@",[SJKVOError errorTransferSetterToGetterFaildedWithSetterName:setterName]);
        return;
    }
    
    // create a super class of a specific instance
    Class superclass = class_getSuperclass(OriginalClass);
    
    struct objc_super superclass_to_call = {
        .super_class = superclass,  //super class
        .receiver = self,           //insatance of this class
    };
    
    // cast method pointer
    void (*objc_msgSendSuperCasted)(void *, SEL, id) = (void *)objc_msgSendSuper;
    
    // call super's setter, the supper is the original class
    objc_msgSendSuperCasted(&superclass_to_call, _cmd, newValue);
    
    // look up observers and call the blocks
    NSMutableSet *observers = objc_getAssociatedObject(self,&SJKVOObservers);
    
    if (observers.count <= 0) {
        SJLog(@"%@",[SJKVOError errorNoObserverOfObject:self]);
        return;
    }
    
    //get the old value
    id oldValue = [self valueForKey:getterName];
    
    for (SJKVOObserverItem *item in observers) {
        if ([item.key isEqualToString:getterName]) {
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
                //call block
                item.block(self, getterName, oldValue, newValue);
            });
        }
    }
}
```


然后实例化对应的观察项：
```objc
- (void)addObserverItem:(NSObject *)observer
                    key:(NSString *)key
         setterSelector:(SEL)setterSelector
           setterMethod:(Method)setterMethod
                  block:(SJKVOBlock)block
{
    
    NSMutableSet *observers = objc_getAssociatedObject(self, &SJKVOObservers);
    if (!observers) {
        observers = [[NSMutableSet alloc] initWithCapacity:10];
        objc_setAssociatedObject(self, &SJKVOObservers, observers, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    
    SJKVOObserverItem *item = [[SJKVOObserverItem alloc] initWithObserver:observer Key:key setterSelector:setterSelector setterMethod:setterMethod block:block];
    
    if (item) {
        [observers addObject:item];
    }
    
}
```

#### 5. 判断新的观察是否会与已经保存的观察项重复（当观察对象和key一致的时候），如果重复，则不添加新的观察：
```objc
    / /ignore same observer and key:if the observer and key are same with saved observerItem,we should not add them one more time
    BOOL findSameObserverAndKey = NO;
    if (observers.count>0) {
        for (SJKVOObserverItem *item in observers) {
            if ( (item.observer == observer) && [item.key isEqualToString:key]) {
                findSameObserverAndKey = YES;
            }
        }
    }
    
    if (!findSameObserverAndKey) {
        [self KVOConfigurationWithObserver:observer key:key block:block kvoClass:KVOClass setterSelector:setterSelector setterMethod:setterMethod];
    }
```

而一次性添加多个key的方法，也只是调用多次一次性添加单个key的方法罢了：
```objc
- (void)sj_addObserver:(NSObject *)observer
               forKeys:(NSArray <NSString *>*)keys
             withBlock:(SJKVOBlock)block
{
    //error: keys array is nil or no elements
    if (keys.count == 0) {
        SJLog(@"%@",[SJKVOError errorInvalidInputObservingKeys]);
        return;
    }
    
    //one key corresponding to one specific item, not the observer
    [keys enumerateObjectsUsingBlock:^(NSString * key, NSUInteger idx, BOOL * _Nonnull stop) {
        [self sj_addObserver:observer forKey:key withBlock:block];
    }];
}
```

关于移除观察的实现，只是在观察项set里面找出封装了对应的观察对象和key的观察项就可以了：
```objc
- (void)sj_removeObserver:(NSObject *)observer
                   forKey:(NSString *)key
{
    NSMutableSet* observers = objc_getAssociatedObject(self, &SJKVOObservers);
    
    if (observers.count > 0) {
        
        SJKVOObserverItem *removingItem = nil;
        for (SJKVOObserverItem* item in observers) {
            if (item.observer == observer && [item.key isEqualToString:key]) {
                removingItem = item;
                break;
            }
        }
        if (removingItem) {
            [observers removeObject:removingItem];
        }
        
    }
}
```

再看一下移除所有观察者：
```objc
- (void)sj_removeAllObservers
{
    NSMutableSet* observers = objc_getAssociatedObject(self, &SJKVOObservers);
    
    if (observers.count > 0) {
        [observers removeAllObjects];
        SJLog(@"SJKVOLog:Removed all obserbing objects of object:%@",self);
        
    }else{
        SJLog(@"SJKVOLog:There is no observers obserbing object:%@",self);
    }
}
```

## SJKVOObserverItem

这个类负责封装每一个观察项的信息，包括：
- 观察者对象。
- 被观察的key。
- setter方法名（SEL）
- setter方法（Method）
- 回调的block

>需要注意的是:
>在这个小轮子里，对于同一个对象可以观察不同的key的情况，是将这两个key区分开来的，是属于不同的观察项。所以应该用不同的``SJKVOObserverItem``实例来封装。

```objc
#import <Foundation/Foundation.h>
#import <objc/runtime.h>

typedef void(^SJKVOBlock)(id observedObject, NSString *key, id oldValue, id newValue);

@interface SJKVOObserverItem : NSObject

@property (nonatomic, strong) NSObject *observer;
@property (nonatomic, copy)   NSString *key;
@property (nonatomic, assign) SEL setterSelector;
@property (nonatomic, assign) Method setterMethod;
@property (nonatomic, copy)   SJKVOBlock block;

- (instancetype)initWithObserver:(NSObject *)observer Key:(NSString *)key setterSelector:(SEL)setterSelector setterMethod:(Method)setterMethod block:(SJKVOBlock)block;

@end

```
 
## SJKVOTool

这个类负责setter方法与getter方法相互转换，以及和运行时相关的操作，服务于``SJKVOController``。看一下它的头文件：
```objc
#import <Foundation/Foundation.h>
#import <objc/runtime.h>
#import <objc/message.h>

@interface SJKVOTool : NSObject

//setter <-> getter
+ (NSString *)getterFromSetter:(NSString *)setter;
+ (NSString *)setterFromGetter:(NSString *)getter;

//get method from a class by a specific selector
+ (Method)objc_methodFromClass:(Class)cls selector:(SEL)selector;

//check a class has a specific selector or not
+ (BOOL)detectClass:(Class)cls hasSelector:(SEL)selector;

@end
```




##SJKVOError

这个小轮子仿照了[JSONModel](https://github.com/jsonmodel/jsonmodel)的错误管理方式，用单独的一个类``SJKVOError``来返回各种错误：

```objc
#import <Foundation/Foundation.h>

typedef enum : NSUInteger {
    
    SJKVOErrorTypeNoObervingObject,
    SJKVOErrorTypeNoObervingKey,
    SJKVOErrorTypeNoObserverOfObject,
    SJKVOErrorTypeNoMatchingSetterForKey,
    SJKVOErrorTypeTransferSetterToGetterFailded,
    SJKVOErrorTypeInvalidInputObservingKeys,
    
} SJKVOErrorTypes;

@interface SJKVOError : NSError

+ (id)errorNoObervingObject;
+ (id)errorNoObervingKey;
+ (id)errorNoMatchingSetterForKey:(NSString *)key;
+ (id)errorTransferSetterToGetterFaildedWithSetterName:(NSString *)setterName;
+ (id)errorNoObserverOfObject:(id)object;
+ (id)errorInvalidInputObservingKeys;

@end
```

OK，这样就介绍完了，希望各位同学可以积极指正～



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



