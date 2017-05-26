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

* 令 high(x) = ${\lfloor{x/{\sqrt{u}}}\rfloor}$, $low(x) = xmod{\sqrt{u}}$, index(x, y) = $x{\sqrt{u}} + y$

### 原型 van Emde Boas 结构

* 每个proto-vEB结构，记作proto-vEB(u)，有一个summary指针，指向一个proto-vEB($\sqrt{u}$)结构; 一个数组cluster，存储$\sqrt{u}$个指针，每个都指向一个proto-vEB($\sqrt{u}$)结构。

* 判断一个值是否在集合中

```
PROTO-vEB-MEMBER(V, x)
    if V.u == 2
        return V.A[x]
    else return PROTO-vEB-MEMBER(V.cluster[high(x)], low(x))

T(u) = O(lglgu)
```

* 查找最小元素

```
PROTO-vEB-MINIMUM(V)
    if V.u == 2
        if V.A[0] == 1
            return 0
        elif V.A[1] == 1
            return 1
        else reuturn NIL
    else min-cluster = PROTO-vEB-MINIMUM(V.summary)
        if min-cluster == NIL
            return NIL
        else offset = PROTO-vEB-MINIMUM(V.cluster[min-cluster])
            return index(min-cluster, offset)

T(u) = 2T(u^.5)+O(1)
根据递归式，运行时间为Theta(lg u)
```

* 查找后继

```
PROTO-vEB-SUCCESSOR(V, x)
    if V.u == 2
        if x == 0 and V.A[1] == 1
            return 1
        else return NIL
    else offset = PROTO-vEB-SUCCESSOR(V.cluster[high(x)], low(x))
        if offset != NIL
            return index(high(x), offset)
        else succ-cluster = PROTO-vEB-SUCCESSOR(V.summary, high(x))
            if succ-cluster == NIL
                return NIL
            else offset = PROTO-vEB-MINIMUM(V.cluster[succ-cluster])
                return index(succ-cluster, offset)
```

* 插入元素

```
PROTO-vEB-INSERT(V, x)
    if V.u == 2
        V.A[x] = 1
    else PROTO-vEB-INSERT(V.cluster[high(x)], low(x))
        PROTO-vEB-INSERT(V.summary, high(x))

运行时间为Theta(lg u)
```

### 20.3 van Emde Boas 树及其操作

* 在本节，我们设计一个类似与proto-vEB结构的数据结构，但要存储稍多一些的信息，由此可以去掉一些递归的需求。

* 现在允许u为任何一个2的幂，因此重新定义一些有用的函数：high(x) = floor(x/u的下平方根)，low(x) = x mod u的下平方根，index(x, y) = x*u的下平方根+y

### 20.3.1 van Emde Boas 树

* 一棵vEB树含有proto-vEB结构中没有的两个属性: min存储vEB树中的最小元素，max存储vEB树中的最大元素。

* 存储在min中的元素并不出现在任何递归的vEB($\sqrt{u}$)树中(不出现在它的任何一个簇中)

* 一棵vEB(2)树结构并不需要数组A, 可以从其min和max属性确定它的元素。

* min和max属性是减少vEB树上这些操作的递归调用次数的关键。

* 一棵van Emde Boas树的总空间需求是O(u), 直接创建一棵空vEB树需要O(u)时间。而红黑树只需要常数时间。

### 20.3.2 van Emde Boas 树的操作

* 查找最小元素和最大元素

```
vEB-TREE-MINNIMUM(V)
    return V.min

vEB-TREE-MAXIMUM(V)
    return V.max

均为O(1)时间
```

* 判断一个值是否在集合中

```
vEB-TREE-MEMBER(V, x)
    if x == V.min or x == V.max
        return True
    elif V.u == 2
        return FALSE
    else return vEB-TREE-MEMBER(V.cluster[high(x)], low(x))

O(lglgu)
```

* 查找前驱和后继

```
vEB-TREEE-SUCCESSOR(V, x)
    if V.u == 2
        if x == 0 and V.max == 1
            return 1
        else return NIL
    elif V.min != NIL an x < V.min
        return V.min
    else max-low = vEB-TREE-MIXIMUM(V.cluster[high(x)])
        if max-low != NIL and low(x) < max-low
            offset = vEB-TREE-SUCCESSOR(V.cluster[high(x)], low(x))
            return index(high(x), offset)
        else succ-cluster = vEB-TREE-SUCCESSOR(V.summary, high(x))
            if succ-cluster == NIL
                return NIL
            else offset = vEB-TREE-MINIMUM(V.cluster[succ-cluster])
                return index(succ-cluster, offset)


vEB-TREE-PREDECESSOR(V, x)
    if V.u == 2
        if x == 1 and V.min == 0
            return 0
        else return NIL
    elif V.max != NIL and x > V.max
        return V.max
    else min-low = vEB-TREE-MINIMUM(V.cluster[high(x)])
        if min-low != NIL and low(x) > min-low
            offset = vEB-TREE-PREDECESSOR(V.cluster[high(x)], low(x))
        return index(high(x), offset)
    else pred-cluster = vEB-TREE-PREDECESSOR(V.summary, high(x))
        if pred-cluster == NIL
            if V.min != NIL and x > V.min
                return V.min
            else return NIL
        else offset = vEB-TREE-MAXIMUM(V.cluster[pred-cluster])
            return index(pred-cluster, offset)
```

* 插入一个元素

```
vEB-EMPTY-TREE-INSERT(V, x)
    V.min = x
    V.max = x

vEB-TREE-INSERT(V, x)
    if V.min == NIL
        vEB-EMPTY-TREE-INSERT(V, x)
    elif x < V.min
        exchange x with V.min
    if V.u > 2
        if vEB-TREE-MINIMUM(V.cluster[high(x)]) == NIL
            vEB-TREE-INSERT(V.summary, high(x))
            vEB-EMPTY-TREE-INSERT(V.cluster[high(x)], low(x))
        else vEB-TREE-INESRT(V.cluster[high(x)], low(x))
    if x > V.max
        V.max = x
```

* 删除一个元素

```
vEB-TREE-DELETE(V, x)
    if V.min == V.max
        V.min = NIL
        V.max = NIL
    elif V.u == 2
        if x == 0
            V.min = 1
        else V.min = 0
    elif x == V.min
        first-cluster = vEB-TREE-MINIMUM(V.summary)
        x = index(first-cluster,
            vEB-TREE-MINIMUM(V.cluster[first-cluster]))
        V.min = x
        vEB-TREE-DELETE(V.cluster[high(x)], low(x))
        if vEB-TREE-DELETE(V.cluster[high(x)]) == NIL
            vEB-TREE-DELETE(V.summary, high(x))
            if x == V.max
                summary-max = vEB-TREE-MAXIMUM(V.summary)
                if summary-max == NIL
                    V.max = V.min
                else V.max = index(summary-max,
                             vEB-TREE-MAXIMUM(V.cluster[summary-max]))
        elif x == V.max
            V.max = index(high(x),
                vEB-TREE-MAXIMUM(V.cluster[high(x)]))
```