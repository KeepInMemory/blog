---
title: toArray()用法
date: 2019-10-18 12:00:00
tags:
  - Java基础
---

```java
import java.util.ArrayList;
import java.util.List;

public class test {
    public static void main(String[] args){
        List<String> list = new ArrayList<String>();

        list.add("1");
        list.add("2");

        String[] tt = list.toArray(new String[4]);
        for(int i = 0; i < tt.length; i++)
            System.out.println(tt[i]);
    }
}
```

输出：

1
2
null
null

```java
import java.util.ArrayList;
import java.util.List;

public class test {
    public static void main(String[] args){
        List<String> list = new ArrayList<String>();

        list.add("1");
        list.add("2");

        String[] tt = list.toArray(new String[0]);
        for(int i = 0; i < tt.length; i++)
            System.out.println(tt[i]);
    }
}
```

输出：

1
2

toArray()传入的对象，大小如果比res的大小要小，那么转换大小为原res的大小。如果大小比res的大小要大，超出的部分会置为null

结合toArray的源码

```java
public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```