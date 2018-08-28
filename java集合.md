# java集合

#### 出现的意义
##涉及到的数据结构：
1、Array
特点：大小固定， 扩展的话可以新建大的把小的copy进去。
优点：方便访问和修改。
缺点：插入数据代价大，需要将插入位置后面的数据依次向后移动。

2、Linked

![20141212083619402.png](http://upload-images.jianshu.io/upload_images/3162050-1c8f8f7c849abaea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
特点：一个节点里面包含了数据和下一个节点的指向，非连续，非顺序
优点：方便插入，删除。 
缺点： 查找慢。

3、Queue

![2243690-3116f05bb106b789.png](http://upload-images.jianshu.io/upload_images/3162050-b4eee82bc18544a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
特点：只能在头尾删除增加，先进先出
优点：普通队列，优先队列

4、Stack

![2243690-2e9540a7b4b61cbd.png](http://upload-images.jianshu.io/upload_images/3162050-86f4c8aeec8519ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
区分：heap，stack
特点：只能在栈顶操作，先进后出
优点：

5、Tree

![2243690-e20ffe8f48bfcfbc.png](http://upload-images.jianshu.io/upload_images/3162050-9c7c356f34148646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
特点：
优点：


###集合图

![687474703a2f2f696d672e626c6f672e6373646e2e6e65742f3230313430363238313434323035363235.jpeg](http://upload-images.jianshu.io/upload_images/3162050-1c9685e2459d6b17.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集合实现类的特征由数据结构特征决定。
基本结构：接口－》aabstract类－》具体实现
###Collection
####list
有序，可以重复（角标操作）
1、ArrayList
底层由数组实现，当超过数组大小，新建复制。
习惯性使用ArrayList，但是由于它的数据结构实现特性，插入数据比较麻烦，需要把插入数据位置后面的数据依次向后移动一个位置，所以如果需要经常插入或者删除元素，就选择LinkedList。其它选ArrayList。
2、LinkedList
底层由链表实现，实现了Queue接口，可以用作队列。

####set
没有重复数据
1、HashSet
底层由HashMap实现，无序。
2、TreeSet
底层由TreeMap实现，有序。
####queue
可以用LinkedList实现，
###Map
键值对，key值不能重复，
1、HashMap
高效：hashCode的实现可以直接通过hash值找到对应的value，在查询上效率很高。底层是用数组实现，数组中的元素包含的字段（value，key，next，hash）
2、LinkedHashMap
有序，插入或者访问顺序，通过链表实现。
3、TreeMap
排序，通过key值排序。
4、HashTable
和hashMap对比，很多是同步方法。
###使用指南

![image.png](http://upload-images.jianshu.io/upload_images/3162050-4533ecaf566e7436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



HashMap是由数组和链表组合构成的数据结构，Java8中链表长度超过8时会把长度超过8的链表转化成红黑树；存取时都会根据键值计算出”类别”(hashCode)，再根据”类别”定位到数组中的位置并执行操作。

hashmap的诞生，就是集合了数组和链表的优势，阀值 = 当前数组长度✖负载因子
初始长度原由， 高并发,链表环（concurrent）。 扩容（rehash）
android 实际使用： android优化过了一套集合框架，sparse～～
避免了对key的自动装箱, key二分查找法 , 选型：根据key类型。