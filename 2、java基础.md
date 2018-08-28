# java基础

#### 方法
原理+小细节

####内容
##### 排序
1、选择排序
首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置（交换位置），然后再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。

```
 /**
     * 选择
     *
     * @param arr
     */
    private void sort1(int[] arr) {
        int i, j, min, temp, leng = arr.length;
        for (i = 0; i < leng - 1; i++) {
            min = i;
            for (j = i + 1; j < leng; j++) {
                if (arr[min] > arr[j]) {
                    min = j;
                }
            }
            temp = arr[min];
            arr[min] = arr[i];
            arr[i] = temp;
        }
    }
    
```

细节：两次for循环的模式：for(起点；终点；++)，
     记录角标，最后交互数据。

2、冒泡
从左到右不断交换相邻逆序的元素，在一轮的循环之后，可以让未排序的最大元素上浮到右侧。

```
public static void bubbleSort(int[] arr) {
    int i, temp, len = arr.length;
    boolean changed;
    do {
      changed = false;
      len-=1;
      for (i = 0; i < len; i++) {
        if (arr[i] > arr[i + 1]) {
          temp = arr[i];
          arr[i] = arr[i + 1];
          arr[i + 1] = temp;
          changed = true;
        }
      }
    } while (changed);
  }
```

细节：终止条件为一道遍历下来，没有交换位置的了。

3、插入
故事：有个在线去了个排队号的顾客，现在来到了餐厅已经排好的队伍前， 怎么实现找到自己的位置。

它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

```
  /**
     * 擦入
     *
     * @param arr
     */
    private void sort3(int[] arr) {
        int i, j, temp, leng = arr.length;
        for (i = 1; i < leng; i++) {//无须
            temp = arr[i];
            j = i - 1;//从有序未开始比较
            while (j >= 0 && arr[j] < temp) {//有序
                arr[j + 1] = arr[j];//后靠～～～
                j--;
            }
            arr[j + 1] = temp;
        }
    }

```

