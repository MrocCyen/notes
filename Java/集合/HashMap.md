# HashMap

## 概述

- 数组+链表

- 根据键的hashcode定位到存储的位置

- 由于是数组，根据已知位置索引定位的时间复杂度为O(1)

- 线程不安全
  - 使用Collections的synchronizedMap转成线程线程安全的Map
  - ConcurrentHashMap替代

## 重要成员

### capacity（其实没有这个变量）

数组的容量（length），始终是2^n，默认是16

### loadfactor

负载因子，默认是0.75，可以在构造函数中进行设置

### threshold

扩容的阈值，等于capacity*loadfactor

### size

当前存储的数量

## put逻辑

- 1、如果table为null或者table的length为0，直接进行resize

- 2、如果tab[i]为空，直接设置值
  - n=table.lenth
  - i=(n-1)&hash

- 3、如果tab[i]不为空
  - 1、首节点的key、hash与查找的key、hash相等，赋值给变量e
  - 2、当前节点是红黑树节点，加入到红黑树中
  - 3、循环
    - 1、当前节点的next节点为空，创建新节点并赋值给next，如果链表节点数量大于等于创建红黑树的阈值（8），就将链表转为红黑树
    - 2、当前节点的key、hash与查询的key、hash相等，退出循环
  - 将新value赋值给旧value

## 扩容逻辑

扩容时机：size大于threshold

### 扩容实现

- 1、省略之前各种判断

- 2、创建一个原来2倍容量的数组

- 3、给新数组赋值

(e.hash & oldCap) == 0时给lotail.next赋值，否则给hiTail.next赋值

```java
if (loTail != null) {
loTail.next = null;
newTab[j] = loHead;
}
if (hiTail != null) {
	hiTail.next = null;
	newTab[j + oldCap] = hiHead;
}
```
## Java1.7产生循环链表

- 1.7存在，1.8已经解决

- 产生原因
  - 1.7使用头插法
  - 过程：线程B读取3->2，线程A会把3->2转换成2->3，然后线程B恢复后读取的是线程A转换后的tab，最终线程B会将3->2，这时形成循环链表

## 容量2^n倍

### 为什么？

- 根据hash值找table中的位置，传统使用hash%length进行取余

- 源码片段：if ((p = tab[i = (n - 1) & hash]) == null)

- 这里n=table.length

#### 源码分析

**代码中使用(n-1)&hash，等价hash%length**

### 初始化时构造2^n容量的算法

#### 反推法

1、如果要获得比一个数大的最小2^n的数，可以获得比这个2^n数小1的数，比如16，可以先获取15，然后再+1

2、无符号右移+或运算：将第一个不为0的位后面的位都设置为1

3、为什么需要减1？

排除本生就是2的n次方的情况，比如8，经过运算后为16，但是8已经满足条件了

4、为什么是1、2、4、8、16？

每次移动的位数，在取或后至少可以保证移动位的值为1

## 红黑树 in Java8

### 红黑树

- 每个节点红或者黑

- 根节点是黑

- 每个叶子节点是黑

- 如果一个节点是红，那么它的两个子节点都是黑

- 对于任意结点而言，其到叶结点树尾端NUIL指针的每条路径都包含相同数目的黑结点

### 为什么使用红黑树，而不使用其他的数据结构，比如AVL（完全平衡二叉树）？

- 红黑树插入、删除和查找都比较快

- AVL树查找很快，但是删除和插入较慢，旋转更难实现

### 为什么不一直使用红黑树，而是先使用链表，再使用红黑树？

- 1、内存占用与桶内查找复杂性之间的权衡

- 2、大多数hash函数hash冲突概率很小，因此为数量很少的（3或者4）维护树代价很高）

### 解决链表查找效率问题

- 链表最坏情况下为 O(N)

- 红黑树最坏为 O(logN)

### 链表与红黑树转换临界值

- 当链表的元素大于等于8的时候，就将链表转为红黑树

- 当链表元素小于等于6的时候，会重新转换为链表

- 为什么是8和6？
  - 长度为8时，红黑树查找平均长度为log(n)等于3，而长度为8的链表查找平均长度为8/2=4，这才有转换的必要 
  - 如果长度为6时，链表平均查找长度为6/2=3，虽然速度快，但是转化为树的过程就很费事，就没必要
  - 为什么中间还留7：避免不停插入和删除带来链表和树之间的转化带来的性能开销