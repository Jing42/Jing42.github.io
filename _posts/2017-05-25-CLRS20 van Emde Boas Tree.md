---
layout: post
title: CLRS20 van Emde Boas Tree
date: 2017-05-25
tag: 算法
---   

## 介绍

* van Emde Boas 树支持优先队列操作以及一些其他操作，每个操作的最坏情况运行时间为O(lg lg n), 这种数据结构限制关键字必须为0~n-1的整数且无重复。

* 支持SEARCH, INSERT, DELETE, MINIMUM, MAXIMUM, SUCCESSOR, PREDECESSOR.

### 20.1 基本方法

* 本届讨论动态集合的几种存储方法。

* 直接寻址: INSERT, DELETE, MEMBER运行时间为O(1), MINIMUM, MAXIMUM, SUCCESSOR, PREDECESSOR最坏情况下运行时间为Theta(u)

* 叠加的二叉树结构：每个操作最坏情况运行时间为O(lgu), MEMBER操作运行时间O(1)

* 叠加一棵高度恒定的树：INSERT为O(1), MINIMUM, MAXIMUM, SUCCESSOR, PREDECESSOR, DELETE为O(u^0.5). 使用度为u^0.5的树是产生 van Emde Boas 树的关键思想。

* **练习20.1-1**  修改本节数据结构，使其支持重复关键字。

```
将0/1表示法改成用关键字出现的具体数目来表示。
```

* **练习20.1-2** 修改本节中数据结构，使其支持带有卫星数据的关键字。

```
每个位保存关键字和指向卫星数据的指针。
```

* **练习20.1-3** 当x不包含在树中时，是说明如何在一棵二叉搜索树中查找x的后继。

```
先找到x应该在的位置，然后从这个虚拟结点出发，查找它后继。
```

* **练习20.1-4** 如果使用一棵叠加度为u^(1/k)的树，高度是多少，每个操作多长时间？

```
高度是k
操作时间是O(k+u^(1/k))
```

### 20.2 递归结构

* 现在使用结构递归，每次递归都以平方根大小缩减全域，直到降低到项数为2的基本大小时为止。

* 递归式 $T(u) = T(\sqrt{u}) + O(1)$的解为 $T(u) = O(lglgu)$，将用这个递归式来指导数据结构上的查找。

* 令 high(x) = $\lfoor{x/{\sqrt{u}}}\rfloor$, $low(x) = xmod{\sqrt{u}}$, index(x, y) = $x{\sqrt{u}} + y$

### 20.2.1 原型 van Emde Boas 结构

* 每个proto-vEB结构，记作proto-vEB(u)，有一个summary指针，指向一个proto-vEB($\sqrt{u}$)结构; 一个数组cluster，存储$\sqrt{u}$个指针，每个都指向一个proto-vEB($\sqrt{u}$)结构。