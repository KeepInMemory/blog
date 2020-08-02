---
title: Java集合初识
date: 2019-09-07 12:00:01
tags:
  - Java核心卷
  - 集合
---

### 接口和实现分离

------

接口：是代表集合的抽象数据类型。例如 Collection、List、Set、Map 等。

实现（类）：是集合接口的具体实现。从本质上讲，它们是可重复使用的数据结构，例如：ArrayList、LinkedList、HashSet、HashMap。

<!--more-->

### 迭代器

------

Java迭代器位于两个元素之间，当调用next时，迭代器就越过下一元素并返回刚刚越过的那个元素的引用。比如想删掉字符串集合中第一个元素的方法

```java
Collection c = new 
Iterator<String> it = c.iterator();
it.next();
it.remove();
```

迭代器访问时必须顺序访问。

### 集合框架

------

Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。

Collection 接口又有 3 种子类型，List、Set 和 Queue

常用List：ArrayList、LinkedList

常用Set：HashSet、LinkedHashSet

常用Map：HashMap、LinkedHashMap 

#### List

ArrayList : 数组结构,查询快,增删慢,线程不安全,因此效率高.

Vector : 数组结构,查询快,增删慢,线程安全,因此效率低.

LinkedList : 链表结构,查询慢,增删快,线程不安全,因此效率高.

```java
package linkedList;
import java.util.*;
public class LinkedListTest {
    public static void main (String[] args)
    {
        List<String> a = new LinkedList<>();
        a.add("Amy");
        a.add("Cral");
        a.add("Erica");

        List<String>b = new LinkedList<>();
        b.add("Bob");
        b.add("Doug");
        b.add("Frances");
        b.add("Gloria");

        ListIterator<String> aIter = a.listIterator();
        Iterator<String> bIter = b.iterator();

        while(bIter.hasNext())
        {
            if(aIter.hasNext()) aIter.next();
            aIter.add(bIter.next());
        }
        System.out.println(a);

        bIter = b.iterator();
        while (bIter.hasNext()){
            bIter.next();
            if(bIter.hasNext()){
                bIter.next();
                bIter.remove();
            }
        }
        System.out.println(b);

        a.removeAll(b);
        System.out.println(a);
    }
}
```

#### Set

HashSet :  存储无序，元素无索引，元素不可以重复，底层是哈希表。通过元素的hashcode和equals来保证元素的唯一性。如果元素的hashcode值相同，才会判断equals是否为true； 如果元素的hashcode的值不同，不会调用equals。所以必须重写hashCode() 和 equals() 方法。

TreeSet：存储有序，元素无索引，不可重复，底层是树结构。排序支持自然排序(Comparable接口)和定制排序(Comparator接口)，增删元素比Hash Set慢，但查找元素比HashSet快。

```java
package treeSet;
import java.util.*;
public class Item implements Comparable<Item>{
    private String description;
    private int partNumber;

    public Item(String description,int partNumber){
        this.description = description;
        this.partNumber = partNumber;
    }
    public String getDescription(){
        return description;
    }
    public String toString(){
        return description+","+partNumber;
    }

    @Override
    public boolean equals(Object obj) {
        if(this == obj) return true;
        if(obj == null) return false;
        if(getClass() != obj.getClass()) return false;
        Item other = (Item)obj;
        if(Objects.equals(description,other.description)&&partNumber == other.partNumber)
            return true;
    }

    @Override
    public int hashCode() {
        return Objects.hash(description,partNumber);
    }
    public int compaerTo(Item other){
        int diff = Integer.compare(partNumber,other.partNumber);
        return diff!=0? diff:description.compareTo(other.description);
    }
}
```

#### Map

**更新映射项**

使用一个映射统计一个单词在文件中出现的次数

```java
counts.put(word,counts.get(word)+1);
```

但当单词第一次出现时，get会返回null出现异常，这里有两种解决办法。

第一种是使用getOrDefault方法

```java
counts.put(word,counts.getOrDefault(word,0)+1);
```

第二种使用putIfAbsent方法，如果传入key对应的value已经存在，就返回存在的value，不进行替换。如果不存在，就添加key和value。

```java
counts.putIfAbsent(word,0);
counts.put(word,counts.get(word)+1);
```

**映射视图**

三种视图：

键集：Set keySet()

值集：Collectuinvalues()

键值对集：Set<Map.Entry<K,V>> entrySet()

查看映射的所有键

```java
Set<String> keys = map.keySet();
for(String key:keys){
    System.out.println(key);
}
```

查看映射的所有键值对

```java
for(Map.Entry<String,Employee> entry: staff.entrySet())
{
	System.out.println(entry.getKey()+entry.getValue());
}
```

### 位集

位集BitSet类用于存放位序列

```java
bucketOfBits.get(i)//如果第i位处于开状态，就返回true否则返回false。

bucketOfBits.set(i)//将第i位设置为开状态

bucketOfBits.clear(i)//将第i位设置为关状态
```

此外还有与、或、异或运算