---
layout: post
title: "Tail-Recursion Optimization"
subtitle: "采用Python代码进行演示，但其本身不支持尾递归优化"
author: "Jack"
date: 2010-09-03
tag: [PAT, Tail-Recursion, Python]
header-img: "img/post-img/post-bg-unix.jpg"
---

尾递归和普通递归的不同在于对内存的占用，普通递归创建stack累积而后计算收缩，尾递归只占用恒量的内存（类似迭代）。可以通过尾递归优化来避免普通递归可能造成的爆栈问题。

以下采用Python代码举例：

- 普通递归

  ```python
  def power(x):
      if x == 1:
          return x
      else:
          return x * power(x - 1)
  ```

  当调用power(5)，Python调试器中发生如下状况：

  ```
  power(5)
  5 * power(4)
  5 * (4 * power(3))
  5 * (4 * (3 * power(2)))
  5 * (4 * (3 * (2 * power(1))))
  5 * (4 * (3 * (2 * 1)))
  5 * (4 * (3 * 3))
  5 * (4 * 6)
  5 * 24
  120
  ```

这个曲线代表了内存占用的峰值，从左至右到达顶峰，又从右至左收缩。

- 尾递归

  ```
  def tailPower(x, total = 1):
      if x == 1:
          return total
      else:
          return tailPower(x - 1, total * x)
  ```

  调用状况如下：

  ```
  tailPower(5, 1)
  tailPower(4, 5)
  tailPower(3, 20)
  tailPower(2, 60)
  tailPower(1, 120)
  120
  ```

观察到`tailPower(x, y)`中形式变量`y`的实际变量值是不断更新的，而普通递归中每个`power()`调用中的`y`值不变，仅在层级上加深，**尾递归是把变化的参数传递给递归函数**。

形式上，只要将最后一个`return语句`写成单纯的函数就可以变成尾递归。如`return x * power(x - 1)`写成`tailPower(x - 1, total * x)`。

***注：Python不支持尾递归优化，这里只是用Python语言写例子***