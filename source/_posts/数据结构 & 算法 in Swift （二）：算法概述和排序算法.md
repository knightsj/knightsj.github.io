---
title: 数据结构 & 算法 in Swift （二）：算法概述和排序算法
tags: [iOS,Swift,Data Structure,Algorithm]
categories: Data Structure & Algorithm
---


![](http://jknight-blog.oss-cn-shanghai.aliyuncs.com/data_structure_algorithm_in_swift/ds_al_header.png)



本篇是《数据结构 & 算法 in Swift》系列连载的第二篇，内容分为如下两个部分：

- 算法基础：简单介绍算法的概念，时间复杂度与空间复杂度，递归，作为本文第二部分的背景知识。
- 排序算法：结合Swift的代码实现来讲解冒泡排序，选择排序，插入排序，归并排序，快速排序。


<!-- more -->


# 算法基础



> 该部分是给那些对算法以及相关知识不了解的读者准备的，如果已经对算法的相关知识有所了解，可以略过该部分，直接看本文的第二部分：排序算法。



关于该部分的讨论不属于本文介绍的重点，因此没有过多非常专业的论述，只是让那些对算法不了解的读者可以对算法先有一个基本的认识，为阅读和理解本文的第二部分做好准备。



## 算法的概念



算法是解决特定问题求解步骤的描述，在计算机中表现为指令的有限序列，并且每条指令表示一个或多个操作。

> 摘自《大话数据结构》



简单说来，算法就是“一个问题的解法”。对于相同一个问题，可能会有多种不同的解法。这些解法虽然可以得到相同的结果，但是每个算法的执行所需要的时间和空间资源却可以是千差万别的。



以消耗的时间的角度为出发点，我们看一下对于同一个问题，两种不同的解法的效率会相差多大：

现在让我们解决这个问题：**计算从1到100数字的总和**。

把比较容易想到的下面两种方法作为比较：

1. 1到100循环遍历逐步相加
2. 等差数列求和



用Swift函数来分别实现一下：

```swift
func sumOpration1(_ n:Int) -> Int{
    
    var sum = 0
    
    for i in 1 ... n {
        sum += i
    }
    
    return sum
}
sumOpration1(100)//5050



func sumOpration2(_ n:Int) -> Int{
    
    return (1 + n) * n/2
}
sumOpration2(100)//5050
```

上面的代码中，``sumOpration1``使用的是循环遍历的方式；``sumOpration2``使用的是等差数列求和的公式。



虽然两个函数都能得到正确的结果，但是不难看出两个函数实现的效率是有区别的:

**遍历求和所需要的时间是依赖于传入函数的n的大小的，而等差数列求和的方法所需要的时间对传入的n的大小是完全不依赖的。**

在遍历求和中，如果传入的n值是100，则需要遍历100次并相加才能得到结果，那么如果传入的n值是一百万呢？

而在等差数列求和的函数中，无论n值有多大，只需要一个公式就可以解决。



我们对此可以以小见大：世上千千万万种问题（算法题）可能也有类似的情况：相同的问题，相同的结果，但是执行效率缺差之千里。那么有没有什么方法可以度量某种算法的执行效率以方便人们去选择或是衡量算法之间的差异呢？ 答案是肯定的。

下面笔者就向大家介绍算法所消耗资源的两个维度：时间复杂度和空间复杂度。



## 时间复杂度与空间复杂度



### 时间复杂度

算法的时间复杂度是指算法需要消耗的时间资源。一般来说，计算机算法是问题规模!n的函数f(n)，算法的时间复杂度也因此记做：

![](https://user-gold-cdn.xitu.io/2018/2/8/1617175d7ca0cf3a)

常见的时间复杂度有：常数阶O(1)，对数阶O(log n），线性阶 O(n)，线性对数阶O(nlog n)，平方阶O(n^{2})，立方阶O(n^{3})，!k次方阶O(n^{k})，指数阶 O(2^{n})}。随着问题规模n的不断增大，上述时间复杂度不断增大，算法的执行效率越低。



拿其中几个复杂度做对比：

![](https://user-gold-cdn.xitu.io/2018/2/8/1617175e46a4baf6?w=840&h=460&f=png&s=366231)

从上图中我们可以看到，平方阶O(n^{2})随着n值的增大，其复杂度近乎直线飙升；而线性阶 O(n)随着n的增大，复杂度是线性增长的；我们还可以看到常数阶 O(1)随着n增大，其复杂度是不变的。



参考上一节的求和问题，我们可以看出来遍历求和的算法复杂度是线性阶O(n)：随着求和的最大数值的大小而线性增长；而等差数列求和算法的复杂度为常数阶 O(1)其算法复杂度与输入n值的大小无关。



读者可以试着想一个算法的复杂度与输入值n的平方成正比的算法。

在这里笔者举一个例子：求一个数组中某两个元素和为某个值的元素index的算法。数组为``[0,2,1,3,6]``，和为8：

```swift
func findTwoSum(_ array: [Int], target: Int) -> (Int, Int)? {
    
    guard array.count > 1 else {
        return nil
    }
    
    for i in 0..<array.count {
        
        let left = array[i]
        
        for j in (i + 1)..<array.count {
            let right = array[j]
            if left + right == target {
                return (i, j)
            }
        }
    }
    return nil
}

let array = [0,2,1,3,6]
if let indexes = findTwoSum(array, target: 8) {
    print(indexes) //1， 4
} else {
    print("No pairs are found")
}

```

上面的算法准确地计算出了两个元素的index为1和4。因为使用了两层的遍历，所以这里算法的复杂度是平方阶O(n^{2}。关于算法复杂度的详细推倒方法，可以参考网上和算法相关书籍的资料。



而其实，不需要遍历两层，只需要遍历一层即可：在遍历的时候，我么知道当前元素的值a，那么只要其余元素里面有值等于（target - a）的值即可。所以这次算法的复杂度就是线性阶O(n)了。



同样地，上面两种算法虽然可以达到相同的效果，但是当n非常大的时候，二者的计算效率就会相差更大：n = 1000的时候，二者得到结果所需要的时间可能会差好几百倍。可以说平方阶O(n^{2})复杂度的算法在数据量很大的时候是无法让人接受的。



### 空间复杂度

算法的空间复杂度是指算法需要消耗的空间资源。其计算和表示方法与时间复杂度类似，一般都用复杂度的渐近性来表示。同时间复杂度相比，空间复杂度的分析要简单得多。而且控件复杂度不属于本文讨论的重点，因此在这里不展开介绍了。





## 递归



在算法的实现中，遍历与递归是经常出现的两种操作。

对于遍历，无非就是使用一个for循环来遍历集合里的元素，相信大家已经非常熟悉了。但是对于递归操作就可能比较陌生。而且由于本文第二部分讲解算法的是时候有两个算法（也是比较重要）的算法使用了递归操作，所以为了能帮助大家理解这两个算法，笔者觉得有必要将递归单独拿出来讲解。



先看一下递归的概念。



### 递归的概念



递归的概念是：在[数学](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%A6)与[计算机科学](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)中，是指在函数的定义中使用函数自身的方法摘自维基百科

> 摘自维基百科



通过使用递归，可以把一个**大型复杂的问题逐层转化为一个与原问题相似的规模较小的问题来求解**。因此如果使用递归，可以达到使用少量的代码就可描述出解题过程所需的多次重复计算的目的，减少了程序的代码量 。



下面用一个例子来具体感受一下递归操作：

大家应该都比较熟悉阶乘的算法：3！= 3 * 2 * 1 ； 4！= 4 * 3 * 2 * 1

不难看出，在这里反复执行了一个逐渐-1和相乘的操作，如果可以使用某段代码达到重复调用的效果就很方便了，在这里就可以使用递归：



```swift
func factorial(_ n:Int) -> Int{
    return n < 2 ? 1: n * factorial(n-1)
}

factorial(3) //6
```



在上面的代码里，``factorial``函数调用了它自己，并且在n<2的时候返回了1；否则继续调用自己。

从代码本身其实不难理解函数调用的方式，但是这个6究竟是怎么算出来的呢？这就涉及到递归的实现原理了。



### 递归的实现原理



递归的调用实际上是通过调用栈（callback stack）来实现的，笔者用一张图从factorial(3)开始调用到最后得出6这个顺序之间发生的事情画了出来：



![](https://user-gold-cdn.xitu.io/2018/2/8/1617175e51624070?w=941&h=429&f=png&s=31535)

由上图可以看出，整个递归的过程和栈的入栈出栈的操作非常类似：橘黄色背景的圆角矩形代表了栈顶元素，也就是正在执行的操作，而灰色背景的圆角矩形则代表了其余的元素，它们的顺序就是当初被调用的顺序，而且在内容上保持了当时被调用时执行的代码。



现在笔者按照时间顺序从左到右来说明一下整个调用的过程：

- 最开始传入3之后，3满足了n>=2的条件，继续调用自己：3 * factorial(2) ，入栈。
- 传入2之后，2满足了n>=2的条件，继续调用自己：2 * factorial(1) ，入栈。
- 传入1之后，1满足了n<2的条件，停止调用自己，返回了1，出栈。
- 此时的栈顶元素为2 * factorial(1) ，而刚刚factorial(1)返回了1，所以现在这里变成了2 * 1 = 2，出栈。
- 同样地，此时栈顶元素为3 * factorial(2)里的 factorial(2)返回了2，所以现在这里变成了3 * 2 = 6，出栈。
- 最后，factorial(3)返回了6，出栈，递归结束。



按照笔者个人的理解：整个递归的过程可以大致理解为：在使递归继续的条件为false之前，持续递归调用，以栈的形式保存调用上下文（临时变量，函数等）。一旦这个条件变为true，则立即按照出栈的顺序（入栈顺序的逆序）来返回值，逐个传递，最终传递到最开始调用的那一层返回最终结果。



再简单点，递归中的“递”就是入栈，传递调用信息；“归”就是出栈，输出返回值。

而这个分界线就是递归的终止条件。很显然，这个终止条件在整个递归过程中起着举足轻重的作用。试想一下，如果这个条件永远不会改变，那么就会一直入栈，就会发生栈溢出的情况。



### 使用递归时需要注意的问题



基于上面递归的例子，我们将递归终止条件去掉：

```swift
func factorialInfinite(_ n:Int) -> Int{
    return n * factorialInfinite(n-1)
}

factorialInfinite(3)
```



这段代码如果放在playground里，经过一小段时间（几秒钟或更多）后，会报一个运行时错误。也可以在return语句上面写一个print函数打印一些字符串，接着就会看到不停的打印，直到运行时错误，栈溢出。



所以说在今后写关于递归的代码的时候，一定要注意递归的终止条件是否合理，因为即使条件存在也不一定就是合理的条件。我们看一下下面这个例子：



```swift
func sumOperation( _ n:Int) -> Int {
    if n == 0 {
        return 0
    }
    return n + sumOperation(n - 1)
}

sumOperation(2) //3
```



上面的代码跟阶乘类似，也是和小于当前参数的值相加，如果传入2，那么知道 n=0时就开始出栈，

2 + 1 + 0 = 3。看似没什么问题，但是如果一开始传入 - 1 呢？结果就是不停的入栈，直到栈溢出。因为 n == 0 这个条件在传入 - 1 的时候是无法终止入栈的，因为 - 1 之后的 -1 操作都是非0的。



所以说这个条件就不是合理的，一个比较合理的条件是 n < = 0。

```swift
func sumOperation( _ n:Int) -> Int {
    if n <= 0 {
        return 0
    }
    return n + sumOperation(n - 1)
}

sumOperation(-1) //0
```



相信到这里，读者应该对递归的使用，调用过程以及注意事项有个基本的认识了。

那么到这里，关于算法的基本介绍已经讲完了，下面正式开始讲解排序算法。





# 排序算法



讲解算法之前，我们先来看一下几个常见的排序算法的对比：


| 排序算法 | 平均情况下    | 最好情况     | 最坏情况     | 稳定性  | 空间复杂度   |
| ---- | -------- | -------- | -------- | ---- | ------- |
| 冒泡   | O(n^2)   | O(n）     | O(n^2)   | 稳定   | 1       |
| 选择排序 | O(n^2)   | O(n^2)   | O(n^2)   | 不稳定  | 1       |
| 插入排序 | O(n^2)   | O(n）     | O(n^2)   | 稳定   | 1       |
| 希尔排序 | O(nlogn) | 依赖步长     | 依赖步长     | 稳定   | 1       |
| 堆排序  | O(nlogn) | O(nlogn) | O(nlogn) | 稳定   | 1       |
| 归并排序 | O(nlogn) | O(nlogn) | O(nlogn) | 稳定   | O(n）    |
| 快速排序 | O(nlogn) | O(nlogn) | O(n^2)   | 不稳定  | O(logn) |



> 最好情况和最坏情况以及稳定性的概念不在本文的讨论范围之内，有兴趣的读者可以查阅相关资料。



现在只看平均情况下的性能：

- 冒泡排序，选择排序，插入排序的时间复杂度为平方阶O(n^{2})
- 希尔排序，堆排序，归并排序，快速排序的时间复杂度为线性对数阶O(nlog n)



本篇要给大家介绍的是冒泡排序，选择排序，插入排序，归并排序和快速排序。

希尔排序是基于插入排序，理解了插入排序以后，理解希尔排序会很容易，故在本文不做介绍。堆排序涉及到一个全新的数据结构：堆，所以笔者将堆这个数据结构和堆排序放在下一篇来做介绍。





## 排序初探



在讲排序算法之前，我们先看一种最简单的排序算法（也是性能最低的，也是最好理解的），在这里先称之为“交换排序”。

> 注意，这个名称是笔者自己起的，在互联网和相关技术书籍上面没有对该算法起名。



### 算法讲解



用两个循环来嵌套遍历：

- 外层遍历数组从0到末尾的元素，索引为i.
- 里层遍历数组从i+1至数组末尾的元素，索引为j。
- 当i上的元素比j上的元素大的时候，交换i和j的元素，目的是保持index为i的元素是最小的。



我们用一个例子看一下是怎么做交换的：

给定一个初始数组：``array = [4, 1, 2, 5, 0]``



**i = 0 时**：

- array[0] > array[1] : 交换4和1：``[1, 4, 2, 5, 0]``，内层的j继续遍历，j++。
- array[0] > array[4] : 交换0和1：``[0, 4, 2, 5, 1]``，i = 0的外层循环结束，i++。

**i  = 1时**：

- array[1] > array[2] : 交换2和4：``[0, 2, 4, 5, 1]``，内层的j继续遍历，j++。
- array[1] > array[4] : 交换1和2：``[0, 1, 4, 5, 2]``，i = 1的外层循环结束，i++。

**i = 2 时**：

- array[2] > array[4] : 交换2和4：``[0, 1, 2, 5, 4]``，i = 2的外层循环结束，i++。

**i = 3 时**：

- array[3] > array[4] : 交换5和4：``[0, 1, 2, 4, 5]``，i = 3的外层循环结束，i++。

**i = 4 时**：不符合内循环的边界条件，不进行内循环，排序结束。



那么用代码如何实现呢？



### 代码实现

```swift
func switchSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    for i in 0 ..< array.count {
        
        for j in i + 1 ..< array.count {
          
            if array[i] > array[j] {
                array.swapAt(i, j) 
                print("\(array)")
            }
        }
    }
    
    return array 
}
```



这里面``swapAt``函数是使用了Swift内置的数组内部交换两个index的函数，在后面会经常用到。

为了用代码验证上面所讲解的交换过程，可以在``swapAt``函数下面将交换元素后的数组打印出来：

```swift
var originalArray = [4,1,2,5,0]
print("original array:\n\(originalArray)\n")

func switchSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    for i in 0 ..< array.count {
        
        for j in i + 1 ..< array.count {
          
            if array[i] > array[j] {
                array.swapAt(i, j) 
                print("\(array)")
            }
        }
    }
    
    return array   
}


switchSort(&originalArray)
```



打印结果：

```
original array:
[4, 1, 2, 5, 0]


switch sort...
[1, 4, 2, 5, 0]
[0, 4, 2, 5, 1]
[0, 2, 4, 5, 1]
[0, 1, 4, 5, 2]
[0, 1, 2, 5, 4]
[0, 1, 2, 4, 5]
```



验证后我们可以看到，结果和上面分析的结果是一样的。

各位读者也可以自己设置原数组，然后在运行代码之前按照自己的理解，把每一次交换的结果写出来，接着和运行算法之后进行对比。该方法对算法的理解很有帮助，推荐大家使用~



> 请务必理解好上面的逻辑，可以通过动笔写结果的方式来帮助理解和巩固，有助于对下面讲解的排序算法的理解。



大家看上面的交换过程（排序过程）有没有什么问题？相信细致的读者已经看出来了：**在原数组中，1和2都是比较靠前的位置，但是经过中间的排序以后，被放在了数组后方，然后再次又交换回来**。这显然是比较低效的，给人的感觉像是做了无用功。



那么有没有什么方法可以优化一下交换的过程，让交换后的结果与元素最终在数组的位置基本保持一致呢？



答案是肯定的，这就引出了笔者要第一个正式介绍的排序算法冒泡排序：



## 冒泡排序



### 算法讲解

与上面讲的交换排序类似的是，冒泡排序也是用两层的循环来实现的；但与其不同的是：



- 循环的边界条件：冒泡排序的外层是[0,array.count-1);内层是[0,array.count-1-i)。可以看到内层的范围是不断缩小的，而且范围的前端不变，后端在向前移。


- 交换排序比较的是内外层索引的元素（array[i] 和 array[j]）,但是冒泡排序比较的是两个相邻的内层索引的元素：array[j]和array[j+1]。

笔者用和上面交换排序使用的同一个数组来演示下元素是如何交换的：



初始数组：``array = [4, 1, 2, 5, 0]``



**i = 0 时**：

- array[0] > array[1] : 交换4和1：``[1, 4, 2, 5, 0]``，内层的j继续遍历，j++。
- array[1] > array[2] : 交换4和2：``[1, 2, 4, 5, 0]``，内层的j继续遍历，j++。
- array[2] < array[3] : 不交换，内层的j继续遍历，j++。
- array[3] > array[4] : 交换5和0：``[1, 2, 4, 0, 5]``，i = 0的外层循环结束，i++。



**i  = 1时**：

- array[2] > array[3] : 交换2和4：``[1, 2, 0, 4, 5]``，内层的j继续遍历，j++。
- array[3] < array[4] : 不交换，i = 1的外层循环结束，i++。



**i = 2 时**：

- array[1] > array[2] : 交换2和0：``[1, 0, 2, 4, 5]``，内层的j继续遍历，j++，直到退出i=2的外层循环，i++。



**i = 3 时**：

- array[0] > array[1] : 交换1和0：``[0, 1, 2, 4, 5]``，内层的j继续遍历，j++，直到退出i=3的外层循环，i++。



i = 4 时：不符合外层循环的边界条件，不进行外层循环，排序结束。



### 代码实现



我们来看一下冒泡排序的代码：

```swift
func bubbleSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    for i in 0 ..< array.count - 1 {

        for j in 0 ..< array.count - 1 - i {
          
            if array[j] > array[j+1] {
                array.swapAt(j, j+1)                 
            }
        }
    }
    return array
}
```

从上面的代码我们可以清楚地看到循环遍历的边界条件和交换时机。同样地，我们添加上log，将冒泡排序每次交换后的数组打印出来（为了进行对比，笔者将交换排序的log也打印了出来）：



```
original array:
[4, 1, 2, 5, 0]

switch sort...
[1, 4, 2, 5, 0]
[0, 4, 2, 5, 1]
[0, 2, 4, 5, 1]
[0, 1, 4, 5, 2]
[0, 1, 2, 5, 4]
[0, 1, 2, 4, 5]

bubble sort...
[1, 4, 2, 5, 0]
[1, 2, 4, 5, 0]
[1, 2, 4, 0, 5]
[1, 2, 0, 4, 5]
[1, 0, 2, 4, 5]
[0, 1, 2, 4, 5]
```



从上面两组打印可以看出，冒泡排序算法解决了交换排序算法的不足：

- 原来就处于靠前位置的1，2两个元素，在排序的过程中一直是靠前的。
- 原来处于末尾的0元素，在冒泡排序的过程中一点一点地向前移动，最终到了应该处于的位置。

现在我们知道冒泡排序是好于交换排序的，而且它的做法是相邻元素的两两比较：如果是逆序（左大右小）的话就做交换。

那么如果在排序过程中，数组已经变成有序的了，那么再进行两两比较就很不划算了。



为了证实上面这个排序算法的局限性，我们用新的测试用例来看一下：

```swift
var originalArray = [2,1,3,4,5]
```



而且这次我们不仅仅在交换以后打log，也记录一下作比较的次数：

```swift
func bubbleSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }    
    var compareCount = 0
    
    for i in 0 ..< array.count - 1 {

        for j in 0 ..< array.count - 1 - i {

            compareCount += 1
            print("No.\(compareCount) compare \(array[j]) and \(array[j+1])")
            
            if array[j] > array[j+1] {
                array.swapAt(j, j+1) //keeping index of j is the smaller one
                print("after swap: \(array)")
                
            }
        }
    }
    return array
}
```



打印结果：

```
original array:
[2, 1, 3, 4, 5]


bubble sort...
No.1 compare 2 and 1
after swap: [1, 2, 3, 4, 5] //already sorted, but keep comparing
No.2 compare 2 and 3
No.3 compare 3 and 4
No.4 compare 4 and 5
No.5 compare 1 and 2
No.6 compare 2 and 3
No.7 compare 3 and 4
No.8 compare 1 and 2
No.9 compare 2 and 3
No.10 compare 1 and 2
```



从打印的结果可以看出，其实在第一次交换过之后，数组已经是有序的了，但是该算法还是继续在比较，做了很多无用功，能不能有个办法可以让这种两两比较在已知有序的情况下提前结束呢？答案是肯定的。



提前结束这个操作很容易，我们只需要跳出最外层的循环就好了。关键是这个时机：我们需要让算法自己知道**什么时候数组已经是有序的了**。



是否已经想到了呢？就是在一次内循环过后，如果没有发生元素交换，就说明数组已经是有序的，不需要再次缩小内循环的范围继续比较了。所以我们需要在外部设置一个布尔值的变量来标记“该数组是否有序”：



我们将这个算法称为：advanced bubble sort

```swift
func bubbleSortAdvanced(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    for i in 0 ..< array.count - 1 {
        
        //bool switch
        var swapped = false
      
        for j in 0 ..< array.count - i - 1 {
            
            if array[j] > array [j+1] {
                array.swapAt(j, j+1) 
                swapped = true;
            }
        }
        
        //if there is no swapping in inner loop, it means the the part looped is already sorted,
        //so it's time to break
        if (swapped == false){ break }
    }
    
    return array
    
}
```



从上面的代码可以看出，在第一个冒泡排序的算法之内，只添加了一个``swapped``这个布尔值，默认为false：

- 如果在当前内循环里面没有发生过元素交换，则说明当前内循环范围的元素都是有序的；那么就说明后续的内循环范围的元素也是有序的（因为内循环每次迭代后都会缩小），就可以跳出循环了。
- 反之，如果在当前内循环里发生过元素交换，则说明当前内循环很可能是无序的（也可能是有序的，但是有序性需要在下一个内循环中验证，所以还是不能提前退出，还需要进行一次内循环）。

为了验证上面这个改进冒泡排序是否能解决最初给出的冒泡排序的问题，我们添加上对比次数的log：



```
original array:
[2, 1, 3, 4, 5]


bubble sort...
No.1 compare 2 and 1
after swap: [1, 2, 3, 4, 5]
No.2 compare 2 and 3
No.3 compare 3 and 4
No.4 compare 4 and 5
No.5 compare 1 and 2
No.6 compare 2 and 3
No.7 compare 3 and 4
No.8 compare 1 and 2
No.9 compare 2 and 3
No.10 compare 1 and 2
bubble sort time duration : 1.96ms

advanced bubble sort...
No.1 compare 2 and 1
after swap: [1, 2, 3, 4, 5]
No.2 compare 2 and 3
No.3 compare 3 and 4
No.4 compare 4 and 5
No.5 compare 1 and 2
No.6 compare 2 and 3
No.7 compare 3 and 4
```



我们可以看到，在使用改进的冒泡排序后，对比的次数少了3次。之所以没有立即返回，是因为即使在交换完变成有序数组以后，也无法在当前内循环判断出是有序的。需要在下次内循环才能验证出来。



因为数组的元素数量比较小，所以可能对这个改进所达到的效果体会得不是很明显。现在我们增加一下数组元素的个数，并用记录**比较总和**的方式来看一下二者的区别：



```swift
original array:
[2, 1, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]

bubble sort...
total compare count： 91


advanced bubble sort...
total compare count： 25
```



从比较结果可以看出，这两种算法在该测试样本下的差距是比较大的，而且随着元素个数的增多这个差距会越来越大（因为做了更多没有意义的比较）。



虽然这种测试样本比较极端，但是在某种意义上还是优化了最初的冒泡排序算法。一般在网上的冒泡排序算法应该都能看到这个优化版的。



现在我们知道这个优化版的冒泡排序算法可以在知道当前数组已经有序的时候提前结束，但是毕竟不断的交换还是比较耗费性能的，有没有什么方法可以只移动一次就能做好当前元素的排序呢？答案又是肯定的，这就引出了笔者即将介绍的选择排序算法。



## 选择排序



### 算法讲解

选择排序也是两层循环：

- 外层循环的边界是[0,array.count-1)，index为i。
- 内层循环的边界是[i+1,array.count)，index为j。可以看到内层的范围也是不断缩小的，而且范围的前端一直后移，后端保持不变。

具体做法是：

- 在外层循环的开始，将i作为最小值index（很可能不是该数组的最小值）。
- 在内层循环里面找到当前内层循环范围内的最小值，并与已经记录的最小值作比较：
  - 如果与当前记录的最小值index不同，则替换
  - 如果与当前记录的最小值index相同，则不替换

我们还是用手写迭代的方式看一下选择排序的机制，使用的数组和上面交换排序和冒泡排序（非优化版）的数组一致：``[4, 1, 2, 5, 0]``





i = 0 时：

1. 记录当前的最小值的index为0，当前最小值为4。
2. 内层循环开始，找到[1,5)之间的最小值为0，0的index为4，与当前最小值的index0不同，所以二者要做交换。交换后的数组：``[0, 1, 2, 5, 4]``。当前内层循环结束，i++。

**i = 1 时**：

1. 记录当前的最小值的index为1，当前最小值为1。
2. 内层循环开始，找到[2,5)之间的最小值为1，与当前记录的最小值index相同。也就是说后面没有比1还要小的了，不做交换。当前内层循环结束，i++。



**i = 2 时**：

1. 记录当前的最小值的index为2，当前最小值为2。
2. 内层循环开始，找到[3,5)之间的最小值为2，与当前记录的最小值index相同。也就是说后面没有比1还要小的了，不做交换。当前内层循环结束，i++。



**i = 3 时**：

1. 记录当前的最小值的index为3，当前最小值为2。
2. 内层循环开始，找到[4,5)之间的最小值为4，4的index为4，与当前记录的最小值index3不同，所以二者要做交换。交换后的数组：``[0, 1, 2, 4, 5]``。当前内层循环结束，i++。



**i = 4 时**：不符合外层循环的边界条件，不进行外层循环，排序结束。



我们可以看到，同样的初始序列，使用选择排序只进行了2次交换，因为它知道需要替换的最小值是什么，做了很少没意义的交换。



### 代码实现

我们用代码来实现一下上面选择排序的算法：

```swift
func selectionSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }

    for i in 0 ..< array.count - 1{
        
        var min = i
        
        for j in i + 1 ..< array.count {
            
            if array[j] < array[min] {
                min = j 
            }
        }
        
        //if min has changed, it means there is value smaller than array[min]
        //if min has not changed, it means there is no value smallter than array[min]
        if i != min {
            array.swapAt(i, min) 
        }
    }
    return array
}
```



从上面的代码可以看到，在这里使用了``min``这个变量记录了当前外层循环所需要被比较的index值，如果当前外层循环的内层循环内部找到了比这个最小值还小的值，就替换他们。



下面我们使用log来看一下此时选择排序作替换的次数：

```
original array:
[4, 1, 2, 5, 0]

advanced bubble sort...
after swap: [1, 4, 2, 5, 0]
after swap: [1, 2, 4, 5, 0]
after swap: [1, 2, 4, 0, 5]
after swap: [1, 2, 0, 4, 5]
after swap: [1, 0, 2, 4, 5]
after swap: [0, 1, 2, 4, 5]

selection sort...
after swap: [0, 1, 2, 5, 4]
after swap: [0, 1, 2, 4, 5]
```



从上面的log可以看出二者的对比应该比较明显了。



为了进一步验证选择排序的性能，笔者在网上找到了两个工具：

- 计算程序运行时间的类：``executionTimeInterval.swift``
- 生成各种类型随机数的Array的分类：``Array+Extension.swift``

首先看``executionTimeInterval.swift``的实现：



```swift
//time interval
public func executionTimeInterval(block: () -> ()) -> CFTimeInterval {
    let start = CACurrentMediaTime()
    block();
    let end = CACurrentMediaTime()
    return end - start
}


//formatted time
public extension CFTimeInterval {
    public var formattedTime: String {
        return self >= 1000 ? String(Int(self)) + "s"
            : self >= 1 ? String(format: "%.3gs", self)
            : self >= 1e-3 ? String(format: "%.3gms", self * 1e3)
            : self >= 1e-6 ? String(format: "%.3gµs", self * 1e6)
            : self < 1e-9 ? "0s"
            : String(format: "%.3gns", self * 1e9)
    }
}
```



第一个函数以block的形式传入需要测试运行时间的函数，返回了函数运行的时间。

第二个函数是``CFTimeInterval``的分类，将秒数添加了单位：毫秒级的以毫秒显示，微秒级的以微秒显示，大于1秒的以秒单位显示。



使用方法是：将两个swift文件拖进playground里面的Sources文件夹里，并点击二者后，进入playground内部：



```swift
var selectionSortedArray = [Int]()
var time4 = executionTimeInterval{
    selectionSortedArray = selectionSort(&originalArray4) //要测试的函数
}

print("selection sort time duration : \(time4.formattedTime)") //打印出时间
```



再来看一下``Array+Extension.swift``类：

先介绍其中的一个方法，生成随机数组：

```swift
import Foundation

extension Array {
    
    static public func randomArray(size: Int, maxValue: UInt) -> [Int] {
        var result = [Int](repeating: 0, count:size)
        
        for i in 0 ..< size {
            result[i] = Int(arc4random_uniform(UInt32(maxValue)))
        }
        
        return result
    }
}
```



这个方法只需要传入数组的大小以及最大值就可以生成一个不超过这个最大值的随机数组。

比如我们要生成一个数组长度为10，最大值为100的数组：

```swift
var originalArray = Array<Int>.randomArray(size: inputSize, maxValue:100)
//originalArray:[87, 56, 54, 20, 86, 33, 41, 9, 88, 55]
```



那么现在有了上面两个工具，我们就可以按照我们自己的意愿来生成测试用例数组，并且打印出所用算法的执行时间。我们现在生成一个数组长度为10，最大值为100的数组，然后分别用优化的冒泡排序和选择排序来看一下二者的性能：



```
original array:
[1, 4, 80, 83, 92, 63, 83, 23, 9, 85]

advanced bubble sort...
advanced bubble sort result: [1, 4, 9, 23, 63, 80, 83, 83, 85, 92] time duration : 8.53ms

selection sort...
selection sort result: [1, 4, 9, 23, 63, 80, 83, 83, 85, 92] time duration : 3.4ms
```



我们现在让数组长度更长一点:一个长度为100，最大值为200：

```
advanced bubble sort...
advanced bubble sort sorted elemets: 100 time duration : 6.27s

selection sort...
selection sort sorted elemets: 100 time duration : 414ms
```



可以看到，二者的差别大概在12倍左右。这个差别已经很大了，如果说用选择排序需要1天的话，冒泡排序需要12天。



现在我们学习了选择排序，知道了它是通过减少交换次数来提高排序算法的性能的。

但是关于排序，**除了交换操作以外，对比操作也是需要时间的**：选择排序通过内层循环的不断对比才得到了当前内层循环的最小值，然后进行后续的判断和操作。



那么有什么办法可以减少对比的次数呢？猜对了，答案又是肯定的。这就引出了笔者下面要说的算法：插入排序算法。





## 插入排序



### 算法讲解

插入排序的基本思想是：从数组中拿出一个元素（通常就是第一个元素）以后，再从数组中按顺序拿出其他元素。如果拿出来的这个元素比这个元素小，就放在这个元素左侧；反之，则放在右侧。整体上看来有点和玩儿扑克牌的时候将刚拿好的牌来做排序差不多。



选择排序也是两层循环：

- 外层循环的边界是[1,array.count)，index为i。
- 内层循环开始的时候初始index j = i，然后使用一个while循环，循环条件是``j>0 && array[j] < array[j - 1]``,循环内侧是交换j-1和j的元素，并使得j-1。可以简单理解为如果当前的元素比前一个元素小，则调换位置；反之进行下一个外层循环。

下面我们还是用手写迭代的方式看一下插入排序的机制，使用的数组和上面选择排序的数组一致：``[4, 1, 2, 5, 0]``





**i = 1 时**：

1. j = 1：array[1] < array[0]， 交换4和1：``[1, 4, 2, 5, 0]``，j-1之后不符合内层循环条件，退出内层循环，i+1。



**i = 2 时**：

1. j = 2，array[3] < array[2]，交换4和2：``[1, 2, 4, 5, 0]``，j向左移动，array[2] > array[1]，不符合内层循环条件，退出内层循环，i+1。



**i = 3 时**：

1. j = 3，array[3] > array[2]，不符合内层循环条件，退出内层循环，i+1。



**i = 4 时**：

1. j = 4，array[4] < array[3]，交换5和0：``[1, 2, 4, 0, 5]``，j -1。
2. j = 3，array[3] < array[2]，交换4和0：``[1, 2, 0, 4, 5]``，j -1。
3. j = 2，array[2] < array[1]，交换4和0：``[1, 0, 2, 4, 5]``，j -1。
4. j = 1，array[1] < array[0]，交换1和0：``[0, 1, 2, 4, 5]``，j -1 = 0，不符合内层循环条件，退出内层循环，i+1 = 5，不符合外层循环条件，排序终止。



从上面的描述可以看出，和选择排序相比，**插入排序的内层循环是可以提前推出的**，其条件就是``array[j] >= array[j - 1]``,也就是说，当前index为j的元素只要比前面的元素大，那么该内层循环就立即退出，不需要再排序了，因为该算法从一开始就是小的放前面，大的放后面。



### 代码实现



下面我们通过代码来看一下如何实现插入排序算法:

```swift
func insertionSort(_ array: inout [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    for i in 1..<array.count {
        
        var j = i
        while j > 0 && array[j] < array[j - 1] {
             array.swapAt(j - 1, j)
            j -= 1
        }
    }
    return array
}
```

从上面的代码可以看出插入排序内层循环的条件：``j > 0 && array[j] < array[j - 1]``。只要当前元素比前面的元素小，就会一直交换下去；反之，当大于等于前面的元素，就会立即跳出循环。



之前笔者有提到相对于选择排序，说插入排序可以减少元素之间对比的次数，下面我们通过打印对比次数来对比一下两种算法：

使用元素个数为50，最大值为50的随机数组：

```
selection sort...
compare times:1225
selection sort time duration : 178ms

insertion sort...
compare times:519
insertion sort time duration : 676ms
```

我们可以看到，使用选择排序的比较次数比插入排序的比较次数多了2倍。但是遗憾的是整体的性能选择排序要高于插入排序。



也就是说虽然插入排序的比较次数少了，但是交换的次数却比选择排序要多，所以性能上有时可能不如选择排序。

> 注意，这不与笔者之前的意思相矛盾，笔者只是说在减少比较次数上插入排序是优于选择排序的，但没有说插入排序整体上优于选择排序。



那么有何种特性的数组可以让排序算法有其用武之地呢？

从上面使用插入排序来排序``[4, 1, 2, 5, 0]``这个数组的时候，我们可以看到，因为0这个元素已经在末尾了，所以在j=4的时候我们费了好大劲才把它移到前面去。



那么将这个情况作为一个极端，我们可以这样想：如果这个数组里的元素里的index大致于最终顺序差不多的情况是不是就不用做这么多的搬移了？。这句话听起来像是理所当然的话，但是有一种数组属于“基本有序”的数组，这种数组也是无需的，但是它在整体上是有序的，比如：

``[2,1,3,6,4,5,9,7,8]``

用笔者的话就叫做整体有序，部分无序。

我们可以简单用这个数组来分别进行选择排序和插入排序做个比较：

```
selection sort...
compare times:36
selection sort time duration : 4.7ms

insertion sort...
compare times:5
insertion sort time duration : 3.2ms
```



我们可以看到插入排序在基本有序的测试用例下表现更好。为了让差距更明显，笔者在``Array+Extension.swift``文件里增加了一个生成基本有序随机数组的方法：



```swift
static public func nearlySortedArray(size: Int, gap:Int) -> [Int] {

    var result = [Int](repeating: 0, count:size)

    for i in 0 ..< size {
        result[i] = i
    }

    let count : Int = size / gap
    var arr = [Int]()

    for i in 0 ..< count {
        arr.append(i*gap)
    }

    for j in 0 ..< arr.count {
        let swapIndex = arr[j]
        result.swapAt(swapIndex,swapIndex+1)
    }

    return result
}
```



该函数需要传入数组的长度以及需要打乱顺序的index的跨度，它的实现是这样子的：

- 首先生成一个完全有序的序列。
- 将数组长度除以跨度来得出需要交换的index的个数count。
- 根据这个count可以得出需要交换的index，把这些index放在一个新的arr里面
- 便利这个arr来取出index，将之前生成好的w安全有序的数组的index于index+1做交换。

举个例子，如果我们生成一个数组长度为12，跨度为3的基本有序的数组，就可以这么调用：

```swift
var originalArray = Array<Int>.nearlySortedArray(size: 12, gap: 3)
//[1, 0, 2, 4, 3, 5, 7, 6, 8, 10, 9, 11]
```

跨度为3，说明有12/3 = 4 - 1 = 3 个元素需要调换位置，序号分别为0，3，6，9。所以序号为0，1；3，4；6，7；9，10的元素被调换了位置，可以看到调换后的数组还是基本有序的。



现在我们可以用一个比较大的数组来验证了：

```swift
var originalArray = Array<Int>.nearlySortedArray(size: 100, gap: 10)
```



结果为：

```
selection sort...
compare times:4950
selection sort time duration : 422ms

insertion sort...
compare times:10
insertion sort time duration : 56.4ms
```



我们可以看到差距是非常明显的，插入排序的性能是选择排序的性能的近乎10倍



## 归并排序



### 算法讲解

归并排序使用了算法思想里的**分治思想**（divide conquer）。顾名思义，就是将一个大问题，分成类似的小问题来逐个攻破。在归并排序的算法实现上，首先逐步将要排序的数组等分成最小的组成部分（通常是1各元素），然后再反过来逐步合并。



用一张图来体会一下归并算法的实现过程： 



![](https://user-gold-cdn.xitu.io/2018/2/8/1617175e83e236cf?w=944&h=637&f=png&s=70522)



上图面的虚线箭头代表拆分的过程；实线代表合并的过程。仔细看可以发现，拆分和归并的操作都是重复进行的，在这里面我们可以使用递归来操作。



首先看一下归并的操作：

归并的操作就是把两个数组（在这里这两个数组的元素个数通常是一致的）合并成一个完全有序数组。

归并操作的实现步骤是：

- 新建一个空数组，该数组用于存放合并后的有序数组。
- 两个传入的数组从index 0 开始两两比较，较小的元素放在新建的空数组中，index + 1; 较大的元素不作操作，index 不变，然后继续两两比较。知道index移到末尾为止。
- 个别情况当两个数组长度不一致的情况下需要将数组里剩余的元素放在新建的数组中。



### 代码实现



我们来看一下归并排序算法的代码实现：

```swift
func _merge(leftPile: [Int], rightPile: [Int]) -> [Int] {
    
    var leftIndex = 0   //left pile index, start from 0
    var rightIndex = 0  //right pile index, start from 0
    
    var sortedPile = [Int]() //sorted pile, empty in the first place
    
    while leftIndex < leftPile.count && rightIndex < rightPile.count {
        
        //append the smaller value into sortedPile
        if leftPile[leftIndex] < rightPile[rightIndex] {
            
            sortedPile.append(leftPile[leftIndex])
            leftIndex += 1
            
        } else if leftPile[leftIndex] > rightPile[rightIndex] {
            
            sortedPile.append(rightPile[rightIndex])
            rightIndex += 1
            
        } else {
            
            //same value, append both of them and move the corresponding index
            sortedPile.append(leftPile[leftIndex])
            leftIndex += 1
            sortedPile.append(rightPile[rightIndex])
            rightIndex += 1
        }
    }
    
    
    //left pile is not empty
    while leftIndex < leftPile.count {
        sortedPile.append(leftPile[leftIndex])
        leftIndex += 1
    }
    
    //right pile is not empty
    while rightIndex < rightPile.count {
        sortedPile.append(rightPile[rightIndex])
        rightIndex += 1
    }
    

    return sortedPile
}
```

> 因为该函数是归并排序函数内部调用的函数，所以在函数名称的前面添加了下划线。仅仅是为了区分，并不是必须的。

从上面代码可以看出合并的实现逻辑：

- 新建空数组，初始化两个传入数组的index为0
- 两两比较两个数组index上的值，较小的放在新建数组里面并且index+1。
- 最后检查是否有剩余元素，如果有则添加到新建数组里面。



理解了合并的算法，下面我们看一下拆分的算法。拆分算法使用了递归：



```swift
func mergeSort(_ array: [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    let middleIndex = array.count / 2
    let leftArray = mergeSort(Array(array[0..<middleIndex]))             // recursively split left part of original array
    let rightArray = mergeSort(Array(array[middleIndex..<array.count]))  // recursively split right part of original array

    return _merge(leftPile: leftArray, rightPile: rightArray)             // merge left part and right part
}
```



我们可以看到``mergeSort``调用了自身，它的递归终止条件是``!(array.count >1)``，也就是说当数组元素个数 = 1的时候就会返回，会触发调用栈的出栈。



从这个递归函数的实现可以看到它的作用是不断以中心店拆分传入的数组。根据他的递归终止条件，当数组元素 > 1的时候，拆分会继续进行。而下面的合并函数只有在递归终止，开始出栈的时候才开始真正执行。也就是说在拆分结束后才开始进行合并，这样符合了上面笔者介绍的归并算法的实现过程。



上段文字需要反复体会。

为了更形象体现出归并排序的实现过程，可以在合并函数(``_merge``)内部添加log来验证上面的说法：

```swift
func _merge(leftPile: [Int], rightPile: [Int]) -> [Int] {
    
    print("\nmerge left pile:\(leftPile)  |  right pile:\(rightPile)")
    
    ...
    
    print("sorted pile：\(sortedPile)")
    return sortedPile
}
```



而且为了方便和上图作比较，初始数组可以取图中的``[3, 5, 9, 2, 7, 4, 8, 0]``。运行一下看看效果：

```
original array:
[3, 5, 9, 2, 7, 4, 8, 0]

merge sort...

merge left pile:[3]  |  right pile:[5]
sorted pile：[3, 5]

merge left pile:[9]  |  right pile:[2]
sorted pile：[2, 9]

merge left pile:[3, 5]  |  right pile:[2, 9]
sorted pile：[2, 3, 5, 9]

merge left pile:[7]  |  right pile:[4]
sorted pile：[4, 7]

merge left pile:[8]  |  right pile:[0]
sorted pile：[0, 8]

merge left pile:[4, 7]  |  right pile:[0, 8]
sorted pile：[0, 4, 7, 8]

merge left pile:[2, 3, 5, 9]  |  right pile:[0, 4, 7, 8]
sorted pile：[0, 2, 3, 4, 5, 7, 8, 9]
```



我们可以看到，拆分归并的操作是先处理原数组的左侧部分，然后处理原数组的右侧部分。这是为什么呢？



我们来看下最初函数是怎么调用的：

最开始我们调用函数：

```swift
func mergeSort(_ array: [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    let middleIndex = array.count / 2
    let leftArray = mergeSort(Array(array[0..<middleIndex]))             //1 
    let rightArray = mergeSort(Array(array[middleIndex..<array.count]))  //2

    return _merge(leftPile: leftArray, rightPile: rightArray)            //3
}
```

在//1这一行开始了递归，这个时候数组是原数组，元素个数是8，而调用mergeSort时原数组被拆分了一半，是4。而4>1，不满足递归终止的条件，继续递归，直到符合了终止条件（[3]）,递归开始返回。以为此时最初被拆分的是数组的左半部分，所以左半部分的拆分会逐步合并，最终得到了``[2,3,5,9]``。

同理，再回到了最初被拆分的数组的右半部分（上面代码段中的//2），也是和左测一样的拆分和归并，得到了右侧部分的归并结果：``[0,4,7,8``。

而此时的递归调用栈只有一个mergeSort函数了，mergeSort会进行最终的合并（上面代码段中的//3），调用``_merge``函数，得到了最终的结果：``[0, 2, 3, 4, 5, 7, 8, 9]``。



关于归并排序的性能：由于使用了分治和递归并且利用了一些其他的内存空间，所以其性能是高于上述介绍的所有排序的，不过前提是初始元素量不小的情况下。



我们可以将选择排序和归并排序做个比较：初始数组为长度500，最大值为500的随机数组：

```
selection sort...
selection sort time duration : 12.7s

merge sort...
merge sort time duration : 5.21s
```



可以看到归并排序的算法是优与选择排序的。

现在我们知道归并排序使用了分治思想而且使用了递归，能够高效地将数组排序。其实还有一个也是用分治思想和递归，但是却比归并排序还要优秀的算法 - 快速排序算法。



## 快速排序



快速排序算法被称之为20世纪十大算法之一，也是各大公司面试比较喜欢考察的算法。



### 算法讲解



快速排序的基本思想是：通过一趟排序将带排记录分割成独立的两部分，其中一部分记录的关键字均比另一部分记录的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序的目的。

> 上述文字摘自《大话数据结构》



它的实现步骤为：

1. 从数列中挑出一个元素（挑选的算法可以是随机，也可以作其他的优化），称为"基准"（pivot）。
2. 重新对数组进行排序：所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面，相同的放两边。
3. 递归地进行分区操作，继续把小于基准值元素的子数列和大于基准值元素的子数列排序。





从上面的描述可以看出，分区操作是快速排序中的核心算法。下面笔者结合实例来描述一下分区操作的过程。

首先拿到初始的数组：``[5,4,9,1,3,6,7,8,2]``

- 选择5作为pivot。
- 从剩下部分的两端开始：左侧1的标记为low，最右侧2的标记为high。
- 先看j：2 < 5 , 交换5和2，j不变 ：``[2,4,9,1,3,6,7,8,5]`` ；
- 再看i：2 < 5 , i ++ ；4 < 5, i++；9 > 5，交换 9 和 5，i不变``[2,4,5,1,3,6,7,8,9]``。





### 代码实现



#### 使用Swift的filter函数

因为在Swift中有一个数组的filter函数可以找出数组中符合某范围的一些数值，所以笔者先介绍一个会用该函数的简单的快速排序的实现：

```swift
func quickSort0<T: Comparable>(_ array: [T]) -> [T] {
    
    guard array.count > 1 else { return array }
    
    let pivot = array[array.count/2]
    let less = array.filter { $0 < pivot }
    let greater = array.filter { $0 > pivot }
    
    return quickSort0(less) + quickSort0(greater)
}
```



不难看出这里面使用了递归：选中pivot以后，将数组分成了两个部分，最后将它们合并在一起。虽然这里面使用了Swift里面内置的函数来找出符合这两个个部分的元素，但是读者可以通过这个例子更好地理解快速排序的实现方式。



#### 使用取index = 0 的partition函数



除了使用swift内置的filter函数，当然我们也可以自己实现分区的功能，通常使用的是自定义的partition函数。

```swift
func _partition(_ array: inout [Int], low: Int, high: Int) -> Int{

    var low       = low
    var high      = high

    let pivotValue = array[low]

    while low < high {

        while low < high && array[high] >= pivotValue {
            high -= 1
        }
        array[low] = array[high]
        
        while low < high && array[low] <= pivotValue {
            low += 1
        }
        array[high] = array[low]
    }

    array[low] = pivotValue
  
    return low
}
```

从代码实现可以看出，最初在这里选择的pivotValue是当前数组的第一个元素。

然后从数组的最右侧的index逐渐向左侧移动，如果值大于pivotValue，那么index-1；否则直接将high与low位置上的元素调换；同样左侧的index也是类似的操作。

该函数执行的最终效果就是将最初的array按照选定的pivotValue前后划分。



那么``_partition``如何使用呢？

```swift
func quickSort1(_ array: inout [Int], low: Int, high: Int){
  
    guard array.count > 1 else { return }
    
    if low < high {        
        let pivotIndex = _partition(&array, low: low, high: high)
        quickSort1(&array, low: low, high: pivotIndex - 1)
        quickSort1(&array, low: pivotIndex + 1, high: high)
    }
    
}
```

外层调用的``quickSort1``是一个递归函数，不断地进行分区操作，最终得到排好序的结果。



我们将上面实现的归并排序，使用swift内置函数的快速排序，以及自定义partition函数的快速排序的性能作对比：

```swift
merge sort...
merge sort time duration : 4.85s

quick sort...
quick sort0 time duration : 984ms //swift filter function
quick sort1 time duration : 2.64s //custom partition
```



上面的测试用例是选择随机数组的，我们看一下测试用例为元素个数一致的基本有序的数组试一下：

```
merge sort...
merge sort time duration : 4.88s

quick sort...
quick sort0 time duration : 921ms
quick sort1 time duration : 11.3s
```



虽然元素个数一致，但是性能却差了很多，是为什么呢？因为我们在分区的时候，pivot的index强制为第一个。那么如果这个第一个元素的值本来就非常小，那么就会造成分区不均的情况（前重后轻），而且由于是迭代操作，每次分区都会造成分区不均，导致性能直线下降。所以有一个相对合理的方案就是在选取pivot的index的时候随机选取。



#### 使用随机选择pivotValue的partition函数



实现方法肯简单，只需在分区函数里将pivotValue的index随机生成即可：

```swift
func _partitionRandom(_ array: inout [Int], low: Int, high: Int) -> Int{
    
    let x      = UInt32(low)
    let y      = UInt32(high)
    
    let pivotIndex = Int(arc4random() % (y - x)) + Int(x)
    let pivotValue = array[pivotIndex] 
    
    ...
}
```

现在用一个数组长度和上面的测试用例一致的基本有序的数组来测试一下随机选取pivotValue的算法：

```
merge sort...
merge sort time duration : 4.73s

quick sort...
quick sort0 time duration : 866ms
quick sort1 time duration : 15.1s  //fixed pivote index
quick sort2 time duration : 4.28s  //random pivote index
```

我们可以看到当随机抽取pivot的index的时候，其运行速度速度是上面方案的3倍。



现在我们知道了3种快速排序的实现，都是根据pivotValue将原数组一分为二。但是如果数组中有大量的重复的元素，而且pivotValue很有可能落在这些元素里，那么显然上面这些算法对于这些可能出现多次于pivotValue重复的情况没有单独做处理。而为了很好解决存在与pivot值相等的元素很多的数组的排序，使用三路排序算法会比较有效果。



#### 三路快速排序



三路快速排序将大于，等于，小于pivotValue的元素都区分开，我们看一下具体的实现。先看一下partition函数的实现：

```swift
func swap(_ arr: inout [Int],  _ j: Int, _ k: Int) {
    
    guard j != k else {
        return;
    }
    
    let temp = arr[j]
    arr[j] = arr[k]
    arr[k] = temp
}



func quickSort3W(_ array: inout [Int], low: Int, high: Int) {
    
    if high <= low { return }
    
    var lt = low       // arr[low+1...lt] < v
    var gt = high + 1  // arr[gt...high] > v
    var i  = low + 1   // arr[lt+1...i) == v
    
    let pivoteIndex = low
    let pivoteValue = array[pivoteIndex]
    
    while i < gt {
        
        if array[i] < pivoteValue {
          
            swap(&array, i, lt + 1)
            i += 1
            lt += 1
           
        }else if pivoteValue < array[i]{
       
            swap(&array, i, gt - 1)
            gt -= 1
            
        }else {
            i += 1
        }
    }
    
    swap(&array, low, lt)
    quickSort3W(&array, low: low, high: lt - 1)
    quickSort3W(&array, low: gt, high: high)
    
    
}


func quickSort3(_ array: inout [Int] ){
    
    quickSort3W(&array, low: 0, high: array.count - 1)
    
}
```

主要看``quickSort3W``方法，这里将数组分成了三个区间，分别是大于，等于，小于pivote的值，对有大量重复元素的数组做了比较好的处理。



我们生成一个元素数量为500，最大值为5的随机数组看一下这些快速排序算法的性能：

```
quick sort1 time duration : 6.19s //fixed pivote index
quick sort2 time duration : 8.1s  //random pivote index
quick sort3 time duration : 4.81s //quick sort 3 way 
```



可以看到三路快速排序（quick sort 3 way）在处理大量重复元素的数组的表现最好。



对于三路快速排序，我们也可以使用Swift内置的filter函数来实现:

```swift
func quicksort4(_ array: [Int]) -> [Int] {
    
    guard array.count > 1 else { return array }
    
    let pivot = array[array.count/2]
    let less = array.filter { $0 < pivot }
    let equal = array.filter { $0 == pivot }
    let greater = array.filter { $0 > pivot }
    
    return quicksort4(less) + equal + quicksort4(greater)
}
```



以上，介绍完了快速排序在Swift中的5中实现方式。



# 最后的话



## 总结

本文讲解了算法的一些基本概念以及结合了Swift代码的实现讲解了冒泡排序，选择排序，插入排序，归并排序，快速排序。相信认真阅读本文的读者能对这些算法有进一步的了解。



## 关于算法学习的思考

关于算法的学习，笔者有一些思考想分享出来，也有可能有不对的地方，但笔者觉得有必要在这里说出来，希望可以引发读者的思考：

![](https://user-gold-cdn.xitu.io/2018/2/8/1617175e884d80a7?w=580&h=378&f=png&s=19270)



上图的Question是指问题；Mind是指想法，或者解决问题的思路；Code是指代码实现。

在阅读资料或书籍的算法学习过程，往往是按照图中1，2，3这些实线的路径进行的：

- 路径1：给出一个既定的问题后，马上给出解题策略
- 路径2：给出一个既定的问题后，马上给出算法实现
- 路径3：给出一个算法实现后，马上告诉你这些实现代码的意思



这些路径在算法的学习中虽然也是必不可少的，但是很容易给人一个错觉，这个错觉就是“**我已经学会了这个算法了**”。但是，仅仅是通过这些路径，对于真正理解算法，和今后对算法的应用还是远远不够的，原因是：

- 今后遇到的问题，几乎不可能与现在学习的问题一模一样，所以应该知其所以然，将问题本身抽象出来，达到触类旁通，举一反三。
- 有了一个新想法，如果没有足够的代码实现经验，很难以非常合理的方式用代码将其实现出来。所以应该增强将想法转化为代码的能力。



上面所说的两点的第一点，对应的是上图的路径4：给定一个策略或是设计，要思考这个策略或是设计是解决什么样的问题的，这样也就理解了这个策略或是设计的意义在哪里；而第二点对应的是上图中的路径5：怎样根据一个给定的策略来正确地，合理地用代码地实现出来；而上图中的路径6，笔者觉得也很重要：给定一份解决问题的代码，是否可以想到它所对应的问题是什么。



综上所述，笔者认为对于算法的学习，需要经常反复在问题，策略以及代码之间反复思考，这样才能真正地达到学以致用。



因为笔者也刚刚接触这一领域的知识，所以难免会在有些地方的表述有不妥当的地方，还需读者多多给出意见和建议。



------



**Swift代码**

本篇中出现的代码已经放在GitHub仓库中：
- 算法基础部分：[Algorithm Introduction](https://github.com/knightsj/data-structure-and-algorithm-in-Swift/tree/master/%5B3%5D.Algorithm)
- 排序算法部分：[Sort Algorithms](https://github.com/knightsj/data-structure-and-algorithm-in-Swift/tree/master/%5B4%5D.Sort%20algorithms)




**参考文献&网站**


[维基百科：算法](https://zh.wikipedia.org/wiki/%E7%AE%97%E6%B3%95)

《大话数据结构》

《数据结构与算法分析：C语言描述》




**下篇预告**

下篇会介绍堆这个数据结构以及堆排序算法。



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


