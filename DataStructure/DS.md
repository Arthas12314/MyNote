# Data Structures

https://blog.csdn.net/johnny901114/category_9275511.html

## 数组

- 泛型的使用

- 动态数组的扩容，ArrayList 1.5*Capacity

- 时间复杂度分析

  * 添加操作O(n)

    addLast() O(1)  addFirst() O(n)  add() O(n/2)=O(n)  最坏情况O(n) resize()   综合后 O(n)

  * 删除操作O(m)

    removeLast() O(1)  removeFirst() O(n)  remove() O(n/2)=O(n)  最坏情况O(n) resize()   综合后 O(n)

  * 修改操作O(1)

    set() O(1)

  * 查找操作

    get() O(1) contains O(n) find O(n)

  resize() 的时间复杂分析

  * 均摊时间复杂度，每次addLast，removeLast均摊复杂度均为O(1)

  * 复杂度震荡 removeLast时 resize策略过于急躁

    Lazy resize

* 使用时最好索引具有语义

## 栈

LIFO

* 时间复杂度

  push() O(1)	pop() O(1)	peek()  O(1)	getSize() O(1)	isEmpty() O(1)

## 队列

FIFO

### 数组队列

* 时间复杂度

  enqueue O(1)均摊	dequeue O(n)	front() O(1)	getSize() O(1)	isEmpty() O(1)

### 循环队列

* 时间复杂度

  enqueue O(1)均摊	dequeue O(1)均摊	front() O(1)	getSize() O(1)	isEmpty() O(1)

## 链表

真正的动态，不需要处理固定容量的问题，丧失了随机访问的能力，不适合索引有语义的情况

* 添加 addLast() O(n) addFirst() O(1) add() O(n)
* 删除 removeLast() O(n) removeFirst() O(1) remove() O(n)
* 修改 set() O(n)
* 查找 get() O(n) contains O(n)

### 双向链表



## 递归

* 注意递归函数的宏观语义
* 求解最基本问题 把原问题转化成更小的问题

## 树

### 二叉搜索树

* 删除节点

  删除左右都有孩子的节点时

  * 找到 s = min(d->right)
  * s 为d 的后继
  * s->right=delMin(d->right)

* 二分搜索树的floor和ceil

  这里寻找floor或ceil的逻辑主要分为3个步骤

  * 如果node的key值和要寻找的key值相等：则node本身就是key的floor节点。
    如果node的key值比要寻找的key值大：则要寻找的key的floor节点一定在node的左子树中。
    如果node的key值比要寻找的key值小：则node有可能是key的floor节点, 也有可能不是(存在比node->key大但是小于key的其余节点)，需要尝试向node的右子树寻找一下。

* 二分搜索树的Rank和Select

  * rank方法的思路：从根结点开始，如果给定的键和根结点的键相等， 则返回左子树中的结点总数t;如果给定的键小于根结点，则返回改键在左子树的排名（递归计算）；如果给定的键大于根结点，则返回t+1(根结点)加上它在右子树中的排名。
    * 如果当前结点键小于key， 则说明key在左子树，向左子树递归。此时尚未确定key排名的下界，不需要增加Rank值。 
    * 如果当前结点键大于key，说明key在右子树， 向右子树递归。此时能够对key排名的下界进行进一步的计算。 计算方法：Rank = Rank的累计值 + 左子树结点总数+ 1
    * 如果当前结点键刚好等于key， 排名的递归计算结束，此时只要再加上左子树的结点总数就可以了。计算方法：Rank = Rank累计值 + 左子树结点总数
  * select方法的思路：查找排名为k的键，如果左子树中的结点数大于k, 那么我们就继续（递归地）在左子树中查找排名为k的键; 如果t等于k,我们就返回根结点中的键，如果t小于k，我们就（递归地）在右子树中查找排名为k-t-1的键

## 集合

LinkedListSet

* add() O(n) 	contanis O(n)	remove() O(n)  O(n)

BSTSet

* add() O(h)	contains O(h)	remove() O(h)   平均O(logn) 最差退化为链表时 O(n)

有序集合中的元素具有顺序性 基于搜索树的实现

无序集合中的元素没有顺序性 基于哈希表的实现

## 映射

LinkedListMap

* add() O(n)	remove() O(n)	set() O(n)	get() O(n)	contains() O(n)

BSTMap

* add() O(logn)	remove() O(logn)	set() O(logn)	get() O(logn)	contains() O(logn)
* 最差情况可能退化为O(n)

有序映射中的键具有顺序性 基于搜索树

无序映射中的键没有顺序性 基于哈希表

只考虑键值的映射即可视为一个集合

## 优先队列

### 堆

二叉堆时一颗完全二叉树

add() O(logn)	remove() O(logn)

### 优先级队列

## 线段树

update O(n) 

#### 动态线段树

1-100000关注某一段则临时生成

#### 树状数组

#### RMQ

Range Minimum Query

## Trie字典树

查询的复杂度与字典有多少条目无关，与查询单词的长度有关O(w)w为单词长度

## 并查集

解决连接问题

## AVL

# Algorithm

## 排序

注意对逆序对长度的评估

### 插入排序

* 提前终止内层循环
* 对几乎有序的序列排序效率甚至能达到O(n)

### 希尔排序

* 增量序列的选取

  * sedgewick序列  9\*pow(4,k)-9\*pow(2,k)+1或 pow(4,k)-3\*pow(2,k)+1

    最坏时间复杂度O(n^(4/3))平均O(n^(7/6))

  * 简单实现则增量序列选取3h+1

### 归并排序

O(nlogn) 

* 针对可能有序数组的优化 if(arr[mid]>arr[mid+1])

* 自顶向下的方式比自底向上的方式统计效率高一点，而自底向下无需额外创建数组空间

  自底向上适用于对链表进行归并

### 快速排序

* 优化主要在于轴点选取
* 相同元素较多则使用3Way快排

### 逆序对

可以应用归并或插入排序获取逆序对距离

快排可以应用于寻找第K大的元素

### 堆排序

O(nlogn) 就地  空间复杂度O(1) 

#### 索引堆

若数组储存的对象较大，swap操作用时长，则使用索引堆进行优化，

#### 优先队列

可优化nlogm

#### 多路归并

N路放入一个一个size为N的最小堆

### 排序的稳定性

稳定排序：对于相等的元素，在排序后，原来靠前的元素依然靠前，相邻元素的相对位置没有发生改变

*  插入排序与归并排序为稳定排序
* 快排与堆排为不稳定

## 二叉搜索树



# Leetcode分门别类

* 对一组数据进行排序

  * 有没有包含大量重复的元素
  * 是否近乎有序
  * 是否取值范围有限
  * 是否需要稳定排序
  * 是否使用链表储存
  * 数据大小是否可以装载在内存里
 优化、代码规范、容错性

## 算法复杂度分析

对一个字符串数组进行排序，先对每个字符串进行字母序排序，再对数组进行字母序排序，再对数组进行字典序排序 及字符串长度s，数组长度n

O(s·n·logn) +O(n·s·logs)=O(ns·logns)

要想在1s内解决

* O(n^2)处理大概10^4级别数据
* O(n)处理10^8级别的数据
* O(logn)处理10^7级别的数据

递归调用的复杂度分析

* 一次递归调用：递归深度depth 时间复杂度T，总体的时间复杂度O(T*depth)

均摊复杂度的分析：对vector的resize

### 写出正确的程序

* 明确变量的含义
* 循环不变量
* 小数据量调试

## 数组相关

* 双指针
* 滑动窗口