---
title: 数据结构 & 算法 in Swift （一）：Swift基础和数据结构
tags: [iOS,Swift,Data Structure,Algorithm]
categories: Data Structure & Algorithm
---



![](http://oih3a9o4n.bkt.clouddn.com/da_header_2.png)






# 写在前面

从本文标题中的序号可以看出，本文是一个连载的开篇。

而且这个连载的标题是：数据结构 & 算法 in Swift。从这个连载的标题中可以看出，笔者分享的是使用Swift语言来实现所学的的数据结构和算法的知识。这里面需要解释两点：



## 第一：为什么学习数据结构和算法？

**学习通用性知识，突破技能瓶颈**：笔者做iOS开发也有两年了，这期间通过从项目，第三方源码，相关应用类的编程书籍提高了些技术水平。而作为没学过数据结构和算法的非科班大军中的一员，这些知识始终是绕不过去的。因为对此类知识的掌握程度会对今后编程技能的提高有着无可估量的影响，所以就决定学习了。



## 第二：为什么用Swift语言来实现？

1. **选择哪个语言并不重要，重要的是数据结构和算法本身的理解**：通过两个星期的学习，如今笔者已经可以使用Swift语言来实现几种数据结构和算法了，但我相信如果我使用C语言或者Objective-C语言的话会学得更快些，因为在实现的时候由于对该语言的不熟悉导致在实现过程中踩了不少坑。不过可以反过来思考：如果我可以使用Swift来实现这些，那么我今后用C，Objective-C，甚至是Java就容易多了，再加上我还顺便学习了Swift不是么？

2. **如今Swift的势头还在上涨**：笔者已经观察到很多新的库，教学都使用了Swift语言。而且听说一些面试的朋友在面试过程中多少有问过Swift相关的知识，一些公司的新项目也有用Swift写了。

   ​

基于上面这些原因，在今年年初把**数据结构，算法和Swift**的学习提上了日程，并且计划以连载的形式把学习过程中的笔记和知识分享出来。



该系列的**最佳受众**是那些已经会Swift，但是对数据结构和算法还没有过多接触过的iOS开发者。其次是那些不会Swift也不会数据结构和算法的iOS开发者，毕竟Swift是大势所趋。



不过对于那些非iOS开发者来说也同样适合，因为还是那句话：**重点不在于使用哪种语言，而是数据结构和算法本身**。除了第一篇会讲解一些在这个系列文章会使用到的Swift基础语法以外，后续的文章我会逐渐**弱化对Swift语言的讲解，将重点放在数据结构和算法这里**。而且后续我还会不断增加其他语言的实现（Java语言是肯定要加的，其他的语言还待定）。


好了，背景介绍完了，现在正式开始：



作为该系列的开篇，本文分为两个部分：

1. **Swift语法基**础：讲解一下后续连载中讲到的数据结构和算法所涉及到的Swift语法知识（并不是很全面，也不是很深入，但是在实现数据结构和算法这块应该是够了）。
2. **数据结构**：简单介绍数据结构和算法的相关概念，以及用Swift来实现几个简单的数据结构（链表，栈，队列）

> 注：该系列涉及到的Swift语法最低基于Swift4.0。

<!-- more -->


# Swift 语法基础



Swift语法基础从以下几点来展开：



1. 循环语句
2. 泛型
3. guard
4. 函数
5. 集合



## 循环语句



### 循环条件的开闭区间



Swift将循环的开闭区间做了语法上的简化：



#### 闭区间：

```swift
for index in 1...5 {
    print("index: \(index)")
}
// index : 1
// index : 2
// index : 3
// index : 4
// index : 5
```



#### 半开闭区间：

```swift
for index in 1..<5 {
    print("index: \(index)")
}
// index : 1
// index : 2
// index : 3
// index : 4
```





### 循环的升序与降序



上面两个例子都是升序的（index从小到大），我们来看一下降序的写法：

```swift
for index in (1..<5).reversed() {
    print("index: \(index)")
}
// index : 4
// index : 3
// index : 2
// index : 1
```



> 降序的应用可以在下篇的冒泡排序算法中可以看到。





## 泛型



使用泛型可以定义一些可复用的函数或类型，Swift中的Array和Dictionary都是泛型的集合。

为了体现出泛型的意义，下面举一个例子来说明一下：

> 实现这样一个功能:将传入该函数的两个参数互换。



整型的交换：

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let tmp = a
    a = b
    b = tmp
}
```



字符串的交换：

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let tmp = a
    a = b
    b = tmp
}
```



浮点型的交换：

```swift
func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let tmp = a
    a = b
    b = tmp
}
```



上面这三种情况的实现部分其实都是一样的，但仅仅是因为传入类型的不一致，导致对于不同的类型还要定义一个新的函数。所以如果类型有很多的话，定义的新函数也会很多，这样显然是不够优雅的。



此类问题可以使用泛型来解决：

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let tmp = a
    a = b
    b = tmp
}
```



上面函数中的T是泛型的固定写法，可以理解为“所有类型”。这样一来，我们可以传入任何相同的类型来作交换了。



泛型还有其他比较强大的功能，由于在后续的数据结构和算法的讲解里面可能不会涉及到，所以在这里先不赘述了。有兴趣的朋友可以参考官方文档：[Swift：Generics](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html#//apple_ref/doc/uid/TP40014097-CH26-ID179)







## guard



> guard是 swift 2.0推出的新的判断语句的用法。
>
> 与if语句相同的是，guard也是基于一个表达式的布尔值去判断一段代码是否该被执行。与if语句不同的是，guard只有在条件不满足的时候才会执行这段代码。你可以把guard近似的看做是Assert，但是你可以优雅的退出而非崩溃



使用guard语法，可以先对每个条件逐一做检查，如果不符合条件判断就退出（或者进行其他某些操作）。这就会让人容易看出来什么条件会让这个函数退出（或者进行其他某些操作）。



可以用一个例子来分别使用if和guard来实现，体会二者的区别：



### 使用if-else

```swift
//money:    holding moneny (用户持有的钱数)
//price:    product price  (商品的价格)
//capacity: bag capacity   (用户用来装商品的袋子容量)
//volume:   product size   (商品的大小)

func buying1( money: Int , price: Int , capacity: Int , volume: Int){
    
    if money >= price{
        
        if capacity >= volume{
            
            print("Start buying...")
            print("\(money-price) money left after buying.")
            print("\(capacity-volume) capacity left after buying.")
        
        }else{
            
            print("No enough capacity")
        }
    
    }else{
        
        print("No enough money")
        
    }
}
```



从上面的逻辑可以看出，当同时满足：

1. 用户的钱数>商品价格
2. 用户用来装商品的袋子容量>商品的大小

这两个情况的时候，购买才会进行，其他所有情况都无法引发购买。



对于大多数习惯使用if-else的朋友来说，上面的代码立即起来并没有难度，但是相同的逻辑，我们看一下使用guard之后的效果：



### 使用guard

```swift
func buying2( money: Int , price: Int , capacity: Int , volume: Int){
    
    guard money >= price else{
        print("No enough money")
        return
    }
    
    guard capacity >= volume else{
        print("No enough capacity")
        return
    }
    
    print("Start buying...")
    print("\(money-price) money after buying.")
    print("\(capacity-volume) capacity left after buying.")
}
```



从上面的实现可以看出：

- 使用guard以后，将``money < price``和``capacity < volume`` 这两个情况首先排除掉并填上了相应的处理代码。
- 在两个guard下面才是真正正确逻辑后的处理代码。

因此通过两个guard判断的语句，我们知道该函数所处理的正确逻辑是什么，非常清晰。





## 函数



因为后续的数据结构和算法的讲解是离不开函数的使用的，所以在这里简单介绍一下Swift中函数的使用。



- 无返回值的函数
- 有返回值的函数
- 省略函数的外部参数名
- 值传递和引用传递

### 无返回值的函数

```swift
func log(message: String) {
    print("log: \(message)!")
}

log(message: "memory warning")
// output: log: memory warning!
```



### 有返回值的函数

```swift
func logString(string: String) -> String {
    return "log: " + string
}

let logStr = logString(string: "memory warning!")
print("\(logStr)")
// output: log: memory warning!
```



### 省略函数外部参数名



通过在函数形参前面加上``_``,可以起到在调用时省略外部参数的作用：

```swift
func logMessage(_ message: String) {
    print("log: \(message)!")
}

logMessage("memory warning")
// output: log: memory warning!
```



再来看一下两个参数的情况：

```swift
func addInt(_ a : Int ,_ b : Int){
    print("sum is \(a + b)")
}

addInt(3, 4)
//output : sum is 7
```



### 值传递和引用传递

Swift中，struct是按值传递，class是按引用传递。数组和字典在Swift里是属于struct，所以需要如果在一个函数里要修改传入的数组，需要做特殊处理：



```swift
var originalArr = [2,1,3]

func removeLastInArray(_ array: inout [Int]){
    array.removeLast()
}
print("\n============ before removing: \(originalArr)")
//[2, 1, 3]

removeLastInArray(&originalArr)

print("============ after   removing: \(originalArr)")
//[2, 1]
```



在这里使用的``inout``关键字就是将传入的数组改为引用传递了。





## 集合



Swift里的集合类型有：数组，集合，字典，下面来分别讲一下。

> 这三种类型都支持泛型，也就是说里面的元素可以是整数，字符串，浮点等等。



### 数组



> Swift’s `Array` type is bridged to Foundation’s `NSArray` class.



#### 可变数组与不可变数组

```swift
// immutable array
let immutableNumbers: [Int] = [1, 3, 5, 4, 4, 1]

// mutable array
var mutableNumbers : [Int] = [2, 1, 5, 4, 1, 3]
```



> Swift中可以用``let``和``var``来分别声明可变和不可变数组：数组的添加删除等操作只能作用于可变数组。



#### 数组的遍历

```swift
// iteration 1
for value in mutableNumbers {
    if let index = mutableNumbers.index(of: value) {
        print("Index of \(value) is \(index)")
    }
}

// iteration 2
mutableNumbers.forEach { value in
    if let index = mutableNumbers.index(of: value) {
        print("Index of \(value) is \(index)")
    }
}

// iteration 3
for (index, value) in mutableNumbers.enumerated() {
    print("Item \(index + 1): \(value)")
}

```





#### 数组的操作

```swift
mutableNumbers.append(11)
// Output: [2, 1, 5, 4, 1, 3, 11]

mutableNumbers.insert(42, at: 4)
// Output: [2, 1, 5, 4, 42, 1, 3, 11]

mutableNumbers.swapAt(0, 1)
// Output: [1, 2, 5, 4, 42, 1, 3, 11]

mutableNumbers.remove(at: 1)
// Output: [2, 5, 4, 42, 1, 3, 11]

mutableNumbers.removeFirst()
// Output: [5, 4, 42, 1, 3, 11]

mutableNumbers.removeLast()
// Output: [5, 4, 42, 1, 3]

mutableNumbers.removeAll()
//[]
```



> append函数的作用是在数组的末尾添加元素
>
> swapAt函数的作用是交换在传入的两个index上的元素，该方法在下篇的排序算法中使用得非常频繁。





### 集合



> Swift’s `Set` type is bridged to Foundation’s `NSSet` class.



#### 集合的无序性，值的唯一性

关于集合与数组的区别，除了数组有序，集合无序以外，数组内部的元素的数值可以不是唯一的；但是集合里元素的数值必须是唯一的，如果有重复的数值会算作是一个：

```swift
//value in set is unique
let onesSet: Set = [1, 1, 1, 1]
print(onesSet)
// Output: [1]

let onesArray: Array = [1, 1, 1, 1]
print(onesArray)
// Output: [1, 1, 1, 1]
```



#### 集合的遍历

```swift
let numbersSet: Set = [1, 2, 3, 4, 5]
print(numbersSet)
// Output: undefined order, e.g. [5, 2, 3, 1, 4]


// iteration 1
for value in numbersSet {
    print(value)
}
// output is in undefined order


// iteration 2
numbersSet.forEach { value in
    print(value)
}
// output is in undefined order
```



#### 集合的操作

```swift
var mutableStringSet: Set = ["One", "Two", "Three"]
let item = "Two"

//contains
if mutableStringSet.contains(item) {
    print("\(item) found in the set")
} else {
    print("\(item) not found in the set")
}

//isEmpty
let strings = Set<String>()
if strings.isEmpty {
    print("Set is empty")
}

//count
let emptyStrings = Set<String>()
if emptyStrings.count == 0 {
    print("Set has no elements")
}

//insert
mutableStringSet.insert("Four")


//remove 1
mutableStringSet.remove("Three")

//remove 2
if let removedElement = mutableStringSet.remove("Six") {
    print("\(removedElement) was removed from the Set")
} else {
    print("Six is not found in the Set")
}

//removeAll()
mutableStringSet.removeAll()
// []
```





### 字典



> A dictionary `Key` type must conform to the `Hashable` protocol, like a set’s value type.



#### 字典的声明

```swift
//empty dictionary
var dayOfWeek = Dictionary<Int, String>()
var dayOfWeek2 = [Int: String]()

//not empty dictionary
var dayOfWeek3: [Int: String] = [0: "Sun", 1: "Mon", 2: "Tue"]
print(dayOfWeek3)
//output:[2: "Tue", 0: "Sun", 1: "Mon"]
```

> 可以看到字典的键值对也是无序的，它与声明时的顺序不一定一致。



#### 字典的遍历

```swift
// iteration 1
for (key, value) in dayOfWeek {
    print("\(key): \(value)")
}

// iteration 2
for key in dayOfWeek.keys {
    print(key)
}

// iteration 3
for value in dayOfWeek.values {
    print(value)
}
```



#### 字典的操作

```swift
// find value
dayOfWeek = [0: "Sun", 1: "Mon", 2: "Tue"]
if let day = dayOfWeek[2] {
    print(day)
}

// addValue 1
dayOfWeek[3] = "Wed"
print(dayOfWeek)
// Prints: [2: "Tue", 0: "Sun", 1: "Mon", 3: "Wed"]

// updateValue 1
dayOfWeek[2] = "Mardi"
print(dayOfWeek)
// Prints: [2: "Mardi", 0: "Sun", 1: "Mon", 3: "Wed"]

// updateValue 2
dayOfWeek.updateValue("Tue", forKey: 2)
print(dayOfWeek)
// Prints: [2: "Tue", 0: "Sun", 1: "Mon", 3: "Wed"]

// removeValue 1
dayOfWeek[1] = nil
print(dayOfWeek)
// Prints: [2: "Tue", 0: "Sun", 3: "Wed"]

// removeValue 2
dayOfWeek.removeValue(forKey: 2)
print(dayOfWeek)
// Prints: [0: "Sun", 3: "Wed"]

// removeAll
dayOfWeek.removeAll()

print(dayOfWeek)
// Output: [:]
```



> 可以看到从字典里面删除某个键值对有两个方法：
>
> 1. 使用``removeValue``方法并传入要删除的键值对里的键。
> 2. 将字典取下标之后将nil赋给它。









# 数据结构



这一部分内容主要是对连载的后续文章作铺垫，让大家对数据结构先有一个基本的认识，因此在概念上不会深入讲解。该部分由以下三点展开：

- 数据结构的基本概念

- 抽象数据类型

- 链表，栈和队列的实现

  ​



## 概念

 

首先我们来看一下数据结构的概念：

> 数据结构：是相互之间存在一种或多种特定关系的数据元素的集合。



由数据结构这个词汇的本身（数据的结构）以及它的概念可以看出，它的重点在于“结构”和“关系”。所以说，数据是何种数据并不重要，重要的是这些数据**是如何联系起来**的。



而这些联系，可以从两个维度来展开：

1. 逻辑结构：指数据对象中元素之间的相互关系。
2. 物理结构：指数据的逻辑结构在计算机中的存储形式。

可以看出，逻辑结构是抽象的联系，而物理结构是实际在计算机内存里的具体联系。那么它们自己又细分为哪些结构呢？



逻辑结构：

- 集合结构：集合结构中的数据元素除了同属于一个集合外，它们之间没有其他关系。
- 线性结构：线性结构中的数据元素之间是一对一的关系。
- 树形结构：数据结构中的元素存在一对多的相互关系。
- 图形结构：数据结构中的元素存在多对多的相互关系。

物理结构：

- 顺序存储结构：把数据元素粗放在地址连续的存储单元里，其数据间的逻辑关系和物理关系是一致的（数组）。
- 链式存储结构：把数据元素存放在任意的存储单元里，这组存储单元可以是连续的，也可以是不连续的。

为了便于记忆，用思维导图总结一下上面所说的：



![](https://user-gold-cdn.xitu.io/2018/1/31/16147cdb8da77739?w=501&h=337&f=png&s=29900)





而通过结合这两个维度中的某个结构，可以定义出来一个实际的数据结构的实现：



比如线性表就是线性结构的一种实现：

- 顺序存储结构的线性表就是**数组**：它的内存分布是连续的，元素之间可以通过内存地址来做关联；
- 链式存储结构的线性表就是**链表**：它的内存分布可以是不连续的，元素之间通过指针来做关联：
  - 如果每个元素（在链表中称作节点）只持有指向后面节点的指针，那此链表就是单链表。
  - 如果每个元素（在链表中称作节点）持有指向前后节点的两个指针，那此链表就是双链表。

为什么会有链表这么麻烦的东西？像数组这样，所有内存地址都是连续的不是很方便么？既生瑜何生亮呢？



对于获取元素（节点）这一操作，使用数组这个数据结构确实非常方便：因为所有元素在内存中是连续的，所以只需要知道数组中第一个元素的地址以及要获取元素的index就能算出该index内存的地址，一步到位非常方便。



但是对于向数组中某个index中插入新的元素的操作恐怕就没有这么方便了：恰恰是因为数组中所有元素的内存是连续的，所以如果想在中间插入一个新的元素，那么这个位置后面的所有元素都要后移，显然是非常低效的。如果插在数组尾部还好，如果插在第一位的话成本就太高了。



而如果使用链表，只要把要插入到的index前后节点的指针赋给这个新的节点就可以了，不需要移动原有节点在内存中的位置。



> 关于链表的这种插入操作会在后面用代码的形式体现出来。



既然有这么多的数据结构，那么有没有一个标准的格式来将这些特定的数据结构（也可以说是数学模型）抽象出来呢？答案是肯定的，它就是我们下一节要讲的**抽象数据类型**。



## 抽象数据类型



首先来看一下抽象数据类型的概念，摘自《大话数据结构》:

> 抽象数据类型（Abstract Data Type，ADT）：是指一个数学模型及定义在该模型上的一组操作。



需要注意的是：抽象数据类型的定义仅仅取决于它的一组**逻辑特性**，而与其在计算机内部如何表示和实现没有关系。而且，抽象数据类型不仅仅指那些已经定义并实现的数据类型，还尅是计算机编程者**自己定义的数据类型**。



我们看一下数据类型的标准格式：

```
ADT 抽象数据类型名

Data
   数据元素之间逻辑关系的定义
   
Operation
   操作1
      初始条件
      操作结果描述
 
   操作2
      初始条件
      操作结果描述
      
   操作n

endADT
```



其实看上去和面向对象编程里的类的定义相似：

- 可以把抽象数据类型的Data 和 类的成员变量联系起来。
- 可以把抽象数据类型的操作和类的函数联系起来。

简单来说，抽象数据类型描述了一个数据模型所使用的数据和数据之间的逻辑关系，以及它可以执行的一些操作。因此，如果知道了一个数学模型的抽象数据类型，那么在真正接触数学模型的实现（代码）之前，就可以对该数学模型能做的事情有一个大致的了解。



下一章笔者会介绍链表，栈和队列这三个数学模型，在讲解每个数学模型的实现之前都会给出它们各自的抽象数据类型，让读者可以先对当前数学模型有个大致的了解。



> 注意：书本文归纳的所有抽象数据类型是笔者自己根据网上资料和相关书籍而定下来的，所以严格来说它们并不是“最官方”的抽象数据类型。读者也可以参考网上的资料或是相关书籍，结合自己的思考来定义自己对着三个数据模型的抽象数据类型。



## 链表，栈和队列的实现



通过上一节的介绍，我们知道了数据结构的概念以及分类，还知道了不同的数据结构在不同的场景下会发挥不同的优势，我们要根据实际的场景来选择合适的数据结构。



下面就来介绍几种在实际应用中使用的比较多的数学模型：

- 链表
- 栈
- 队列



### 链表（Linked list）



说到链表就不得不提线性表这一数据结构，在介绍链表之前，首先看一下线性表的定义：

> 线性表：零个或多个数据元素的有限序列。



而根据物理结构的不同，线性表有两种具体的实现方式：

- 线性表的顺序存储结构：线性表的数据元素是被一段**地址连续**的存储单存储起来的。
- 线性表的链式存储结构: 线性表的数据元素是被用一组**连续或不连续**的存储单元存储起来的，这些元素通过指针来作为逻辑上的连接。

> 注：上面两个概念是笔者用自己的话总结出来的。



在这里，线性表的顺序存储结构的实现就是我们熟悉的数组；而线性表的链式存储结构的实现就是笔者即将要介绍的链表。





#### 链表的定义



相信对于读完上一节的朋友来说，应该对链表有一个比较清晰的认识了。关于链表的定义有很多不同的版本，笔者个人比较喜欢百度百科里的定义：

> 链表是一种物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。



而且由于数据元素所持有的指针个数和链接特性可以将链表分为：



- 单向链表：单向链表的链接方向是单向的，其中每个结点都有指针成员变量指向列表中的下一个结点；
- 双向链表：双向链表的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点，它的链接方向是双向的。
- 循环链表：循环链表是另一种形式的链式存贮结构。它的特点是表中最后一个结点的指针域指向头结点，整个链表形成一个环。

笔者从中挑选出双向链表来进行讲解，它的难度适中，而且能够很好地让读者体会出链表的优势。



#### 双向链表的抽象数据类型



因为节点是链表的基本组成单元，所以想要实现链表，必须先要介绍链表的组成部分-节点。



节点：

```
ADT 节点(node)

Data
  value:持有的数据

Operation
   init:初始化
   previous:指向上一节点的指针
   next:指向下一节点的指针
   
endADT
```



再来看一下链表的抽象数据类型：

```
ADT 链表（linked list）

Data
  linked list:持有的线性表

Operation
   init:初始化
   count:持有节点总个数
   isEmpty:是否为空
   first:头节点
   last:尾节点
   node:传入index返回节点
   insert:插入node到指定index
   insertToHead:插入节点到表头
   appendToTail:插入节点到表尾
   removeAll:移除所有节点
   remove:移除传入的节点
   removeAt:移除传入index的节点
   
endADT
```





#### 双向链表的实现



##### 节点

```swift
public class LinkedListNode<T> {
    
    //value of a node
    var value: T
  
   //pointer to previous node
    weak var previous: LinkedListNode?
  
    //pointer to next node
    var next: LinkedListNode?
  
    
    //init
    public init(value: T) {
        self.value = value
    }
}
```



再来看一下链表的实现：

因为整个链表的插入，删除等操作比较多，整个链表的定义超过了200行代码，所以为了看着方便一点，在这里来分段说明一下。



首先看一下链表的成员变量:



##### 成员变量

```swift
public class LinkedList<T> {
    
    public typealias Node = LinkedListNode<T>
    
    //if empty
    public var isEmpty: Bool {
        return head == nil
    }
    
    //total count of nodes
    public var count: Int {
      
        guard var node = head else {
            return 0
        }
        
        var count = 1
        while let next = node.next {
            node = next
            count += 1
        }
        return count
    }
    
    //pointer to the first node, private
    private var head: Node?
    
    //pointer to the first node, public
    public var first: Node? {
        return head
    }
    
    //pointer to the last node
    public var last: Node? {
        
        guard var node = head else {
            return nil
        }
        
        //until node.next is nil
        while let next = node.next {
            node = next
        }
        return node
    }
    
    ...
  
}
```



相信看上面的命名以及注释大家可以对链表的成员变量有个初步的理解，这里面需要说三点：

1. ``typealias``是用来重新为已经存在的类型命名的：这里用``Node``代替了``LinkedListNode<T>``（节点类型），降低了不少阅读代码的成本。
2. 在获取``count``和``last``的实现，都先判断了``head``这个指针是否为nil，如果是则判定为空链表，自然也就不存在节点个数和最后的节点对象了。
3. 同样地，也是在获取``count``和``last``的实现里，使用了``while``控制语句来判断node.next节点是否存在：如果存在，则继续+1或者继续往下寻找，直到node.next为nil时才停止。在这里我们可以看到链表的寻址方式：是通过头结点开始，以节点的.next指针来寻找下一个节点的。而且作为链表的尾节点，它的.next指针不指向任何对象，因为它本来就是链表的最后一项。



> 最下方的…代表即将在下面介绍的一些函数，这些函数都定义在的``LinkedList``这个class里面。





##### 获取index上node

```swift
//get node of index
public func node(atIndex index: Int) -> Node? {

    if index == 0 {
        //head node
        return head!

    } else {

        var node = head!.next

        guard index < count else {
            return nil;
        }

        for _ in 1..<index {
            // go on finding by .next
            node = node?.next
            if node == nil {
                break
            }
        }

        return node!
    }
}
```

注意在这里返回的node是可以为nil的，而且在这里可以看出来，链表在寻找特定node的时候，是根据节点的.next指针来一个一个寻找的。这个与顺序存储结构的数组是不同的，在后面我会重点讲解一下这二者的不同。



##### 插入节点

```swift
//insert node to last index
public func appendToTail(value: T) {

    let newNode = Node(value: value)

    if let lastNode = last {

        //update last node: newNode becomes new last node;
        //the previous last node becomes the second-last node
        newNode.previous = lastNode
        lastNode.next = newNode

    } else {

        //blank linked list
        head = newNode
    }
}

//insert node to index 0
public func insertToHead(value: T) {

    let newHead = Node(value: value)

    if head == nil {
		//blank linked list
        head = newHead

    }else {

        newHead.next = head
        head?.previous = newHead
        head = newHead

    }
}

//insert node in specific index
public func insert(_ node: Node, atIndex index: Int) {

    if index < 0 {
        print("invalid input index")
        return
    }

    let newNode = node

    if count == 0 {

        head = newNode

    }else {

        if index == 0 {

            newNode.next = head
            head?.previous = newNode
            head = newNode

        } else {

            if index > count {

                print("out of range")
                return

            }

            let prev = self.node(atIndex: index-1)
            let next = prev?.next

            newNode.previous = prev
            newNode.next = prev?.next
            prev?.next = newNode
            next?.previous = newNode
        }

    }
}
```



链表的插入节点的操作分为三种，按照从上到下的顺序依次是：

1. 在头部插入
2. 在尾部插入
3. 指定index插入

需要注意的是

- 在前两种插入函数中，需要先判断该链表是否是空的，如果是，则要将链表的该节点赋给链表的``head``指针。
- 在第三种插入函数中，还是先判断该链表是否是空的，如果是，则无论index是多少(只要不小于0)，都插在链表的头部。如果不是空的，再判断index是否为0，如果是，则直接插在头部；如果index不为0，则判断index是否大于count，如果是，则无法插入；如果不是，则获取插入位置的前后节点进行重连。

> 在这里判断链表为空链表后的处理是笔者自己加上去的，笔者在网上的资料里没有看到过。大家不必纠结于这种处理方式，毕竟链表操作的重点在于前后节点的重连。





##### 移除节点

```swift
//removing all nodes
public func removeAll() {
    head = nil
}

//remove the last node
public func removeLast() -> T? {

    guard !isEmpty else {
        return nil
    }

    return remove(node: last!)
}

//remove a node by it's refrence
public func remove(node: Node) -> T? {

    guard head != nil else {
        print("linked list is empty")
        return nil
    }

    let prev = node.previous
    let next = node.next

    if let prev = prev {
        prev.next = next
    } else {
        head = next
    }

    next?.previous = prev

    node.previous = nil
    node.next = nil
    return node.value
}


//remove a node by it's index
public func removeAt(_ index: Int) -> T? {

    guard head != nil else {
        print("linked list is empty")
        return nil
    }

    let node = self.node(atIndex: index)
    guard node != nil else {
        return nil
    }
    return remove(node: node!)
}
```



- 如果要移除链表上所有节点，只需要将head指针置空就可以了，因为它是所有节点的“源头”，是链表寻址的第一个节点。
- 在持有某个节点的指针的时候可以指定链表来移除这个节点（使用``remove``函数）。在这个函数内部，首先需要将该节点的前后节点对接，然后将该几点的前后指针置空。
- 当有要移除节点的指针但是知道该节点在链表中的index，可以使用``removeAt``函数。在这个函数内部，首先根据index来获取对应的node的指针，然后再调用``remove``函数删除这个node。

##### 打印所有节点

```swift
public func printAllNodes(){

    guard head != nil else {
        print("linked list is empty")
        return
    }

    var node = head

    print("\nstart printing all nodes:")

    for index in 0..<count {

        if node == nil {
            break
        }

        print("[\(index)]\(node!.value)")
        node = node!.next

    }
}
```

该函数只是为了方便调试，为了跟踪链表的状态而定义的，它并不存在于链表的模型里。



为了验证上面这些方法的有效性，我们来实例化一个链表后实际操作一下，读者可以结合注释来看一下每一步对应的结果：



```swift
let list = LinkedList<String>()
list.isEmpty   // true
list.first     // nil
list.count     // 0

list.appendToTail(value: "Swift")
list.isEmpty         // false
list.first!.value    // "Swift"
list.last!.value     // "Swift"
list.count           //1

list.appendToTail(value:"is")
list.first!.value    // "Swift"
list.last!.value     // "is"
list.count           // 2

list.appendToTail(value:"great")
list.first!.value    // "Swift"
list.last!.value     // "great"
list.count           // 3


list.printAllNodes()
//[0]Swift
//[1]is
//[2]Great

list.node(atIndex: 0)?.value // Swift
list.node(atIndex: 1)?.value // is
list.node(atIndex: 2)?.value // great
list.node(atIndex: 3)?.value // nil


list.insert(LinkedListNode.init(value: "language"), atIndex: 1)
list.printAllNodes()
//[0]Swift
//[1]language
//[2]is
//[3]great


list.remove(node: list.first!)
list.printAllNodes()
//[0]language
//[1]is
//[2]great


list.removeAt(1)
list.printAllNodes()
//[0]language
//[1]great

list.removeLast()
list.printAllNodes()
//[0]language

list.insertToHead(value: "study")
list.count             // 2
list.printAllNodes()
//[0]study
//[1]language


list.removeAll()
list.printAllNodes()//linked list is empty

list.insert(LinkedListNode.init(value: "new"), atIndex: 3)
list.printAllNodes()
//[0]new

list.insert(LinkedListNode.init(value: "new"), atIndex: 3) //out of range
list.printAllNodes()
//[0]new

list.insert(LinkedListNode.init(value: "new"), atIndex: 1)
list.printAllNodes()
//[0]new
//[1]new
```





### 栈（Stack）



栈的讲解从

- 栈的定义
- 栈的抽象数据类型
- 栈的实现

三个部分来展开。



#### 栈的定义



首先来看一下栈的定义：

> 栈是限定仅在表的尾部进行插入和删除操作的线性表。



从定义中可以看出，我们知道我们只能在栈的一端来操作栈：

- 允许插入和删除的一端成为栈顶
- 另一端成为栈底

用一张图来看一下栈的操作：



![](https://user-gold-cdn.xitu.io/2018/1/31/16147cdb8db5eaf6?w=618&h=432&f=png&s=73000)





> 图源：《维基百科：Stack (abstract data type)》



从上图可以看出，最先压入栈里面的只能最后访问，也就是说，栈遵循后进先出（Last In First Out, LIFO）的原则。



#### 栈的抽象数据类型



```
ADT 栈（Stack）

Data
  linked list:持有的线性表

Operation
   init:初始化
   count:栈的元素个数
   isEmpty:是否为空
   push:入栈
   pop:出栈
   top:返回顶部元素
   
endADT
```



上面的operation可能不全，但是涵盖了栈的一些最基本的操作。那么基于这个抽象数据类型，我们来看一下如何使用Swift来实现它。





#### 栈的实现



笔者将数组（顺序存储）作为栈的线性表的实现，同时支持泛型。

```swift
public struct Stack<T> {
    
    //array
    fileprivate var stackArray = [T]()
    
    //count
    public var count: Int {
        return stackArray.count
    }
    
    //is empty ?
    public var isEmpty: Bool {
        return stackArray.isEmpty
    }
    
    //top element
    public var top: T? {
        
        if isEmpty{
            return nil
        }else {
            return stackArray.last
        }
        
    }
    
    //push operation
    public mutating func push(_ element: T) {
        stackArray.append(element)
    }
    
    
    //pop operation
    public mutating func pop() -> T? {
        
        if isEmpty{
            print("stack is empty")
            return nil
        }else {
            return stackArray.removeLast()
        }
    }
    
    //print all
    public mutating func printAllElements() {
        
        guard count > 0 else {
            print("stack is empty")
            return
        }
        
        print("\nprint all stack elemets:")
        for (index, value) in stackArray.enumerated() {
            print("[\(index)]\(value)")
        }
    }
}
```



- ``fileprivate``：是Swift3.0新增的访问控制，表示在定义的声明文件里可访问。它代替了过去意义上的``private``。而有了``fileprivate``以后，新的``private``则代表了真正的私有：在这个类或结构体的外部无法访问。
- 这里``printAllElements``方法也不属于抽象数据类型里的方法，也是为了方便调试，可以打印出所有的数据元素。



我们来实例化上面定义的栈实际操作一下：

```swift
var stack = Stack.init(stackArray: [])
stack.printAllElements() //stack is empty
stack.isEmpty //true

stack.push(2)
stack.printAllElements()
//[0]2

stack.isEmpty //false
stack.top     //2


stack.push(3)
stack.printAllElements()
//[0]2
//[1]3

stack.isEmpty //false
stack.top     //3


stack.pop()
stack.printAllElements()
//[0]2

stack.isEmpty //false
stack.top     //2


stack.pop()
stack.printAllElements() //stack is empty
stack.top //nil
stack.isEmpty //true

stack.pop() //stack is empty
```





### 队列（Queue）



队列的讲解从

- 队列的定义
- 队列的抽象数据类型
- 队列的实现

三个部分来展开。



#### 队列的定义





![](https://user-gold-cdn.xitu.io/2018/1/31/16147cdb8db00b50?w=618&h=432&f=png&s=95179)



> 图源：《维基百科：FIFO (computing and electronics)》





#### 队列的抽象数据类型

```
ADT 队列（Queue）

Data
  linked list:持有的线性表

Operation
   init:初始化
   count:栈的元素个数
   isEmpty:是否为空
   front:获取队列头元素
   enqueue:插入到队尾
   dequeue:删除队列头元素并返回
   
endADT
```



和上面的栈的实现一致，队列的实现也使用数组来实现队列内部的线性表。



#### 队列的实现

```swift
public struct Queue<T> {
    
    //array
    fileprivate var queueArray = [T]()
    
    
    //count
    public var count: Int {
        return queueArray.count
    }
    
    
    //is empty?
    public var isEmpty: Bool {
        return queueArray.isEmpty
    }
    
    
    //front element
    public var front: T? {
        
        if isEmpty {
            print("queue is empty")
            return nil
        } else {
            return queueArray.first
        }
    }
    
    
    //add element
    public mutating func enqueue(_ element: T) {
        queueArray.append(element)
    }
    
    
    //remove element
    public mutating func dequeue() -> T? {
        if isEmpty {
            print("queue is empty")
            return nil
        } else {
            return queueArray.removeFirst()
        }
    }
    
    //print all
    public mutating func printAllElements() {
        
        guard count > 0 else {
            print("queue is empty")
            return
        }
        
        print("\nprint all queue elemets:")
        for (index, value) in queueArray.enumerated() {
            print("[\(index)]\(value)")
        }
    }
    
}
```



我们初始化一个队列后实际操作一下：

```swift
var queue = Queue.init(queueArray: [])
queue.printAllElements()//queue is empty
queue.isEmpty //true
queue.count   //0


queue.enqueue(2)
queue.printAllElements()
queue.isEmpty  //false
//[0]2

queue.enqueue(3)
queue.printAllElements()
//[0]2
//[1]3


queue.enqueue(4)
queue.printAllElements()
//[0]2
//[1]3
//[2]4
queue.front //2


queue.dequeue()
queue.printAllElements()
//[0]3
//[1]4
queue.front //3


queue.dequeue()
queue.printAllElements()
//[0]4
queue.front //4

queue.dequeue()
queue.printAllElements() //queue is empty
queue.front //return nil, and print : queue is empty
queue.isEmpty //true
queue.count//0
```





# 最后的话



这两周学习数据结构和算法让我收获很多，除了强化了Swift语法以外，感觉自己看代码的感觉变了：看到一个设计就会想到里面所用到的数据结构，或是算法上面有没有可以优化的可能等等。

我相信对我来说编程的一扇新的门被打开了，希望自己可以坚持下去～



该系列的所有代码会放在我的GitHub的一个项目里面，项目地址：[Github:data-structure-and-algorithm-in-Swift](https://github.com/knightsj/data-structure-and-algorithm-in-Swift)

本篇文章的代码：

- Swift语法部分：[[1].Swift syntax](https://github.com/knightsj/data-structure-and-algorithm-in-Swift/tree/master/%5B1%5D.Swift%20syntax)
- 数据结构部分：[[2].Data structure](https://github.com/knightsj/data-structure-and-algorithm-in-Swift/tree/master/%5B2%5D.Data%20structure)



下篇预告：

从下一篇会开始正式讲解算法。本系列第二篇的主题是**排序算法**，内容是用Swift语言实现并讲解几种比较常见的排序算法：冒泡排序，选择排序，插入排序，希尔排序，堆排序，快速排序。

