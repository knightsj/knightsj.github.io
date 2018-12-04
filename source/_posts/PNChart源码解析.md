---
title: PNChart源码解析
tags: [iOS,Objective-C,源码解析]
categories: iOS
---

## 一. 框架介绍

[PNChart](https://github.com/kevinzhow/PNChart)是国内开发者开发的iOS图表框架，现在已经7900多颗star了。它涵盖了折线图，柱状图，饼图，散点图等图表。图表的可定制性很高，而且UI设计简洁大方。

该框架分为两层：视图层和数据层。视图层里有两层继承关系，第一层是所有类型图表的父类``PNGenericChart``,第二层就是所有类型的图表。提供一张图来直观感受一下：

![层级图](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/source_code_analysis/pnchart_header.png)

>在这张图里，需要注意以下几点：
1. 带箭头的线和不带箭头的线的区别。
2. ``Data``类对应图表的一组数据，因为当前类型的图表支持多组数据（例如：饼状图没有``Data``类，因为饼状图没有多组数据，而折线图``LineChart``是支持多组数据的，所以有``Data``类。
3. ``Item``类负责将传入图表的某个真实值转化为图表中显示的值，具体做法会在下文详细讲解。
4. ``BarChart``类里面的每一根柱子都是``PNBar``的实例（该类型的图表不在本篇讲解的范围之内）。

<!-- more -->

今天就来介绍一下该框架里的折线图的源码。上文提到过，该框架的折线图是支持多组数据的，也就是在同一张图表上显示多条折线。先带大家看一下效果图：

![折线图](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/source_code_analysis/pnchart_chart.png)


折线图在效果上还是很简洁美观的，如果现在的你还不知道如何使用``CAShapeLayer``和``UIBezierPath``画图并附加动画效果，那么本篇源码解析非常适合你。

阅读本文之后，你可以掌握有关图形绘制的相关知识，也可以掌握自定义各种图形（``UIView``）的方法，而且你也应该有能力作出这样的图表，甚至更好！

在开始讲解之前，我先粗略介绍一下利用``CAShapeLayer``画图的过程。这个过程有三个大前提：

- 因为``UIView``是对``CALayer``的封装，所以我们可以通过改变``UIView``所持有的``layer``属性来直接改变``UIView``的显示效果。
- ``CAShapeLayer``是``CALayer``的子类。
- ``CAShapeLayer``的使用是依赖于``UIBezierPath``的。``UIBezierPath``就是“路径”，可以理解为形状。不难理解，想象一下，如果我们想画一个图形，那么这个图形的形状（包括颜色）是必不可少的，而这个角色，就需要``UIBezierPath``来充当。

那么了这三个大前提，我们就可以知道如何画图了：
1. 实例化一个``UIBezierPath``，并赋给``CAShapeLayer``实例的``path``属性。
2. 将这个``CAShapeLayer``的实例添加到``UIView``的``layer``上。

简单的代码演示上述过程：
```objc
UIBezierPath *path = [UIBezierPath bezierPath];
...自定义path...
CAShapeLayer *shapLayer = [CAShapeLayer alloc] init];
shapLayer.path = path;
[self.view.layer addSubLayer:shapeLayer];
```

现在大致了解了画图的过程，我们来看一下该框架的作者是如何实现一个折线图的吧！

## 二. 源码解析

首先看一下整个绘制折线图的步骤：

1. 图表的初始化。
2. 获取横轴和纵轴的数据。
3. 计算折线上所有拐点的x，y值。
4. 计算每个拐点中间的圆圈的贝塞尔曲线（UIBezierPath）。
5. 生成每个拐点上面的Label（可有可无）。
6. 计算每条线段的贝塞尔曲线（UIBezierPath）。
7. 将上面得到的贝塞尔曲线赋给每条线段和圆圈的layer（CAShapeLayer）。
8. 绘制所有折线（所有线段+所有圆圈）。
9. 添加动画(可有可无)。
10. 绘制x，y坐标轴。

在集合代码具体讲解之前，我们要清楚三点（非常非常重要）：
1. 此折线图框架是可以设置拐点的样式的:可以设置为没有样式，也可以设置有样式：圆圈，方块，三角形。
  - 如果没有样式，则是简单的线段与线段的连接，在拐点处没有任何其他控件。
  - 如果是有样式的，那么这条折线里的每条线段（在本篇文章里统一说成线段）之间是**分离的**，因为线段中间有一个拐点控件。本篇文章介绍的是圆圈样式（如上图所示，拐点控件是一个圆圈）。
2. 上文提到过，该折线图框架可以在一张图表里同时显示多条折线，也就是可以设置多组数据（一条折线对应一组数据）。因此，上面的3，4，5，6，7项都是用各自不同的一个数组保存的，数组里的每一个元素对应一条折线的数据。
3. 既然同一个张图表可以显示多条折线：
  - 那么有些属性就是这些折线共有的，比如横坐标的value，这些属性保存在``PNLineChart``的实例里面。
  - 有些属性是每条折线私有的，比如每条折线的颜色，纵坐标value等等，这些属性保存在``PNLineChartData``里面。每一条折线对应一个``PNLineChartData``实例。这些实例汇总到一个数组里面，这个数组由``PNLineChart``的实例管理。

在充分了解了这三点之后，我们结合一下代码来看一下具体的实现：

### 1. 图表的初始化

```objc
- (id)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    
    if (self) {
        [self setupDefaultValues];
    }
    
    return self;
}
- (void)setupDefaultValues {
    
    [super setupDefaultValues];
    
    ...
    
    //四个内边距
    _chartMarginLeft = 25.0;
    _chartMarginRight = 25.0;
    _chartMarginTop = 25.0;
    _chartMarginBottom = 25.0;
    
    ...
    
    //真正绘制图表的画布（CavanWidth）的宽高
    _chartCavanWidth = self.frame.size.width - _chartMarginLeft - _chartMarginRight;
    _chartCavanHeight = self.frame.size.height - _chartMarginBottom - _chartMarginTop;
    ...
}
```

>上面这段代码我刻意省去了其他一些基本的设置，突出了图表布局的设置。

>布局的设置是图表绘制的前提，因为在最开始的时候，就应该计算出“画布”，也就是图表内容（不包括坐标轴和坐标label）的具体大小和位置（内边距以内的部分）。

>在这里，我们需要获取真正绘制图表的画布的宽高(``_chartCavanWidth``和``_chartCavanHeight``)。而且，要留意的是``_chartMarginLeft``在将来是要用作y轴Label的宽度，而``_chartMarginBottom``在将来是要用作x轴Label的高度的。

用一张图直观看一下：

![整个控件的大小和画布的大小](https://jknight-blog.oss-cn-shanghai.aliyuncs.com/source_code_analysis/pnchart_linechart.png)


### 2. 获取横轴和纵轴的数据

现在画布的位置和大小确定了，我们可以来看一下折线图是怎么画的了。
整个图表的绘制都基于三组数据（也可以是两组，为什么是两组，我稍后会给出解释），在讲解该框架是如何利用这些数据之前，我们来看一下这些数据是如何传进图表的：

```objc
...
//设置x轴的数据
[self.lineChart setXLabels:@[@"SEP 1", @"SEP 2", @"SEP 3", @"SEP 4", @"SEP 5", @"SEP 6", @"SEP 7"]];
//设置y轴的数据
[self.lineChart setYLabels:@[
                             @"0",@"50",@"100",@"150",@"200",@"250",@"300",
                             ]
 ];
// Line Chart
//设置每个点的y值
NSArray *dataArray = @[@0.0, @180.1, @26.4, @202.2, @126.2, @167.2, @276.2];
PNLineChartData *data = [PNLineChartData new];
data.pointLabelColor = [UIColor blackColor];
data.color = PNTwitterColor;
data.alpha = 0.5f;
data.itemCount = dataArray.count;
data.inflexionPointStyle = PNLineChartPointStyleCircle;
//这个block的作用是将上面的dataArray里的每一个值传给line chart。
data.getData = ^(NSUInteger index) {
    CGFloat yValue = [dataArray[index] floatValue];
    return [PNLineChartDataItem dataItemWithY:yValue];
};
//因为只有一条折线，所以只有一组数据
self.lineChart.chartData = @[data];
//绘制图表
[self.lineChart strokeChart];
//设置代理，响应点击
self.lineChart.delegate = self;
[self.view addSubview:self.lineChart];
```

上面的代码我可以略去了很多多余的设置，目的是突出图表数据的设置。

不难看出，这里有三个数据传给了lineChart：

1.x轴的数据：

```objc
[self.lineChart setXLabels:@[@"SEP 1", @"SEP 2", @"SEP 3", @"SEP 4", @"SEP 5", @"SEP 6", @"SEP 7"]];
```

>这段代码调用之后，实现了：
>1. 根据传入的xLabel数组里元素的数量，内容宽度(``_chartCavanWidth``)和下边距（``_chartMarginBottom``），计算每个xlabel的size。
>2. 根据xLabel所需要展示的内容(``NSString``)和宽度，实例化所有的xLabel（包括内容，位置）并显示出来，最后保存在``_xChartLabels``里面。

2.y轴的数据：

```objc
[self.lineChart setYLabels:@[
                @"0",@"50",@"100",@"150",@"200",@"250",@"300",
                ]
    ];
```

>这段代码调用之后，实现了：
>1. 根据传入的yLabel数组里元素的数量，内容高度(``_chartCavanHeight``)和左边距(``_chartMarginLeft``)，计算出每个ylabel的size。
>2. 根据xLabel所需要展示的内容(``NSString``)和宽度，实例化所有的yLabel（包括内容，位置）并显示出来，最后保存在``_yChartLabels``里面。

3.一条折线上每个点的实际值：

```objc
NSArray *dataArray = @[@0.0, @180.1, @26.4, @202.2, @126.2, @167.2, @276.2];
data.getData = ^(NSUInteger index) {
        CGFloat yValue = [dataArray[index] floatValue];
        return [PNLineChartDataItem dataItemWithY:yValue];
    };
self.lineChart.chartData = @[data];
```

>着重讲一下block：为什么不直接把这个数组(``dataArray``)作为line chart的属性传进去呢？我认为作者是想提供一个接口给用户一个自己转化y值的机会。


>像上文所说的，这里1，2是属于``lineChart``的数据，它适用于这张图表上所有的折线的。而3是属于某一条折线的。

>现在回答一下为什么可以只传入两组数据：因为y轴数据可以由每个点的实际值数组得出。可以简单想一下，我们可以获取这些真实值里面的最大值，然后将它n等分，就自然得到了y轴数据了。

我们已经布局了x轴和y轴的所有label，现在开始真正计算图表的数据了。

>注意：下面要介绍的3，4，5，6项都是在同一方法中计算出来，为了避免代码过长，我将每个部分分解开来做出解释。因为在同一方法里，所以这些涉及到for循环的语句是一致的。

>整个图表的绘制都是依赖于数据的处理，所以3，4，5，6项也是理解该框架的一个关键！


首先，我们需要计算每个数据点（拐点）的准确位置：

### 3. 计算折线上所有拐点的x，y值。

```objc
//遍历图表里每条折线
//还记得chartData属性么？它是用来保存多组折线的数据的，在这里只有一个折线，所以这个循环只循环一次）
for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
   //保存每条折线上的所有点的CGPoint  
   NSMutableArray *linePointsArray = [[NSMutableArray alloc] init];
    //遍历每条折线里的每个点    
    for (NSUInteger i = 0; i < chartData.itemCount; i++) {
       
        //传入index，获取y值(调用的是上文提到的block)
        yValue = chartData.getData(i).y;
        //当前点的x： _chartMarginLeft + _xLabelWidth / 2.0为0坐标，每多一个点就多一个_xLabelWidth
        int x = (int) (i * _xLabelWidth + _chartMarginLeft + _xLabelWidth / 2.0);
            
        //当前点的y：根据当前点的值和当前点所在的数组里的最大值的比例 以及 图表的总高度，算出当前点在图表里的y坐标
        int y = (int)[self yValuePositionInLineChart:yValue];
        //保存所有拐点的坐标
        [linePointsArray addObject:[NSValue valueWithCGPoint:CGPointMake(x, y)]];
    }
  //保存多条折线的CGPoint（这里只有一条折线，所以该数组只有一个元素）
  [pathPoints addObject:[linePointsArray copy]];
}
```

>在这里需要注意两点：
>1. 这里的``pathPoints``对应的是``lineChart``的``_pathPoints``属性。它是一个二维数组，保存每条折线上所有点的``CGPoint``。
>2. y值的计算：是需要从y的真实值转化为这个拐点在图表里的y坐标，转化方法的实现(仔细看几遍就懂了)：

```objc
- (CGFloat)yValuePositionInLineChart:(CGFloat)y {
    
    CGFloat innerGrade;//真实的最大值与最小值的差 与 当前点与最小值的差 的比值
    
    if (!(_yValueMax - _yValueMin)) {
        
        //特殊情况：当_yValueMax和_yValueMin相等的时候
        innerGrade = 0.5;
        
    } else {
        
        innerGrade = ((CGFloat) y - _yValueMin) / (_yValueMax - _yValueMin);
    }
    
    //innerGrade 与画布的高度（_chartCavanHeight）相乘，就能得出在画布中的高度
    return _chartCavanHeight - (innerGrade * _chartCavanHeight) - (_yLabelHeight / 2) + _chartMarginTop;
}
```



### 4. 计算每个拐点中间的圆圈的贝塞尔曲线（UIBezierPath）

```objc
//遍历图表里每条折线
for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
    //每条折线所有圆圈的贝塞尔曲线
    UIBezierPath *pointPath = [UIBezierPath bezierPath];    
    //inflexionWidth默认是6,是两个线段中间的距离（因为中间有一个圈圈，所以需要定一个距离）
    CGFloat inflexionWidth = chartData.inflexionPointWidth;
    //遍历每条折线里的每个点
    for (NSUInteger i = 0; i < chartData.itemCount; i++) {  
        //1. 计算圆圈的rect：已当前点为中心，以inflexionWidth为半径
        CGRect circleRect = CGRectMake(x - inflexionWidth / 2, y - inflexionWidth / 2, inflexionWidth, inflexionWidth);    
        //2. 计算圆圈的中心：由圆圈的x，y和inflexionWidth算出
        CGPoint circleCenter = CGPointMake(circleRect.origin.x + (circleRect.size.width / 2), circleRect.origin.y + (circleRect.size.height / 2));
        //3.1 移动到圆圈的右中部
        [pointPath moveToPoint:CGPointMake(circleCenter.x + (inflexionWidth / 2), circleCenter.y)];        
        //3.2 画线（圆形）
        [pointPath addArcWithCenter:circleCenter radius:inflexionWidth / 2 startAngle:0 endAngle:(CGFloat) (2 * M_PI) clockwise:YES];
    }
    //保存到pointsPath数组里
    [pointsPath insertObject:pointPath atIndex:lineIndex];
}
```

>在这里，``pointsPath``对应的是``lineChart``的``_pointsPath``属性。它是一个一维数组，保存每条折线上的圆圈贝塞尔曲线（UIBezierPath）。

### 5. 生成每个拐点上面的Label（可有可无）

```objc
//遍历图表里每条折线
for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
    //遍历每条折线里的每一段
    for (NSUInteger i = 0; i < chartData.itemCount; i++) {
    
        if (chartData.showPointLabel) {
            [gradePathArray addObject:[self createPointLabelFor:chartData.getData(i).rawY pointCenter:circleCenter width:inflexionWidth withChartData:chartData]];
        }
    }
}
```

>注意，在这里，这些label的实现是通过一个``CATextLayer``实现的，并不是生成一个个``Label``放在数组里保存，具体实现方法如下：

```objc
- (CATextLayer *)createPointLabelFor:(CGFloat)grade pointCenter:(CGPoint)pointCenter width:(CGFloat)width withChartData:(PNLineChartData *)chartData {
    
    //grade：提供textLayer显示的数值
    //pointCenter：根据pointCenter算出textLayer的x，y
    //width：根据width得到textLayer的总宽度
    //chartData：获取chartData里保存的textLayer上应该保存的字体大小和颜色
    
    CATextLayer *textLayer = [[CATextLayer alloc] init];
    [textLayer setAlignmentMode:kCAAlignmentCenter];
    
    //设置textLayer的背景色
    [textLayer setForegroundColor:[chartData.pointLabelColor CGColor]];
    [textLayer setBackgroundColor:self.backgroundColor.CGColor];
    
    //设置textLayer的字体大小和颜色
    if (chartData.pointLabelFont != nil) {
        [textLayer setFont:(__bridge CFTypeRef) (chartData.pointLabelFont)];
        textLayer.fontSize = [chartData.pointLabelFont pointSize];
    }
    
    //设置textLayer的高度
    CGFloat textHeight = (CGFloat) (textLayer.fontSize * 1.1);
    
    CGFloat textWidth = width * 8;
    CGFloat textStartPosY;
    
    textStartPosY = pointCenter.y - textLayer.fontSize;
    
    [self.layer addSublayer:textLayer];
    
    //设置textLayer的文字显示格式
    if (chartData.pointLabelFormat != nil) {
        [textLayer setString:[[NSString alloc] initWithFormat:chartData.pointLabelFormat, grade]];
    } else {
        [textLayer setString:[[NSString alloc] initWithFormat:_yLabelFormat, grade]];
    }
    
    //设置textLayer的位置和scale（1x，2x，3x）
    [textLayer setFrame:CGRectMake(0, 0, textWidth, textHeight)];
    [textLayer setPosition:CGPointMake(pointCenter.x, textStartPosY)];
    textLayer.contentsScale = [UIScreen mainScreen].scale;
    
    return textLayer;
}
```


### 6. 计算每条线段的贝塞尔曲线（UIBezierPath）

```objc
//遍历图表里每条折线
for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
    
    //每一条线段的贝塞尔曲线（UIBezierPath），用数组装起来
    NSMutableArray<UIBezierPath *> *progressLines = [NSMutableArray new];
    
    //chartPath（二维数组）：保存所有折线上所有线段的贝塞尔曲线。现在只有一条折线，所以只有一个元素
    [chartPath insertObject:progressLines atIndex:lineIndex];
    
    //progressLinePaths的每个元素是一个字典，字典里存放每一条线段的端点（from，to）
    NSMutableArray<NSDictionary<NSString *, NSValue *> *> *progressLinePaths = [NSMutableArray new];
    
    int last_x = 0;
    int last_y = 0;
    
    //遍历每条折线里的每一段
    for (NSUInteger i = 0; i < chartData.itemCount; i++) {
        
        if (i > 0) {
            //x，y的算法参考上文第三项
            // 计算index为0以后的点的位置
            float distance = (float) sqrt(pow(x - last_x, 2) + pow(y - last_y, 2));
            float last_x1 = last_x + (inflexionWidth / 2) / distance * (x - last_x);
            float last_y1 = last_y + (inflexionWidth / 2) / distance * (y - last_y);
            float x1 = x - (inflexionWidth / 2) / distance * (x - last_x);
            float y1 = y - (inflexionWidth / 2) / distance * (y - last_y);
            
            //当前线段的端点
            from = [NSValue valueWithCGPoint:CGPointMake(last_x1, last_y1)];
            to = [NSValue valueWithCGPoint:CGPointMake(x1, y1)];
            
            
            if(from != nil && to != nil) {
                //保存每一段的端点
                [progressLinePaths addObject:@{@"from": from,  @"to":to}];
                //保存所有的端点
                [lineStartEndPointsArray addObject:from];
                [lineStartEndPointsArray addObject:to];
            }
            //保存所有折点的坐标
            [linePointsArray addObject:[NSValue valueWithCGPoint:CGPointMake(x, y)]];
            //将当前的x转化为下一个点的last_x（y也一样）
            last_x = x;
            last_y = y;
        }
    }
    
    //pointsOfPath：保存所有折线里的所有线段两端的端点
    [pointsOfPath addObject:[lineStartEndPointsArray copy]];
    
    //根据每一条线段的两个端点，成生每条线段的贝塞尔曲线
    for (NSDictionary<NSString *, NSValue *> *item in progressLinePaths) {
        NSArray<NSDictionary *> *calculatedRanges =
        ...
        
        for (NSDictionary *range in calculatedRanges) {
            
            UIBezierPath *currentProgressLine = [UIBezierPath bezierPath];
            [currentProgressLine moveToPoint:[range[@"from"] CGPointValue]];
            [currentProgressLine addLineToPoint:[range[@"to"] CGPointValue]];
            [progressLines addObject:currentProgressLine];
            
        }
        }    
    }
```

### 7. 将上面得到的贝塞尔曲线赋给每条线段和圆圈的layer（CAShapeLayer）。


#### 7.1 所有线段的layer：
```objc
- (void)populateChartLines {
    
    //遍历每条线段
    for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
        
        NSArray<UIBezierPath *> *progressLines = self.chartPath[lineIndex];
        
        ...
        
        //_chartLineArray:二维数组，装载每个chartData对应的一个数组。这个数组的元素是这一条折线上所有线段对应的CAShapeLayer
        [self.chartLineArray[lineIndex] removeAllObjects];
        
        NSUInteger progressLineIndex = 0;;
        
        //遍历含有UIBezierPath对象元素的数组。在每个循环里新建一个CAShapeLayer对象，将UIBezierPath赋给它。
        for (UIBezierPath *progressLinePath in progressLines) {
            
            PNLineChartData *chartData = self.chartData[lineIndex];
            CAShapeLayer *chartLine = [CAShapeLayer layer];
            
            ...
            
            //将当前线段的UIBezierPath赋给当前线段的CAShapeLayer
            chartLine.path = progressLinePath.CGPath;
            
            //添加layer
            [self.layer addSublayer:chartLine];
            
            //保存当前线段的layer
            [self.chartLineArray[lineIndex] addObject:chartLine];
            progressLineIndex++;
        }
    }
}
```

#### 7.2 所有圆圈的layer：
```objc
- (void)recreatePointLayers {
- 
    for (PNLineChartData *chartData in _chartData) {
    
        // create as many chart line layers as there are data-lines
        [self.chartLineArray addObject:[NSMutableArray new]];

        // create point
        CAShapeLayer *pointLayer = [CAShapeLayer layer];
        pointLayer.strokeColor = [[chartData.color colorWithAlphaComponent:chartData.alpha] CGColor];
        pointLayer.lineCap = kCALineCapRound;
        pointLayer.lineJoin = kCALineJoinBevel;
        pointLayer.fillColor = nil;
        pointLayer.lineWidth = chartData.lineWidth;
        [self.layer addSublayer:pointLayer];
        [self.chartPointArray addObject:pointLayer];
    }
}
```
>注意，这里并没有将所有圆圈的``UIBezierPath``赋给对应的``layer``，而是在下一步，绘图的时候做的。


### 8.绘制所有折线（所有线段+所有圆圈）&& 9. 添加动画
```objc
- (void)strokeChart {
    
    ...
    
    // 绘制所有折线（所有线段+所有圆圈）
    // 遍历所有折线
    for (NSUInteger lineIndex = 0; lineIndex < self.chartData.count; lineIndex++) {
       
        PNLineChartData *chartData = self.chartData[lineIndex];
        
        //当前折线的所有线段的CAShapeLayer
        NSArray<CAShapeLayer *> *chartLines =self.chartLineArray[lineIndex];
        
        //当前折线的所有圆圈的CAShapeLayer
        CAShapeLayer *pointLayer = (CAShapeLayer *) self.chartPointArray[lineIndex];
        
        //开始绘制折线
        UIGraphicsBeginImageContext(self.frame.size);
        
        ...
        
        //当前折线的所有线段的UIBezierPath
        NSArray<UIBezierPath *> *progressLines = _chartPath[lineIndex];
        
        //当前折线的所有圆圈的UIBezierPath
        UIBezierPath *pointPath = _pointPath[lineIndex];

        //7.2将圆圈的UIBezierPath赋给了圆圈的CAShapeLayer
        pointLayer.path = pointPath.CGPath;

        //添加动画
        [CATransaction begin];
        
        for (NSUInteger index = 0; index < progressLines.count; index++) {
            CAShapeLayer *chartLine = chartLines[index];
            //chartLine strokeColor is already set. no need to override here
            [chartLine addAnimation:self.pathAnimation forKey:@"strokeEndAnimation"];
            chartLine.strokeEnd = 1.0;
        }

        // if you want cancel the point animation, comment this code, the point will show immediately
        if (chartData.inflexionPointStyle != PNLineChartPointStyleNone) {
            [pointLayer addAnimation:self.pathAnimation forKey:@"strokeEndAnimation"];
        }

        //提交动画
        [CATransaction commit];

       ...

        //绘制完毕
        UIGraphicsEndImageContext();
    }
    [self setNeedsDisplay];
}
```

这里要注意两点：
>1.如果想给layer添加动画，只需要实例化一个animation（在这里是``CABasicAnimation``）并调用layer的``addAnimation:``方法即可。我们看一下关于``CABasicAnimation``的实例化代码：

```objc
- (CABasicAnimation *)pathAnimation {
    if (self.displayAnimated && !_pathAnimation) {
        _pathAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
        //持续时间
        _pathAnimation.duration = 1.0;
         //类型
        _pathAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
        _pathAnimation.fromValue = @0.0f;
        _pathAnimation.toValue = @1.0f;
    }
    if(!self.displayAnimated) {
        _pathAnimation = nil;
    }
    return _pathAnimation;
}
```

>2.在这里调用了``setNeedsDisplay``方法之后，会调用``drawRect：``方法，在这个方法里，完成了x，y坐标轴的绘制：

### 10.绘制x，y坐标轴

```objc
- (void)drawRect:(CGRect)rect {
    
    //绘制坐标轴和背景竖线
    if (self.isShowCoordinateAxis) {
        
        CGFloat yAxisOffset = 10.f;
        
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        UIGraphicsPopContext();
        UIGraphicsPushContext(ctx);
        CGContextSetLineWidth(ctx, self.axisWidth);
        CGContextSetStrokeColorWithColor(ctx, [self.axisColor CGColor]);
        
        CGFloat xAxisWidth = CGRectGetWidth(rect) - (_chartMarginLeft + _chartMarginRight) / 2;
        CGFloat yAxisHeight = _chartMarginBottom + _chartCavanHeight;
        
        // 绘制xy轴
        CGContextMoveToPoint(ctx, _chartMarginBottom + yAxisOffset, 0);
        CGContextAddLineToPoint(ctx, _chartMarginBottom + yAxisOffset, yAxisHeight);
        CGContextAddLineToPoint(ctx, xAxisWidth, yAxisHeight);
        CGContextStrokePath(ctx);
        
        // 绘制y轴的箭头
        CGContextMoveToPoint(ctx, _chartMarginBottom + yAxisOffset - 3, 6);
        CGContextAddLineToPoint(ctx, _chartMarginBottom + yAxisOffset, 0);
        CGContextAddLineToPoint(ctx, _chartMarginBottom + yAxisOffset + 3, 6);
        CGContextStrokePath(ctx);
        
        // 绘制x轴的箭头
        CGContextMoveToPoint(ctx, xAxisWidth - 6, yAxisHeight - 3);
        CGContextAddLineToPoint(ctx, xAxisWidth, yAxisHeight);
        CGContextAddLineToPoint(ctx, xAxisWidth - 6, yAxisHeight + 3);
        CGContextStrokePath(ctx);
        
        //绘制x轴和y轴的label
        if (self.showLabel) {
            
            // 绘制x轴的小分割线
            CGPoint point;
            for (NSUInteger i = 0; i < [self.xLabels count]; i++) {
                point = CGPointMake(2 * _chartMarginLeft + (i * _xLabelWidth), _chartMarginBottom + _chartCavanHeight);
                CGContextMoveToPoint(ctx, point.x, point.y - 2);
                CGContextAddLineToPoint(ctx, point.x, point.y);
                CGContextStrokePath(ctx);
            }
            
            // 绘制y轴的小分割线
            CGFloat yStepHeight = _chartCavanHeight / _yLabelNum;
            for (NSUInteger i = 0; i < [self.xLabels count]; i++) {
                point = CGPointMake(_chartMarginBottom + yAxisOffset, (_chartCavanHeight - i * yStepHeight + _yLabelHeight / 2));
                CGContextMoveToPoint(ctx, point.x, point.y);
                CGContextAddLineToPoint(ctx, point.x + 2, point.y);
                CGContextStrokePath(ctx);
            }
        }
        
        UIFont *font = [UIFont systemFontOfSize:11];
        
        // 绘制y轴单位
        if ([self.yUnit length]) {
            CGFloat height = [PNLineChart sizeOfString:self.yUnit withWidth:30.f font:font].height;
            CGRect drawRect = CGRectMake(_chartMarginLeft + 10 + 5, 0, 30.f, height);
            [self drawTextInContext:ctx text:self.yUnit inRect:drawRect font:font color:self.yLabelColor];
        }
        
        // 绘制x轴的单位
        if ([self.xUnit length]) {
            CGFloat height = [PNLineChart sizeOfString:self.xUnit withWidth:30.f font:font].height;
            CGRect drawRect = CGRectMake(CGRectGetWidth(rect) - _chartMarginLeft + 5, _chartMarginBottom + _chartCavanHeight - height / 2, 25.f, height);
            [self drawTextInContext:ctx text:self.xUnit inRect:drawRect font:font color:self.xLabelColor];
        }
    }
    
    //绘制竖线
    if (self.showYGridLines) {
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        CGFloat yAxisOffset = _showLabel ? 10.f : 0.0f;
        CGPoint point;
        
        //每一条竖线的跨度
        CGFloat yStepHeight = _chartCavanHeight / _yLabelNum;
        
        //颜色
        if (self.yGridLinesColor) {
            CGContextSetStrokeColorWithColor(ctx, self.yGridLinesColor.CGColor);
        } else {
            CGContextSetStrokeColorWithColor(ctx, [UIColor lightGrayColor].CGColor);
        }
        
        //绘制每一条竖线
        for (NSUInteger i = 0; i < _yLabelNum; i++) {
            
            //拿到起点
            point = CGPointMake(_chartMarginLeft + yAxisOffset, (_chartCavanHeight - i * yStepHeight + _yLabelHeight / 2));
            
            //将画笔移动到起点
            CGContextMoveToPoint(ctx, point.x, point.y);
            
            //设置线的属性
            CGFloat dash[] = {6, 5};
            CGContextSetLineWidth(ctx, 0.5);
            CGContextSetLineCap(ctx, kCGLineCapRound);
            CGContextSetLineDash(ctx, 0.0, dash, 2);
            
            //设置这条线的终点
            CGContextAddLineToPoint(ctx, CGRectGetWidth(rect) - _chartMarginLeft + 5, point.y);
            
            //画线
            CGContextStrokePath(ctx);
        }
    }
    
    [super drawRect:rect];
}
```

到这里，一张完整的图表就可以画出来了。但是当前绘制的图表的折线都是直线，在上面还展示了一张曲线图。那么如果想绘制带有曲线的折线图应该怎么做呢？对，就是在贝塞尔曲线上下功夫。

当我们获取了所有线段的端点数组后，我们可以通过他们绘制弯曲的贝塞尔曲线（注意：该方法是对应上面对第6项的下半部分:生成每一个线段对贝塞尔曲线）：

```objc
//_showSmoothLines是用来控制是否绘制曲线折线的开关属性
if (self.showSmoothLines && chartData.itemCount >= 4) {
    
    for (NSDictionary<NSString *, NSValue *> *item in progressLinePaths) {
        
        ...
        for (NSDictionary *range in calculatedRanges) {
            
            UIBezierPath *currentProgressLine = [UIBezierPath bezierPath];
            CGPoint segmentP1 = [range[@"from"] CGPointValue];
            CGPoint segmentP2 = [range[@"to"] CGPointValue];
            
            [currentProgressLine moveToPoint:segmentP1];
            
            CGPoint midPoint = [PNLineChart midPointBetweenPoint1:segmentP1 andPoint2:segmentP2];
            
            //以每条线段以中间点为分割点，分成两组。每一组形成柔和的外凸曲线，而不是内凹
            [currentProgressLine addQuadCurveToPoint:midPoint
                                        controlPoint:[PNLineChart controlPointBetweenPoint1:midPoint andPoint2:segmentP1]];
            
            [currentProgressLine addQuadCurveToPoint:segmentP2
                                        controlPoint:[PNLineChart controlPointBetweenPoint1:midPoint andPoint2:segmentP2]];
            
            [progressLines addObject:currentProgressLine];
            [progressLineColors addObject:range[@"color"]];
        }
    }
}
```

注意一下生成弯曲的贝塞尔曲线的方法：``controlPointBetweenPoint1:andPoint2``:

```objc
//返回的点的x：是两点的中间；返回的点的y：与第二个点保持一致
+ (CGPoint)controlPointBetweenPoint1:(CGPoint)point1 andPoint2:(CGPoint)point2 {
    
    //线段两端的中间点
    CGPoint controlPoint = [self midPointBetweenPoint1:point1 andPoint2:point2];
    
    //末端点 和  中间点y的差
    CGFloat diffY = abs((int) (point2.y - controlPoint.y));
    
    if (point1.y < point2.y)
    //如果前端点更高
        controlPoint.y += diffY;
    
    else if (point1.y > point2.y)
    //如果后端点更高
        controlPoint.y -= diffY;
    
    return controlPoint;
}
```

OK，这样一来，直线的曲线图还有曲线的曲线图就大概掌握了。不过还差一个东西，就是图表对点击的响应。

我们需要思考一下：既然一张图表里可以显示多条折线，所以，当手指点击图表上的点以后，应该同时返回两个数据：
1. 点击了哪条折线上的这个点。
2. 点击了这条折线上的哪个点。

该框架的作者很好地完成了这两个任务，我们来看一下他是如何实现的：

### 响应点击的代理方法

#### 点击了哪条折线的判断
```objc
- (void)touchPoint:(NSSet *)touches withEvent:(UIEvent *)event {
    // Get the point user touched
    UITouch *touch = [touches anyObject];
    CGPoint touchPoint = [touch locationInView:self];
    
    for (NSUInteger p = 0; p < _pathPoints.count; p++) {
        
        NSArray *linePointsArray = _endPointsOfPath[p];
        
        //遍历每个端点
        for (NSUInteger i = 0; i < (int) linePointsArray.count - 1; i += 2) {
            
            CGPoint p1 = [linePointsArray[i] CGPointValue];
            CGPoint p2 = [linePointsArray[i + 1] CGPointValue];
            
            // Closest distance from point to line
            //触摸点到线段的距离
            float distance = (float) fabs(((p2.x - p1.x) * (touchPoint.y - p1.y)) - ((p1.x - touchPoint.x) * (p1.y - p2.y)));
            distance /= hypot(p2.x - p1.x, p1.y - p2.y);
            
            //如果距离小于5，则判断为“点击了当前的线段”，剩下的工作是判断具体点击了哪一条线段
            if (distance <= 5.0) {
                // Conform to delegate parameters, figure out what bezier path this CGPoint belongs to.
                NSUInteger lineIndex = 0;
                for (NSArray<UIBezierPath *> *paths in _chartPath) {
                    for (UIBezierPath *path in paths) {
                        //如果当前点处于UIBezierPath曲线上
                        BOOL pointContainsPath = CGPathContainsPoint(path.CGPath, NULL, p1, NO);
                        if (pointContainsPath) {
                            //点击了某一条折线
                            [_delegate userClickedOnLinePoint:touchPoint lineIndex:lineIndex];
                            return;
                        }
                    }
                    lineIndex++;
                }
            }
        }
    }
}
```

#### 点击了哪个点的判断
```objc
- (void)touchKeyPoint:(NSSet *)touches withEvent:(UIEvent *)event {
    // Get the point user touched
    UITouch *touch = [touches anyObject];
    CGPoint touchPoint = [touch locationInView:self];
    
    for (NSUInteger p = 0; p < _pathPoints.count; p++) {
        NSArray *linePointsArray = _pathPoints[p];
        
        //遍历所有的点
        for (NSUInteger i = 0; i < (int) linePointsArray.count - 1; i += 1) {
            
            CGPoint p1 = [linePointsArray[i] CGPointValue];
            CGPoint p2 = [linePointsArray[i + 1] CGPointValue];
            
            //获取到前一点的距离和后一点的距离
            float distanceToP1 = (float) fabs(hypot(touchPoint.x - p1.x, touchPoint.y - p1.y));
            float distanceToP2 = (float) hypot(touchPoint.x - p2.x, touchPoint.y - p2.y);
            
            float distance = MIN(distanceToP1, distanceToP2);
            
            //如果较小的距离小于10，则判定为点击了某个点
            if (distance <= 10.0) {
                //点击了某一条折线上的某个点
                [_delegate userClickedOnLineKeyPoint:touchPoint
                                           lineIndex:p
                                          pointIndex:(distance == distanceToP2 ? i + 1 : i)];
                
                return;
            }
        }
    }
}
```


这下就完整了，一个带有响应功能的图表就做好啦！


### 关于自定义UIView
这里只是将图表的``layer``加在了``UIView``的layer上，那如果想完全自定义view的话，只需将图表的``layer``完全赋给``UIView``的layer即可，这样一来，想要画出任意形状的``UIView``都可以。

---

## 三. 最后的话

关于图表的绘制，相对贝塞尔曲线与``CALayer``来说，数据的处理是一个比较麻烦的点。但是一旦学会了折线图的绘制，了解了绘图原理，那么其他类型的图表就可以触类旁通。



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



