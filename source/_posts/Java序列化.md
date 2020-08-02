---
title: Java序列化
date: 2020-06-24 15:18:13
tags:
  - 序列化
---

Java序列化是指把Java对象转换为字节序列的过程，而Java反序列化是指把字节序列恢复为Java对象的过程。

作用：（1）永久性保存对象，保存对象的字节序列到本地文件或者数据库中；（2）通过序列化以字节流的形式使对象在网络中进行传递和接收；（3）通过序列化在进程间传递对象；

<!--more-->

#### Serializable接口

```java
import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionID = 1L;
    private int age;
    private  String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}

```

```java
import java.io.*;

public class Test {
    public static void main(String[] args) throws Exception {
        //SerializeUser();
        DeSerializeUser();
    }
    //序列化方法
    private static void SerializeUser() throws Exception{
        User user = new User();
        user.setName("Java的架构师技术栈");
        user.setAge(24);
        //序列化对象到文件中
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D://test.txt"));
        oos.writeObject(user);
        oos.close();
        System.out.println("序列化对象成功");
    }
    //反序列化方法
    private static void DeSerializeUser() throws Exception{
        File file = new File("D://test.txt");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        User newUser = (User)ois.readObject();
        System.out.println("反序列化对象成功"+newUser.toString());
    }
}

```



#### Externalizable接口

```java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;

public class User1 implements Externalizable {
    private int age;
    private String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User1{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);
        out.writeInt(age);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        name = (String) in.readObject();
        age = in.readInt();
    }
}
```

```java
import javax.sql.DataSource;
import java.io.*;

public class Test1 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //SerializeUser();
        DeSerializeUser();
    }
    public static void SerializeUser() throws IOException {
        User1 user1 = new User1();
        user1.setAge(24);
        user1.setName("zhangsan");
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D://test.txt"));
        user1.writeExternal(oos);
        oos.close();
    }
    public static void DeSerializeUser() throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D://test.txt"));
        User1 user1 = new User1();
        user1.readExternal(ois);
        System.out.println(user1.toString());
        ois.close();
    }
}
```

两者的区别：

Externalizable自定义序列化可以控制序列化的过程和决定哪些属性不被序列化，Serializable决定不了

使用Externalizable时，需要我们重写writeExternal()与readExternal()方法，必须按照写入时的确切顺序读取所有字段状态，否则会产生异常。

#### 需要注意的点

1.静态变量不会被序列化，对象的序列化是操作的堆内存中的数据，静态变量又称作类变量，类一加载就初始化了。

2.变量声明前加上该关键字Transient，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

3.serialVersionUID代表序列化的版本控制，如果序列化serialVersionUID=123456L，改为123456789L后反序列化，就会报错版本不一致。

