# 归并排序

归并排序，是创建在归并操作上的一种有效的排序算法。算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。归并排序思路简单，速度仅次于快速排序，为稳定排序算法，一般用于对总体无序，但是各子项相对有序的数列。

## 算法描述

1.申请空间使其为两个已排序序列长度之和，用于存放合并后的序列

2.设定两个指针，最初位置为两个序列的起始位置

3.比较两个指针指向的元素，将相对较小(大)的元素放入合并空间，并移动指针到下一个位置

4.将另一序列剩余元素存入合并序列尾部

## 算法实现

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
     *
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

## 拓展——使用分治算法合并N个已排序链表

### 问题描述

合并 k 个排序链表，返回合并后的排序链表。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

### 算法描述

* 将K个链表配对合并为一个链表
* 继续合并K/2个链表
* 直到合并为一个链表

### 算法实现

``` java

public ListNode mergeKLists(ListNode[] lists) {
    int len = lists.length;
    if (len == 0) {
        return null;
    } else if (len == 1) {
        return lists[0];
    } else if (len == 2) {
        return merge(lists[0], lists[1]);
    }
    int mid = len / 2;
    ListNode[] l1;
    l1 = Arrays.copyOfRange(lists, 0, mid + 1);
    ListNode[] l2;
    l2 = Arrays.copyOfRange(lists, mid + 1, len);
    return merge(mergeKLists(l1), mergeKLists(l2));
}

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

