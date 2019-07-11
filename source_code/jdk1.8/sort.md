## 排序

* 集合排序最终转换为数组排序

排序前后实现类型转换

```

//1. Collections.sort -> List.sort -> Arrays.sort()

default void sort(Comparator<? super E> c) {
        //转换成对象数组
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        //数组转换成特定类型元素的集合
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

```

* 元素个数<32 时，采用插入排序

```
// If array is small, do a "mini-TimSort" with no merges
if (nRemaining < MIN_MERGE) {
    //插入排序前先找到或生成有序部分的序列
    int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
    //插入排序，用二分法查找插入点
    binarySort(a, lo, hi, lo + initRunLen, c);
    return;
}

```

* 元素超过32时，采用归并排序

```


1 元素个数 >= 32, 采用归并排序，归并的核心是分区(Run)
2 找连续升或降的序列作为分区，分区最终被调整为升序后压入栈
3 如果分区长度太小，通过二分插入排序扩充分区长度到分区最小阙值
4 每次压入栈，都要检查栈内已存在的分区是否满足合并条件，满足则进行合并
5 最终栈内的分区被全部合并，得到一个排序好的数组

```


Timsort的合并算法非常巧妙：

* 找出左分区最后一个元素(最大)及在右分区的位置
* 找出右分区第一个元素(最小)及在左分区的位置
* 仅对这两个位置之间的元素进行合并，之外的元素本身就是有序的

```

 /*
  * Find where the first element of run2 goes in run1. Prior elements
  * in run1 can be ignored (because they're already in place).
  */
 int k = gallopRight((Comparable<Object>) a[base2], a, base1, len1, 0);
 assert k >= 0;
 base1 += k;
 len1 -= k;
 if (len1 == 0)
     return;

 /*
  * Find where the last element of run1 goes in run2. Subsequent elements
  * in run2 can be ignored (because they're already in place).
  */
 len2 = gallopLeft((Comparable<Object>) a[base1 + len1 - 1], a,
         base2, len2, len2 - 1);
 assert len2 >= 0;
 if (len2 == 0)
     return;

 // Merge remaining runs, using tmp array with min(len1, len2) elements
 if (len1 <= len2)
     mergeLo(base1, len1, base2, len2);
 else
     mergeHi(base1, len1, base2, len2);

```
