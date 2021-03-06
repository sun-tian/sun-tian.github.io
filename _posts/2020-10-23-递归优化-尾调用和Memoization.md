---
layout:     post           # 使用的布局（不需要改）
title:      递归优化-尾调用和Memoization           # 标题 
subtitle:   递归优化-尾调用和Memoization #副标题
date:       2020-10-23             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/home-bg-art.jpg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 前端
    - 面试
---

# 递归优化：尾调用和Memoization

## 一、递归

### 1.递归含义：

>   一个过程或函数在其定义或说明中有直接或间接调用自身的一种方法，它通常把一个大型复杂的问题层层转化为一个与原问题相似的规模较小的问题来求解，递归策略只需少量的程序就可描述出解题过程所需要的多次重复计算，大大地减少了程序的代码量。

### 2.递归的优点

-   简洁
-   在树的前序，中序，后序遍历算法中，递归的实现明显要比循环简单得多。

例子1

```
function foo(i) {
  if (i < 0)
  return;
  console.log('begin:' + i);
  foo(i - 1);
  console.log('end:' + i);
}
foo(3);

// begin:3
// begin:2
// begin:1
// begin:0
// end:0
// end:1
// end:2
// end:3
复制代码
```

**以函数栈的方式来理解以上代码就是：**

1.  第一次进入函数foo(3),此时的参数为3,假设为foo1()被推入执行栈 首先 i不小于0，输出begin:3
2.  进入foo(i - 1)，参数为3-1 = 2，假设为foo2()被推入执行栈 i不小于0，输出begin:2
3.  进入foo(i - 1)，参数为2-1 = 1，假设为foo3()被推入执行栈 i不小于0，输出begin:1
4.  进入foo(i - 1)，参数为1-1 = 0，假设为foo4()被推入执行栈 i不小于0，输出begin:0
5.  进入foo(i - 1)，参数为0-1 = -1，假设为foo5()被推入执行栈 i小于0 return
6.  执行栈弹出当前的函数foo5()，进入到上一个函数foo4()，继续执行未完成的代码 输出end:0
7.  执行栈弹出当前的函数foo4()，进入到上一个函数foo3()，继续执行未完成的代码 输出end:1
8.  执行栈弹出当前的函数foo3()，进入到上一个函数foo2()，继续执行未完成的代码 输出end:2
9.  执行栈弹出当前的函数foo2()，进入到上一个函数foo1()，继续执行未完成的代码 输出end:3
10.  执行栈弹出当前的函数foo1()，到此执行栈全部执行完毕

例子2 阶乘函数

```
function factorial(n) {
  // console.trace()
  if (n === 0) {
    return 1
  }

  return n * factorial(n - 1)
}

factorial(5)

// 拆分成分步的函数调用
// factorial(5) = factorial(4) * 5
// factorial(5) = factorial(3) * 4 * 5
// factorial(5) = factorial(2) * 3 * 4 * 5
// factorial(5) = factorial(1) * 2 * 3 * 4 * 5
// factorial(5) = factorial(0) * 1 * 2 * 3 * 4 * 5
// factorial(5) = 1 * 1 * 2 * 3 * 4 * 5
复制代码
```

**下面是以上函数运行的图例**



![img](https://user-gold-cdn.xitu.io/2019/9/29/16d7abe6f2b26e60?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果在factorial函数中插入console.trace()来查看每次函数运行时的调用栈的状态，当递归到调用**factorial(5) = factorial(0) \* 1 \* 2 \* 3 \* 4 \* 5**时，输出结果如下：

```
console.trace
factorial @ VM159:2
factorial @ VM159:7
factorial @ VM159:7
factorial @ VM159:7
factorial @ VM159:7
factorial @ VM159:7
(anonymous) @ VM159:10
复制代码
```

### 3.递归的问题（缺点）

-   **性能**：如以上例子所示：假设传入的参数值特别大，那么这个调用栈将会非常之大，最终可能超出调用栈的缓存大小而崩溃导致程序执行失败。每一次函数调用会在内存栈中分配空间，而每个进程的栈的容量是有限的，当调用的层次太多时，就会超出栈的容量，从而导致栈溢出。

-   效率

    ：

    -   递归由于是函数调用自身，而函数调用是有时间和空间的消耗的：每一次函数调用，都需要在内存栈中分配空间以保存参数、返回地址以及临时变量，而往栈中压入数据和弹出数据都需要时间。
    -   递归中很多计算都是重复的，由于其本质是把一个问题分解成两个或者多个小问题，多个小问题存在相互重叠的部分，则存在重复计算，如fibonacci斐波那契数列的递归实现。

解决递归的性能问题的方法可以使用尾递归

## 二、尾递归

尾递归是一种递归的写法，可以避免不断的将函数压栈最终导致堆栈溢出。通过设置一个累加参数，并且每一次都将当前的值累加上去，然后递归调用。通过尾递归，我们可以**把复杂度从O(n)降低到O(1)**

先说尾调用来理解尾递归

>   尾调用是指一个函数里的最后一个动作是返回一个函数的调用结果的情形，即最后一步新调用的返回值直接被当前函数的返回结果

代码表现形式为：

```
function f(x) {
  a(x)
  b(x)
  return g(x) //函数执行的最后调用另一个函数
}
复制代码
```

#### 1.尾调用核心理解

就是看一个函数在调用另一个函数得时候，**本身是否可以被“释放”**

#### 2.尾调用好处

以下面函数调用栈和调用帧为例

```
function f(x) {
  res = g(x)
  return res+1
}

function g(x) {
  res = r(x)
  return res + 1
}

function r(x) {
  res = x + 1
  return res + 1
}
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/9/28/16d75db7cf081502?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如图，普通调用过程中，假如函数的调用层数非常多时，调用栈会消耗大量内存，甚至栈溢出，造成程序严重卡顿或意外崩溃。



**用尾调用解决栈溢出风险**

```
function f() {
  m = 10
  n = 20
  return g(m + n)
}
f()

// 等同于
function f() {
  return g(30)
}
f()

// 等同于
g(30)
复制代码
```

上述代码，我们可以看到，我们调用g之后，和f就没有任何关系了，函数f就结束了，所以执行到最后一步，完全可以删除 f() 的调用记录，只保留 g(30) 的调用记录。

**尾调用的意义** 如果将函数优化为尾调用，那么完全可以做到每次执行时，调用帧为一，这将大大节省内存，提高能效。

#### 3.尾递归 = 尾调用 + 递归

```
function factorial(n, total = 1) {
  // console.trace()
  if (n === 0) {
    return total
  }

  return factorial(n - 1, n * total)
}
复制代码
```

调用factorial(3)函数执行步骤如下：

```
factorial(3, 1) 
factorial(2, 3) 
factorial(1, 6) 
factorial(0, 6) // n = 0; return 6
复制代码
```

调用栈不再需要多次对factorial进行压栈处理，因为每一个递归调用都不在依赖于上一个递归调用的值。因此，空间的复杂度为o(1)而不是0(n)。查看控制台，发现第三次打印的结果如下：

```
console.trace
factorial @ VM362:2
factorial @ VM362:7
factorial @ VM362:7
factorial @ VM362:7
(anonymous) @ VM362:9
复制代码
```

既然说了调用栈不再需要多次对factorial进行压栈处理，那为什么结果还是不会在每次调用的时候压栈，只有一个factorial呢?

正确的使用方式应该是

```
'use strict';

function factorial(n, total = 1) {
  // console.trace()
  if (n === 0) {
    return total
  }

  return factorial(n - 1, n * total)
}

// 注意，虽然说这里启用了严格模式，但是经测试，在Chrome和Firefox下，还是会报栈溢出错误，并没有进行尾调用优化
// Safari浏览器进行了尾调用优化，factorial(500000, 1)结果为Infinity，因为结果超出了JS可表示的数字范围
// 如果在node v6版本下执行，需要加--harmony_tailcalls参数，node --harmony_tailcalls test.js
// 但是node最新版本已经移除了--harmony_tailcalls功能
复制代码
```

## 三、Memoization

>   memoization最初是用来优化计算机程序使之计算的更快的技术，是通过存储调用函数的结果并且在同样参数传进来的时候返回结果。大部分应该是在递归函数中使用。memoization 是一种优化技术，避免一些不必要的重复计算，可以提高计算速度。

同样以阶乘函数为例：

#### 1.不使用memoization

```
const factorial = n => {
  if (n === 1) {
    return 1
  } else {
    return factorial(n - 1) * n
  }
}
复制代码
```

#### 2.使用memoization

```
const cache = [] // 定义一个空的存放缓存的数组
const factorial = n => {
  if (n === 1) {
    return 1
  } else if (cache[n - 1]) { // 先从cache数组里查询结果，如果没找到的话再计算
    return cache[n - 1]
  } else {
    let result = factorial(n - 1) * n
    cache[n - 1] = result
    return result
  }
}
复制代码
```

#### 3.搭配闭包使用memoization

```
const factorialMemo = () => {
  const cache = []
  const factorial = n => {
    if (n === 1) {
      return 1
    } else if (cache[n - 1]) {
      console.log(`get factorial(${n}) from cache...`)
      return cache[n - 1]
    } else {
      let result = factorial(n - 1) * n
      cache[n - 1] = result
      return result
    }
  }
  return factorial
}

const factorial = factorialMemo()
复制代码
```

#### 4.总结

memorization 可以把函数每次的返回值存在一个数组或者对象中，在接下来的计算中可以直接读取已经计算过并且返回的数据，不用重复多次相同的计算。**是一个空间换时间的方式**，这种方法可用于部分递归中以提高递归的效率。


[Link](https://juejin.im/post/6844903954845810702)