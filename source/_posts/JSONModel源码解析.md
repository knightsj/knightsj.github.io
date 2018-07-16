---
title: JSONModel源码解析
tags: [iOS,Objective-C,源码解析]
categories: iOS
---

这一系列的[源码解析](http://www.jianshu.com/nb/9137726)分享到现在已经是第五篇了。这五篇讲解的都是view层的一些流行的iOS开源框架。而从本篇开始开始，我打算要逐渐加深难度，讲解一些model层和网络层相关的开源框架。

想来想去，还是从[JSONModel](https://github.com/jsonmodel/jsonmodel)开始吧～

首先因为该框架还是比较流行的，在GitHub上也有将近6000颗星了，而且我自己对这个框架的使用也比较熟悉。还有一点是这个框架运用了运行时的相关知识，对想要了解运行时的童鞋还是很有帮助的。

该框架的核心代码并不是很多，主要还是错误类型判断和容错处理占了不少内容。读过一遍之后，感觉到作者思维的严谨性是非常值得我们学习的：作者专门建立了一个展示错误(NSError)的类，里面封装了很多错误类型，而且这个框架还允许用户根据自己的需求来自定义错误类型并阻止最终模型的生成，在后文会有详细讲解。

<!-- more -->

在讲解源码之前，有必要先给不会使用JSONModel的同学们通过实际的例子来介绍一下它的使用方法（而且后面的源码解析部分也是结合这些例子给出的，因为结合例子有助于加快理解）：

# 使用方法

## 1. 最基本的使用

第一种就是单纯地传入一个字典，并转换成模型：
首先我们需要定义我们自己的模型类：
```objc
@interface Person : JSONModel
@property (nonatomic, copy)   NSString *name;
@property (nonatomic, copy)   NSString *sex;
@property (nonatomic, assign) NSInteger age;
@end
```

然后再使用字典来转换为模型：
```objc
NSDictionary *dict = @{
                        @"name":@"Jack",
                        @"age":@23,
                        @"gender":@"male",
                      };
 NSError *error;
 Person *person = [[Person alloc] initWithDictionary:dict error:&error];
```

输出：
```objc
 <Person> 
   [name]: Jack
   [sex]: male
   [gender]: 23
</Person>
```

可以看出来，该框架的使用非常方便，一行代码就将模型转换好了。
但是该框架的功能远不止这些：

## 2. 转换属性名称

有的时候，传入的字典里的key发生了变化（比如接口重构之类的原因），但是我们前端这边已经写好的模型属性可能不容易被修改（因为业务逻辑很复杂什么的），所以这个时候，最好有一个转化的功能。

在这里举个例子：原来字典里的``gender``这个key变成了``sex``，这就需要我们定义一个转换的mapper（``JSONKeyMapper``）：
```objc
@implementation Person
+ (JSONKeyMapper *)keyMapper
{
    return [[JSONKeyMapper alloc] initWithModelToJSONDictionary:@{
                                                                  @"gender": @"sex",                                                             }];
}
```

这样一来，``JSONKeyMapper``就会自动帮我们做转换。
为了验证效果，我们修改一下传入的字典里的``gender``字段为``sex``：
```objc
NSDictionary *dict = @{
                        @"name":@"Jack",
                        @"age":@23,
                        @"sex":@"male",
                      };
NSError *error;
Person *person = [[Person alloc] initWithDictionary:dict error:&error];
```
再看一下输出：

```objc
<Person> 
   [name]: Jack
   [age]: 23
   [gender]: male
</Person>
```

没有受到传入字典里key值的变化的影响，是吧？

## 3. 自定义错误

除了一些框架里自己处理的错误（比如传入的对象不是字典等），框架的作者也允许我们自己定义属于我们自己的错误。

比如，当``age``对应的数值小于25的时候，打印出``Too young!``,并阻止模型的转换：

首先，我们在模型的实现文件里添加：

```objc
- (BOOL)validate:(NSError **)error
{
    if (![super validate:error])
        return NO;
    
    if (self.age < 25)
    {
        *error = [NSError errorWithDomain:@"Too young!" code:10 userInfo:nil];
        NSError *errorLog = *error;
        NSLog(@"%@",errorLog.domain);
        return NO;
    }
    
    return YES;
}
```

看一下结果：
```objc
2017-02-20 16:14:54.217 jsonmodel_demo[32942:1967433] Too young!
2017-02-20 16:14:54.217 jsonmodel_demo[32942:1967433] (null)
```

打印了错误，而且模型也没有被转换。

## 4. 模型嵌套

有的时候，我们需要在模型里加一个数组，而这个数组里面的元素是另一个对象：这就涉及到了模型的嵌套。

举个例子，我们让上面的``Person``对象含有一个数组``Friends``，它里面的元素是对象``Friend``，也就是好友信息。若要实现模型的嵌套，我们只需在原来的模型类里增加一个协议``Friend``：

```objc
#import "JSONModel.h"
@protocol Friend;
@interface Friend : JSONModel
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@end
@interface Person : JSONModel
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, copy) NSString *gender;
@property (nonatomic, strong) NSArray<Friend> *friends;//数组，嵌套模型
@end
```

而且要在``Person``的实现文件里加上这一段代码：
```objc
@implementation Friend
@end
```
注意！如果不添加，则会令程序崩溃。

最后，在使用的时候，我们只需将持有一个数组的字典里传入即可：

```objc
NSArray *array = @[
                      @{
                        @"name":@"Peter",
                        @"age":@35,
                        },
                   ];
    
 NSDictionary *dict = @{
                           @"name":@"Jack",
                           @"age":@23,
                           @"sex":@"male",
                           @"friends":array,//朋友列表（模型嵌套）
                         };
NSError *error;
Person *person = [[Person alloc] initWithDictionary:dict error:&error];
NSLog(@"%@",person);
```

输出结果：
```objc
<Person> 
   [age]: 23
   [gender]: male
   [friends]: (
       "<Friend> \n   [name]: Peter\n   [age]: 35\n</Friend>"
   )
   [name]: Jack
</Person>
```
我们可以看到，person对象里含有一个数组，这个数组只有一个元素，对应着上面字典里的array里的信息。

OK，这样一来，大家已经可以掌握该框架的主要用法了，现在开始详细讲解代码：



# 源码解析

本篇源码解析主要围绕着``initWithDictionary:error:``来展开，在这一个方法里作者做到了所有的容错和模型的转化。

按照老规矩，先上流程图：

![字典->模型](http://upload-images.jianshu.io/upload_images/859001-94b356eeb7b560e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


该流程图对应的方法实现是：

```objc
-(id)initWithDictionary:(NSDictionary*)dict error:(NSError**)err
{
    //方法1. 参数为nil
    if (!dict) {
        if (err) *err = [JSONModelError errorInputIsNil];
        return nil;
    }
    //方法2. 参数不是nil，但也不是字典
    if (![dict isKindOfClass:[NSDictionary class]]) {
        if (err) *err = [JSONModelError errorInvalidDataWithMessage:@"Attempt to initialize JSONModel object using initWithDictionary:error: but the dictionary parameter was not an 'NSDictionary'."];
        return nil;
    }
    //方法3. 初始化
    self = [self init];
    if (!self) {
        //初始化失败
        if (err) *err = [JSONModelError errorModelIsInvalid];
        return nil;
    }
    //方法4. 检查用户定义的模型里的属性集合是否大于传入的字典里的key集合（如果大于，则返回NO）
    if (![self __doesDictionary:dict matchModelWithKeyMapper:self.__keyMapper error:err]) {
        return nil;
    }
    //方法5. 核心方法：字典的key与模型的属性的映射
    if (![self __importDictionary:dict withKeyMapper:self.__keyMapper validation:YES error:err]) {
        return nil;
    }
    //方法6. 可以重写[self validate:err]方法并返回NO，让用户自定义错误并阻拦model的返回
    if (![self validate:err]) {
        return nil;
    }
    //方法7. 终于通过了！成功返回model
    return self;
}
```

其中，
- 方法1-4:都是对错误的发现与处理。
- 方法5:是真正的mapping。
- 方法6:是作者给用户自己定义错误的方法，如果复合了用户自己定义的错误，那么即使mapping成功了，也要返回nil。
  -方法7:成功返回模型对象。

在讲解代码之前，有必要先了解一下JSONModel所持有的一些数据：

- **关联对象kClassPropertiesKey**:(用来保存所有属性信息的NSDictionary)
```objc
{
       age = "@property primitive age (Setters = [])";
      name = "@property NSString* name (Standard JSON type, Setters = [])";
    gender = "@property NSString* gender (Standard JSON type, Setters = [])";
}
```

- **关联对象kClassRequiredPropertyNamesKey**：(用来保存所有属性的名称NSSet)
```objc
{(
    name,
    age,
    gender
)}
```

- **关联对象kMapperObjectKey**：(用来保存JSONKeyMapper)：自定义的mapper，具体的使用方法在上面的例子中可以看到。


- **JSONModelClassProperty**：封装的jsonmodel的一个属性，它包含了对应属性的名字（name：gender），类型（type：NSString），是否是JSONModel支持的类型（isStandardJSONType：YES/NO），是否是可变对象（isMutable:YES/NO）等属性。

再大致讲解一下整个的流程：
首先，在这个模型类的对象被初始化的时候，遍历自身到所有的父类（直到JSONModel为止），获取所有的属性，并将其保存在一个字典里。获取传入字典的所有key，将这些key与保存的所有属性进行匹配。如果匹配成功，则进行kvc赋值。


OK，现在从上到下逐步讲解上段代码：

首先，在``load``方法里，定义了该框架支持的类型：
```objc
+(void)load
{
    static dispatch_once_t once;
    dispatch_once(&once, ^{
        @autoreleasepool {           
            //兼容的对象属性
            allowedJSONTypes = @[
                [NSString class], [NSNumber class], [NSDecimalNumber class], [NSArray class], [NSDictionary class], [NSNull class], //immutable JSON classes
                [NSMutableString class], [NSMutableArray class], [NSMutableDictionary class] //mutable JSON classes
            ];
            //兼容的基本类型属性
            allowedPrimitiveTypes = @[
                @"BOOL", @"float", @"int", @"long", @"double", @"short",
                //and some famous aliases
                @"NSInteger", @"NSUInteger",
                @"Block"
            ];
            //转换器
            valueTransformer = [[JSONValueTransformer alloc] init];
            
            //自己的类型
            JSONModelClass = NSClassFromString(NSStringFromClass(self));
        }
    });
}
```

然后我们看一下从方法3的init方法开始，作者都做了什么：

```objc
-(id)init
{
    self = [super init];
    if (self) {
        [self __setup__];
    }
    return self;
}
-(void)__setup__
{
    //只有第一次实例化时，才执行
    if (!objc_getAssociatedObject(self.class, &kClassPropertiesKey)) {
        [self __inspectProperties];
    }
    
    //如果存在自定义的mapper，则将它保存在关联对象里面，key是kMapperObjectKey
    id mapper = [[self class] keyMapper];
    if ( mapper && !objc_getAssociatedObject(self.class, &kMapperObjectKey) ) {
        objc_setAssociatedObject(
                                 self.class,
                                 &kMapperObjectKey,
                                 mapper,
                                 OBJC_ASSOCIATION_RETAIN // This is atomic
                                 );
    }
}
```

值得注意的是，这里的``__inspectProperties:``方法是该框架的核心方法之一：它的任务是保存了所有需要赋值的属性。用作在将来与传进来字典进行映射：

```objc
-(void)__inspectProperties
{
//    最终保存所有属性的字典，形式为：
//    {
//        age = "@property primitive age (Setters = [])";
//        friends = "@property NSArray*<Friend> friends (Standard JSON type, Setters = [])";
//        gender = "@property NSString* gender (Standard JSON type, Setters = [])";
//        name = "@property NSString* name (Standard JSON type, Setters = [])";
//    }
    NSMutableDictionary* propertyIndex = [NSMutableDictionary dictionary];
    //获取当前的类名
    Class class = [self class];    
    NSScanner* scanner = nil;
    NSString* propertyType = nil;
    // 循环条件：当class 是 JSONModel自己的时候终止
    while (class != [JSONModel class]) {        
        //属性的个数
        unsigned int propertyCount;
        //获得属性列表（所有@property声明的属性）
        objc_property_t *properties = class_copyPropertyList(class, &propertyCount);
        //遍历所有的属性
        for (unsigned int i = 0; i < propertyCount; i++) {
            //获得属性名称
            objc_property_t property = properties[i];//获得当前的属性
            const char *propertyName = property_getName(property);//name（C字符串）            
            //JSONModel里的每一个属性，都被封装成一个JSONModelClassProperty对象
            JSONModelClassProperty* p = [[JSONModelClassProperty alloc] init];
            p.name = @(propertyName);//propertyName:属性名称，例如：name，age，gender
            //获得属性类型
            const char *attrs = property_getAttributes(property);
            NSString* propertyAttributes = @(attrs);
            // T@\"NSString\",C,N,V_name
            // Tq,N,V_age
            // T@\"NSString\",C,N,V_gender
            // T@"NSArray<Friend>",&,N,V_friends            
            NSArray* attributeItems = [propertyAttributes componentsSeparatedByString:@","];
            //说明是只读属性，不做任何操作
            if ([attributeItems containsObject:@"R"]) {
                continue; //to next property
            }
            //检查出是布尔值
            if ([propertyAttributes hasPrefix:@"Tc,"]) {
                p.structName = @"BOOL";//使其变为结构体
            }            
            //实例化一个scanner
            scanner = [NSScanner scannerWithString: propertyAttributes];
            [scanner scanUpToString:@"T" intoString: nil];
            [scanner scanString:@"T" intoString:nil];
            //http://blog.csdn.net/kmyhy/article/details/8258858           
            if ([scanner scanString:@"@\"" intoString: &propertyType]) {                
                 //属性是一个对象
                [scanner scanUpToCharactersFromSet:[NSCharacterSet characterSetWithCharactersInString:@"\"<"]
                                        intoString:&propertyType];//propertyType -> NSString                
                p.type = NSClassFromString(propertyType);// p.type = @"NSString"
                p.isMutable = ([propertyType rangeOfString:@"Mutable"].location != NSNotFound); //判断是否是可变的对象
                p.isStandardJSONType = [allowedJSONTypes containsObject:p.type];//是否是该框架兼容的类型
                //存在协议(数组，也就是嵌套模型)
                while ([scanner scanString:@"<" intoString:NULL]) {
                    NSString* protocolName = nil;
                    [scanner scanUpToString:@">" intoString: &protocolName];
                    if ([protocolName isEqualToString:@"Optional"]) {
                        p.isOptional = YES;
                    } else if([protocolName isEqualToString:@"Index"]) {
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wdeprecated-declarations"
                        p.isIndex = YES;
#pragma GCC diagnostic pop
                        objc_setAssociatedObject(
                                                 self.class,
                                                 &kIndexPropertyNameKey,
                                                 p.name,
                                                 OBJC_ASSOCIATION_RETAIN // This is atomic
                                                 );
                    } else if([protocolName isEqualToString:@"Ignore"]) {
                        p = nil;
                    } else {
                        p.protocol = protocolName;
                    }
                    //到最接近的>为止
                    [scanner scanString:@">" intoString:NULL];
                }
            }            
            else if ([scanner scanString:@"{" intoString: &propertyType])                
                //属性是结构体
                [scanner scanCharactersFromSet:[NSCharacterSet alphanumericCharacterSet]
                                    intoString:&propertyType];
                p.isStandardJSONType = NO;
                p.structName = propertyType;
            }
            else {
                //属性是基本类型：Tq,N,V_age
                [scanner scanUpToCharactersFromSet:[NSCharacterSet characterSetWithCharactersInString:@","]
                                        intoString:&propertyType];
                //propertyType:q
                propertyType = valueTransformer.primitivesNames[propertyType];              
                //propertyType:long
                //基本类型数组
                if (![allowedPrimitiveTypes containsObject:propertyType]) {
                    //类型不支持
                    @throw [NSException exceptionWithName:@"JSONModelProperty type not allowed"
                                                   reason:[NSString stringWithFormat:@"Property type of %@.%@ is not supported by JSONModel.", self.class, p.name]
                                                 userInfo:nil];
                }
            }
            NSString *nsPropertyName = @(propertyName);            
            //可选的
            if([[self class] propertyIsOptional:nsPropertyName]){
                p.isOptional = YES;
            }
            //可忽略的
            if([[self class] propertyIsIgnored:nsPropertyName]){
                p = nil;
            }
            //集合类
            Class customClass = [[self class] classForCollectionProperty:nsPropertyName];            
            if (customClass) {
                p.protocol = NSStringFromClass(customClass);
            }
            //忽略block
            if ([propertyType isEqualToString:@"Block"]) {
                p = nil;
            }
            //如果字典里不存在，则添加到属性字典里（终于添加上去了。。。）
            if (p && ![propertyIndex objectForKey:p.name]) {
                [propertyIndex setValue:p forKey:p.name];
            }
            //setter 和 getter
            if (p)
            {   //name ->Name
                NSString *name = [p.name stringByReplacingCharactersInRange:NSMakeRange(0, 1) withString:[p.name substringToIndex:1].uppercaseString];
                // getter
                SEL getter = NSSelectorFromString([NSString stringWithFormat:@"JSONObjectFor%@", name]);
                if ([self respondsToSelector:getter])
                    p.customGetter = getter;
                // setters
                p.customSetters = [NSMutableDictionary new];
                SEL genericSetter = NSSelectorFromString([NSString stringWithFormat:@"set%@WithJSONObject:", name]);
                if ([self respondsToSelector:genericSetter])
                    p.customSetters[@"generic"] = [NSValue valueWithBytes:&genericSetter objCType:@encode(SEL)];
                for (Class type in allowedJSONTypes)
                {
                    NSString *class = NSStringFromClass([JSONValueTransformer classByResolvingClusterClasses:type]);
                    if (p.customSetters[class])
                        continue;
                    SEL setter = NSSelectorFromString([NSString stringWithFormat:@"set%@With%@:", name, class]);

                    if ([self respondsToSelector:setter])
                        p.customSetters[class] = [NSValue valueWithBytes:&setter objCType:@encode(SEL)];
                }
            }
        }
        free(properties);
        //再指向自己的父类，知道等于JSONModel才停止
        class = [class superclass];
    }
    //最后保存所有当前类，JSONModel的所有的父类的属性
    objc_setAssociatedObject(
                             self.class,
                             &kClassPropertiesKey,
                             [propertyIndex copy],
                             OBJC_ASSOCIATION_RETAIN
                             );
}
```

>需要注意几点：
1. 作者利用一个``while``函数，获取当前类和当前类的除JSONModel的所有父类的属性保存在一个字典中。在将来用于和传入的字典进行映射。
2. 作者用``JSONModelClassProperty``类封装了JSONModel的每一个属性。这个类有两个重要的属性：一个是``name``，它是属性的名称(例如gender)。另一个是``type``，它是属性的类型（例如NSString）。
3. 作者将属性分为了如下几个类型：
    1. 对象（不含有协议）。
    2. 对象（含有协议，属于模型嵌套）。
    3. 基本数据类型。
    4. 结构体。


我们来看一下方法4的实现：

```objc
-(BOOL)__doesDictionary:(NSDictionary*)dict matchModelWithKeyMapper:(JSONKeyMapper*)keyMapper error:(NSError**)err
{
    //拿到字典里所有的key
    NSArray* incomingKeysArray = [dict allKeys];    
    //返回保存所有属性名称的数组(name,age,gender...)
    NSMutableSet* requiredProperties = [self __requiredPropertyNames].mutableCopy;    
    //从array拿到set
    NSSet* incomingKeys = [NSSet setWithArray: incomingKeysArray];
    //如果用户自定义了mapper，则进行转换
    if (keyMapper || globalKeyMapper) {
        NSMutableSet* transformedIncomingKeys = [NSMutableSet setWithCapacity: requiredProperties.count];
        NSString* transformedName = nil;
        //便利需要转换的属性列表
        for (JSONModelClassProperty* property in [self __properties__]) {
            //被转换成的属性名称 gender（模型内） -> sex（字典内）
            transformedName = (keyMapper||globalKeyMapper) ? [self __mapString:property.name withKeyMapper:keyMapper] : property.name;
            //拿到sex以后，查看传入的字典里是否有sex对应的值
            id value;
            @try {
                value = [dict valueForKeyPath:transformedName];
            }
            @catch (NSException *exception) {
                value = dict[transformedName];
            }
            //如果值存在，则将sex添加到传入的keys数组中
            if (value) {
                [transformedIncomingKeys addObject: property.name];
            }
        }
        incomingKeys = transformedIncomingKeys;
    }
    //查看当前的model的属性的集合是否大于传入的属性集合，如果是，则返回错误。
    //也就是说模型类里的属性是不能多于传入字典里的key的，例如：
    if (![requiredProperties isSubsetOfSet:incomingKeys]) {
        //获取多出来的属性
        [requiredProperties minusSet:incomingKeys];
        //not all required properties are in - invalid input
        JMLog(@"Incoming data was invalid [%@ initWithDictionary:]. Keys missing: %@", self.class, requiredProperties);
        if (err) *err = [JSONModelError errorInvalidDataWithMissingKeys:requiredProperties];
        return NO;
    }
    //不需要了，释放掉
    incomingKeys= nil;
    requiredProperties= nil;
    return YES;
}
```

>这里需要需要注意的：
1. model类里面定义的属性集合是不能大于传入的字典里的key集合的。
2. 如果存在了用户自定义的mapper，则需要按照用户的定义来进行转换。
  （在这里是奖gender转换为了sex）。


最后来看一下本框架第二个核心代码(上面的方法5)，也就是真正从字典里获取值并赋给当前模型对象的实现：

```objc
-(BOOL)__importDictionary:(NSDictionary*)dict withKeyMapper:(JSONKeyMapper*)keyMapper validation:(BOOL)validation error:(NSError**)err
{
    //遍历保存的所有属性的字典
    for (JSONModelClassProperty* property in [self __properties__]) {
        //将属性的名称拿过来，作为key，用这个key来查找传进来的字典里对应的值
        NSString* jsonKeyPath = (keyMapper||globalKeyMapper) ? [self __mapString:property.name withKeyMapper:keyMapper] : property.name;
        //用来保存从字典里获取的值
        id jsonValue;        
        @try {
            jsonValue = [dict valueForKeyPath: jsonKeyPath];
        }
        @catch (NSException *exception) {
            jsonValue = dict[jsonKeyPath];
        }
        //字典不存在对应的key
        if (isNull(jsonValue)) {
            //如果这个key是可以不存在的
            if (property.isOptional || !validation) continue;            
            //如果这个key是必须有的，则返回错误
            if (err) {
                NSString* msg = [NSString stringWithFormat:@"Value of required model key %@ is null", property.name];
                JSONModelError* dataErr = [JSONModelError errorInvalidDataWithMessage:msg];
                *err = [dataErr errorByPrependingKeyPathComponent:property.name];
            }
            return NO;
        }        
        //获取 取到的值的类型
        Class jsonValueClass = [jsonValue class];
        BOOL isValueOfAllowedType = NO;
        //查看是否是本框架兼容的属性类型
        for (Class allowedType in allowedJSONTypes) {
            if ( [jsonValueClass isSubclassOfClass: allowedType] ) {
                isValueOfAllowedType = YES;
                break;
            }
        }        
        //如果不兼容，则返回NO，mapping失败
        if (isValueOfAllowedType==NO) {
            //type not allowed
            JMLog(@"Type %@ is not allowed in JSON.", NSStringFromClass(jsonValueClass));
            if (err) {
                NSString* msg = [NSString stringWithFormat:@"Type %@ is not allowed in JSON.", NSStringFromClass(jsonValueClass)];
                JSONModelError* dataErr = [JSONModelError errorInvalidDataWithMessage:msg];
                *err = [dataErr errorByPrependingKeyPathComponent:property.name];
            }
            return NO;
        }
        //如果是兼容的类型：
        if (property) {
            // 查看是否有自定义setter，并设置
            if ([self __customSetValue:jsonValue forProperty:property]) {
                continue;
            };
            // 基本类型
            if (property.type == nil && property.structName==nil) {
                //kvc赋值
                if (jsonValue != [self valueForKey:property.name]) {
                    [self setValue:jsonValue forKey: property.name];
                }
                continue;
            }
            // 如果传来的值是空，即使当前的属性对应的值不是空，也要将空值赋给它
            if (isNull(jsonValue)) {
                if ([self valueForKey:property.name] != nil) {
                    [self setValue:nil forKey: property.name];
                }
                continue;
            }
            // 1. 属性本身是否是jsonmodel类型
            if ([self __isJSONModelSubClass:property.type]) {
                //通过自身的转模型方法，获取对应的值
                JSONModelError* initErr = nil;
                id value = [[property.type alloc] initWithDictionary: jsonValue error:&initErr];
                if (!value) {               
                    //如果该属性不是必须的，则略过
                    if (property.isOptional || !validation) continue;
                    //如果该属性是必须的，则返回错误
                    if((err != nil) && (initErr != nil))
                    {
                        *err = [initErr errorByPrependingKeyPathComponent:property.name];
                    }
                    return NO;
                }            
                //当前的属性值为空，则赋值
                if (![value isEqual:[self valueForKey:property.name]]) {
                    [self setValue:value forKey: property.name];
                }
                continue;
            } else {
                // 如果不是jsonmodel的类型，则可能是一些普通的类型：NSArray，NSString。。。
                // 是否是模型嵌套（带有协议）
                if (property.protocol) {
                    //转化为数组，这个数组就是例子中的friends属性。
                    jsonValue = [self __transform:jsonValue forProperty:property error:err];
                   
                    if (!jsonValue) {
                        if ((err != nil) && (*err == nil)) {
                            NSString* msg = [NSString stringWithFormat:@"Failed to transform value, but no error was set during transformation. (%@)", property];
                            JSONModelError* dataErr = [JSONModelError errorInvalidDataWithMessage:msg];
                            *err = [dataErr errorByPrependingKeyPathComponent:property.name];
                        }
                        return NO;
                    }
                }
                // 对象类型
                if (property.isStandardJSONType && [jsonValue isKindOfClass: property.type]) {
                    //可变类型
                    if (property.isMutable) {
                        jsonValue = [jsonValue mutableCopy];
                    }
                    //赋值
                    if (![jsonValue isEqual:[self valueForKey:property.name]]) {
                        [self setValue:jsonValue forKey: property.name];
                    }
                    continue;
                }
                // 当前的值的类型与对应的属性的类型不一样的时候，需要查看用户是否自定义了转换器（例如从NSSet到NSArray转换：- (NSSet *)NSSetFromNSArray:(NSArray *)array）
                if (
                    (![jsonValue isKindOfClass:property.type] && !isNull(jsonValue))
                    ||
                    //the property is mutable
                    property.isMutable
                    ||
                    //custom struct property
                    property.structName
                    ) {
                    // searched around the web how to do this better
                    // but did not find any solution, maybe that's the best idea? (hardly)
                    Class sourceClass = [JSONValueTransformer classByResolvingClusterClasses:[jsonValue class]];
                    //JMLog(@"to type: [%@] from type: [%@] transformer: [%@]", p.type, sourceClass, selectorName);

                    //build a method selector for the property and json object classes
                    NSString* selectorName = [NSString stringWithFormat:@"%@From%@:",
                                              (property.structName? property.structName : property.type), //target name
                                              sourceClass]; //source name
                    SEL selector = NSSelectorFromString(selectorName);
                    //查看自定义的转换器是否存在
                    BOOL foundCustomTransformer = NO;
                    if ([valueTransformer respondsToSelector:selector]) {
                        foundCustomTransformer = YES;                        
                    } else {
                        //try for hidden custom transformer
                        selectorName = [NSString stringWithFormat:@"__%@",selectorName];
                        selector = NSSelectorFromString(selectorName);
                        if ([valueTransformer respondsToSelector:selector]) {
                            foundCustomTransformer = YES;
                        }
                    }
                    //如果存在自定义转换器，则进行转换
                    if (foundCustomTransformer) {                        
                        IMP imp = [valueTransformer methodForSelector:selector];
                        id (*func)(id, SEL, id) = (void *)imp;
                        jsonValue = func(valueTransformer, selector, jsonValue);

                        if (![jsonValue isEqual:[self valueForKey:property.name]])
                            [self setValue:jsonValue forKey:property.name];                        
                    } else {                       
                        //没有自定义转换器，返回错误
                        NSString* msg = [NSString stringWithFormat:@"%@ type not supported for %@.%@", property.type, [self class], property.name];
                        JSONModelError* dataErr = [JSONModelError errorInvalidDataWithTypeMismatch:msg];
                        *err = [dataErr errorByPrependingKeyPathComponent:property.name];
                        return NO;                        
                    }
                } else {
                    // 3.4) handle "all other" cases (if any)
                    if (![jsonValue isEqual:[self valueForKey:property.name]])
                        [self setValue:jsonValue forKey:property.name];
                }
            }
        }
    }
    return YES;
}
```


>值得注意的是：
- 作者在最后给属性赋值的时候使用的是kvc的``setValue:ForKey:``的方法。
- 作者判断了模型里的属性的类型是否是JSONModel的子类，可见作者的考虑是非常周全的。
- 整个框架看下来，有很多的地方涉及到了错误判断，作者将将错误类型单独抽出一个类（``JSONModelError``），里面支持的错误类型很多，可以侧面反应作者思维之缜密。而且这个做法也可以在我们写自己的框架或者项目中使用。

错误判断的一个例子：
```objc
//JSONModelError.m
+(id)errorInvalidDataWithMessage:(NSString*)message
{
    message = [NSString stringWithFormat:@"Invalid JSON data: %@", message];
    return [JSONModelError errorWithDomain:JSONModelErrorDomain
                                      code:kJSONModelErrorInvalidData
                                  userInfo:@{NSLocalizedDescriptionKey:message}];
}
```

夸了作者这么多，唯一我个人不太喜欢的地方就是if语句下只有一行的时候，作者不喜欢加上大括号：
```objc
if (![jsonValue isEqual:[self valueForKey:property.name]])
    [self setValue:jsonValue forKey:property.name];
```

但是我觉得应该加的：

```objc
if (![jsonValue isEqual:[self valueForKey:property.name]]){
  [self setValue:jsonValue forKey:property.name];
}
```



## 知识扩展

- 作者用NSScanner来扫描字符串，将从类结构体里拿过来的属性的描述字符串``T@\"NSString\",C,N,V_name``中扫描出了类型：``NSString``。
- 作者两次用到了NSSet：当集合里的元素顺序不重要的时候，优先考虑用NSSet。


总的来说这个框架的难度还是不大的，但可能因为是第一次阅读不涉及UIVIiew的框架，感觉有些枯燥，不过慢慢习惯就好啦～





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



