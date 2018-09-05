# Android基础

#### LruCache （最近最少使用算法）

解决的问题：
在Android缓存这块，因为内存空间是有限的，不断的加入数据，迟早会到达最大值，LrhCache就是要解决这个问题的。

方案：
android提供API：LrhCache和DisLruCache

内容原理：
核心原理在于怎么实现删除过期数据。
LrhCache的实现使用了LinkedHashMap的特性，它可以做到按访问顺序排序，将最近访问过的放在最前面，一直没访问的对象，将放在队尾，即将被淘汰。

![hahah](./image/android/lrucache.png)

LinkedHashMap是由数组+双向链表的数据结构来实现的。其中双向链表的结构可以实现访问顺序和插入顺序，使得LinkedHashMap中的<key,value>对按照一定顺序排列起来。

双向链表：
也叫双链表，是链表的一种，它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点

应用：
Android中的三级缓存。



#### ConstraintLayout

相对定位：constraint
边距：margin
连接到GONE小部件时的边距:goneMargin(订单详情可以用用)
居中定位
偏差：constraintHorizontal_bias
循环定位：Circle
尺寸限制ConstraintLayout上的最小尺寸：min、max
使用0dp，相当于“ MATCH_CONSTRAINT”
防止约束失效:app:layout_constrainedWidth=”true” ,app:layout_constrainedHeight=”true”
链:

注意：您不能在约束中具有循环依赖关系。

