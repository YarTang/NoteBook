# 排序算法的描述和实现



## 归并排序

归并排序，是创建在归并操作上的一种有效的排序算法。算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。归并排序思路简单，速度仅次于快速排序，为稳定排序算法，一般用于对总体无序，但是各子项相对有序的数列。

### 算法描述

1. 申请空间使其为两个已排序序列长度之和，用于存放合并后的序列

2. 设定两个指针，最初位置为两个序列的起始位置

3. 比较两个指针指向的元素，将相对较小(大)的元素放入合并空间，并移动指针到下一个位置

4. 将另一序列剩余元素存入合并序列尾部

### 算法实现

```java
/**
 * 归并排序
 * @param arr 输入数组
 * @param <T> 输入数组类型
 */
@SuppressWarnings("unchecked")
public static <T extends Comparable<? super T>> void mergeSort(T[] arr) {
    T[] tmpArr = (T[])new Comparable[arr.length];
    mergeSortBacktrack(arr, tmpArr, 0, arr.length - 1);
}

/**
 * 归并排序递归辅助方法
 * @param arr 输入数组
 * @param tmpArr 暂存数组
 * @param left 子数组的最左端下标
 * @param right 子数组最右端下标
 * @param <T> 输入数组类型
 */
private static <T extends Comparable<? super T>> void mergeSortBacktrack(T[] arr, T[] tmpArr, int left, int right) {
    if (left < right) {
        int center = (left + right) / 2;
        mergeSortBacktrack(arr, tmpArr, left, center);
        mergeSortBacktrack(arr, tmpArr, center + 1, right);
        merge(arr, tmpArr, left, center + 1, right);
    }
}

/**
 * 将两个子数组合并为一个
 * @param arr 输入数组
 * @param tmpArr 暂存数组
 * @param leftPos 子数组左半部粉开始位置在输入数组中的下标
 * @param rightPos 子数组右半部分开始位置在输入数组中的下标
 * @param rightEnd 子数组右半部分结束位置在输入数组中的下标
 * @param <T> 输入数组类型
 */
private static <T extends Comparable<? super T>> void merge(T[] arr, T[] tmpArr, int leftPos, int rightPos, int rightEnd) {
    int leftEnd = rightPos - 1;
    int tmpPos = leftPos;
    int numElements = rightEnd - leftPos + 1;
    // 比较两个子数组，并将元素放入暂存数组中
    while (leftPos <= leftEnd && rightPos <= rightEnd) {
        if (arr[leftPos].compareTo(arr[rightPos]) <= 0){
            tmpArr[tmpPos++] = arr[leftPos++];
        } else {
            tmpArr[tmpPos++] = arr[rightPos++];
        }
    }
    // 将两个子数组中的剩余元素复制到暂存数组中
    while (leftPos <= leftEnd) {
        tmpArr[tmpPos++] = arr[leftPos++];
    }
    while (rightPos <= rightEnd) {
        tmpArr[tmpPos++] = arr[rightPos++];
    }
    // 将暂存数组中的元素复制到原数组中
    for (int i = 0; i < numElements;i++, rightEnd--) {
        arr[rightEnd] = tmpArr[rightEnd];
    }
}
```

---

### 拓展——使用分治算法合并N个已排序链表

#### 问题描述

合并 k 个排序链表，返回合并后的排序链表。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

#### 算法描述

* 将K个链表配对合并为一个链表
* 继续合并K/2个链表
* 直到合并为一个链表

#### 算法实现

``` java
/**
 * 合并K个已排序链表
 */
public ListNode mergeKLists(ListNode[] lists) {
    int len = lists.length;
    if (len == 0) {
        return null;
    } else if (len == 1) {
        return lists[0];
    } else if (len == 2) {
        return merge(lists[0], lists[1]);
    }
    // 递归的方法将链表分为两组
    int mid = len / 2;
    ListNode[] l1;
    l1 = Arrays.copyOfRange(lists, 0, mid + 1);
    ListNode[] l2;
    l2 = Arrays.copyOfRange(lists, mid + 1, len);
    return merge(mergeKLists(l1), mergeKLists(l2));
}

/**
 * 合并两个已排序链表
 */
public ListNode merge(ListNode left, ListNode right) {
    ListNode dummy = new ListNode(0);
    ListNode pre = dummy;
    while (left != null && right != null) {
        if (left.val < right.val) {
            pre.next = left;
            left = left.next;
        } else {
            pre.next = right;
            right = right.next;
        }
        pre = pre.next;
    }
    if (left == null) {
        pre.next = right;
    } else {
        pre.next = left;
    }
    return dummy.next;
}
```

## 快速排序

### 算法描述

1. 如果数组 S 中的元素是 0 或 1 ，则返回；

2. 选取数组 S 中任意元素 v，作为枢纽元( **pivot** );

   三数中值分割法：使用左端、右端和中心位置上的三个元素的中值作为枢纽元

3. 将 S - v ( S 中其余元素 ) 划分为两个不相交集合 S1 和 S2;

   (1). 将 pivot 移到最右边；

   (2). 将小元素移到数组左边，将大元素移到数组右边

4. 返回 quickSort(S1) 后跟 v, 以及 quickSort(S2).

### 算法实现

```java
/**
 * 快速排序的截止范围，输入数组长度小于截止范围，则快排效率不高
 */
private static final int CUTOFF = 10;

/**
 * 快速排序递归实现
 * @param arr 输入数组
 * @param <T> 输入数组的类型
 */
public static <T extends Comparable<? super T>> void quickSort(T[] arr) {
    quickSortBacktrack(arr, 0, arr.length - 1);
}


/**
 * 快排递归实现辅助方法
 * @param arr 输入数组
 * @param left 子数组的左端在输入数组中的位置
 * @param right 子数组的右端在输入数组中的位置  
 * @param <T> 输入数组的类型
 */
private static <T extends Comparable<? super T>> void quickSortBacktrack(T[] arr, int left, int right) {
    //输入数组长度小于 CUTOFF 使用插入排序，大于CUTOFF使用快排
    if (left + CUTOFF <= right) {
        T pivot = median3(arr, left, right);
        int i = left, j = right - 1;
        while (true) {
            // 移动指针，将大于枢纽点的元素移动到左边，小于枢纽点的元素移到右边
            while (arr[++i].compareTo(pivot) < 0) {

            }
            while (arr[--j].compareTo(pivot) > 0) {

            }
            if (i < j) {
                swapReference(arr, i, j);
            } else {
                break;
            }
        }
        // 交换枢纽点 和 大元素子数组的第一个元素，保证枢纽点的左边都是小元素，右边都是大元素
        swapReference(arr, i, right - 1);

        // 对小元素子数组进行快排
        quickSortBacktrack(arr, left, i - 1);
        // 对大元素子数组进行快排
        quickSortBacktrack(arr, i + 1, right);
    }
    else {
        // 数组长度小于10，插入排序速度优于快排
        insertSort(arr, left, right);
    }

}

/**
 * 使用三数中值分割法找到输入数组的枢纽元, 且枢纽元存放在arr[right - 1]
 * @param arr 输入数组             
 * @param left 子数组的左端在输入数组中的位置 
 * @param right 子数组的右端在输入数组中的位置
 * @param <T> 输入数组的类型          
 * @return arr[left], arr[right], arr[center]的中值
 */
private static <T extends  Comparable<? super T>> T median3 (T[] arr, int left, int right) {
    int center = (left + right) / 2;
    // guarantee arr[left] < arr[center]
    if (arr[center].compareTo(arr[left]) < 0) {
        swapReference(arr, left, center);
    }
    // guarantee arr[left] < arr[right]
    if (arr[right].compareTo(arr[left]) < 0) {
        swapReference(arr, left, right);
    }
    // guarantee arr[center] < arr[right]
    if (arr[right].compareTo(arr[center]) < 0) {
        swapReference(arr, right, center);
    }
    // arr[right] > arr[center],则仅需将枢纽元放在倒数第二的位置上
    swapReference(arr, center, right - 1);
    return arr[right - 1];
}

/**
 * 数组宗两个元素交换位置
 * @param arr 输入数组
 * @param idx1 第一个元素的下标
 * @param idx2 第二个元素的下标
 * @param <T> 输入类型
 */
private static <T> void swapReference(T[] arr, int idx1, int idx2) {
    T tmp = arr[idx1];
    arr[idx1] = arr[idx2];
    arr[idx2] = tmp;
}
```

## 堆排序

使用堆( 优先队列 )实现堆排序。

### 算法描述



### 算法实现

```java
/**
 * 堆排序
 * @param arr 输入数组
 * @param <T> 输入数组的类型
 */
public static <T extends Comparable <? super T>> void heapSort(T[] arr) {
    // 建立一个最小堆
    for (int i = arr.length / 2- 1; i >=0; i--) {
        percDown(arr, i, arr.length);
    }
    // 将堆顶元素放到数组最后，将数组最后一个元素入堆
    // 因此该算法的结果是降序数组
    for (int i = a.length - 1; i >= 0; i--) {
        T tmp = arr[0];
        arr[0] = arr[i];
       	arr[i] = tmp;
        percDown(arr, 0, i);
    }
}

/**
 * 实现下滤操作
 * @param arr 输入数组
 * @param hole 空穴位置
 * @param len 当前二叉堆的大小
 * @param <T> 输入数组的类型
 */
public static <T extends Comparable <? super T>> void percDown(T[] arr, int hole, int len) {
    int child;
    T tmp;
    for (tmp = arr[hole]; hole * 2 + 1 < len; hole = child) {
        child = hole * 2;
        if (child != len - 1 && arr[child + 1].compareTo(arr[child]) < 0) {
            child++;
        }
        if (arr[child].compareTo(tmp) < 0) {
            arr[hole] = arr[child];
        } else {
            break;
        }
    }
    arr[hole] = tmp;
}
```



## 插入排序

### 算法描述

### 算法实现

```java
/** 
 * 插入排序
 * @param arr 输入数组
 * @param <T> 输入数组的类型，此处考虑使用泛型
 */
public static <T extends Comparable<? super T>> void insertSort(T[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int j;
        T tmp = arr[i];
        for (j = i; j > 0 && tmp.compareTo(arr[j - 1]) < 0; j--) {
            arr[j] = arr[j - 1];
        }
        arr[j] = tmp;
    }
}
```

## 冒泡排序

### 算法描述

### 算法实现

```java
/** 
 * 冒泡排序
 * @param arr 输入数组
 * @param <T> 输入数组的类型，此处考虑使用泛型
 */
public static <T extends Comparable<? super T>> void bubbleSort(T[] arr) {
    for (int i = 0; i < arr.length; i++) {
        for (int j = 0; j < arr.length; j++) {
            if (arr[j].compareTo(arr[i]) < 0) {
                T tmp = arr[i];
                arr[i] = arr[j];
                arr[j] = tmp;
            }
        }
}
```



