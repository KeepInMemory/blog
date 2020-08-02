---
title: Java核心卷1笔记
date: 2019-08-31 12:00:02
tags:
  - Java核心卷
---

#### Java的基本程序设计结构

- 十六进制数值有一个前缀0x或0X，八进制有一个前缀0。

- float数值有一个后缀F或f，double有D或者d后缀或无后缀。

- 浮点数值的舍入误差原因是浮点数值采用二进制系统表示，无法精确表示。

- c/c++中整形和布尔值可以转换，Java中不行。

- final表示常量，static final表示类常量，类常量定义在方法的外部，可以在同一个类的多个方法中使用。

- String字符串中的每个字符均不可修改。

- ```java
  s = s.substring(a,b);
  int age = 1;
  String s = s + age;
  ```

  s指向从位置a开始，长度为b的子串，并将age拼接在字串后面。

- 带标签的break语句类似于goto语句，仅将break换成goto即可

- 数组拷贝，第二个参数为数组的长度，该方法也用于增加数组的大小，增加的部分整形为0，布尔型为false

  ```java
  int []copiedNum=Arrays.copyOf(Nums,Nums.length);
  Nums = Arrays.copyOf(Nums,Nums.length + 2);
  ```

<!--more-->

#### 对象与类

- 如果需要返回一个可变对象的引用，应对它进行克隆，以致于不会破坏封装性，因为返回的对象可以通过方法改变其中的值，进而改变了包含可变对象的原对象的值。

- 在编写一个类时没有编写构造器，系统会提供一个无参数的构造器，将实例域设置为默认值。

- 编译器的内联：调用e.getName()将被替换为访问e.name域

- private——仅对本例可见，public——对所有类可见，protected——对本包和所有子类可见，无修饰符——对本包可见。

- a.instanceof(b)方法判断的是a类是否是b类或者b类的派生类，a.getClass(b)方法判断的是a类是否是b类。

- int a = 5;自动装箱：Integer i = a；手动装箱：Integer i = new Integer(a);

- Integer b = new Integer(5);自动拆箱：int i = b;手动拆箱：int j = b.intValue(); 

- lambda表达式语法：参数，箭头以及一个表达式。

  ```java
  (String first,String second) ->
  {
  	if(first.length() < second.length()) return -1;
  	else if(first.length() > second.length()) return 1;
  	else return 0;
  }
  
  ()->{for(int i = 100; i >=0 ; i--)System.out.println(i);}
  ```

