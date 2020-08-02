---
title: 数组列表ArrayList
date: 2019-09-03 12:00:00
tags:
  - Java核心卷
  - 数组
---

```java
import java.util.*;
public class ArrayListTest {
    public static  void main (String []args){
        ArrayList<Integer> arrayList = new ArrayList<>();
        arrayList.ensureCapacity(5);//预估元素个数为5，影响初始数组内存的分配
        System.out.println(arrayList.size());//数组列表的大小
        arrayList.add(1);//添加元素
        arrayList.add(2);
        arrayList.add(3);
        arrayList.add(4);
        arrayList.trimToSize();//将比size大的预留空间去掉
        arrayList.set(0,2);//设置元素值，前下标后对象
        arrayList.add(4,5);//插入元素值，前下标后对象
        System.out.println(arrayList.get(3));//获取某个元素值
        System.out.println(arrayList.toString());
        arrayList.remove(4);//去除元素，参数为下标
        System.out.println(arrayList.size());
    }
}
```

ArrayList类解决了Java中数组不可变长的缺点，在添加和删除元素的时候可以自动调节数组的容量。

**数组列表管理着对象引用的一个内部数组，当添加元素到数组预定的最大空间时，数组列表就自动地创建一个更大的数组，将所有的对象从小数组拷贝到大数组中去。**

在本例中初始化了Integer的数组列表，**ensureCapacity**方法可以在填充元素前预估数组可能存储的元素数量5，**但数组的大小仍然是0**，用**add**方法添加元素，注意这里使用了**自动装箱**。随后使用**trimToSize**方法将数组预留的多余的空间去除，但不影响数组的大小。使用**set**方法将下标为0的元素赋值。使用**add**方法在下标4的位置插入元素5。使用**remove**方法去掉下标为4的元素，使用**size**方法显示元素数量。

程序输出：

0
4
[2, 2, 3, 4, 5]
4