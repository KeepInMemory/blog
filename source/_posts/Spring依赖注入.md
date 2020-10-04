---
title: Spring依赖注入
date: 2020-10-04 16:12:00
tags:
  - spring
---

如果类A的功能实现需要借助于类B，那么类B是类A的依赖，如果在类A的内部去实例化类B，两者之间会出现较高的耦合度，一旦类B出现了问题，类A也需要进行改造，程序难维护。要解决这个问题就要把A类对B类的控制权抽离出来，交给第三方去做，把控制权反转给第三方，叫做控制反转

依赖注入是控制反转的典型实现方式，由第三方来控制依赖，把他通过构造函数、属性或者工厂模式等方法，注入到类A内，这样就实现了类A和类B的解耦

### 依赖注入的方式

#### XML方式

xml的bean标签申明对象，使用xml的property标签set方法注入，constructor-arg标签构造方法注入，factory-method标签工厂方法注入

##### 构造方法注入

```java
public class UserService implements IUserService {

	private IUserDao userDao;
	// 构造注入
	public UserService(IUserDao userDao) {
		this.userDao = userDao;
	}	
	// 注入后才能够使用
	public void loginUser() {
		userDao.loginUser();
	}
}
```

xml

```xml
<bean id="userService" class="com.lyu.spring.service.impl.UserService">
	// 构造属性注入bean
	<constructor-arg ref="userDaoJdbc"></constructor-arg>
</bean>

<bean id="userDaoJdbc" class="com.lyu.spring.dao.impl.UserDaoJdbc"></bean>
```

##### set方法注入

```java
public class UserService implements IUserService {

	private IUserDao userDao;
	// set注入,set+配置中name属性，首字母大写
	public setUserDao(IUserDao userDao) {
		this.userDao = userDao;
	}
	// 需加入空构造方法
	public UserService( ) {
	}
	public void loginUser() {
		userDao.loginUser();
	}
}
```

xml

```xml
<bean id="userService" class="com.lyu.spring.service.impl.UserService">
	// name属性注入bean，通过反射实现注入
	<property name="userDao" ref="userDaoMyBatis"></property>
</bean>

<bean id="userDaoJdbc" class="com.lyu.spring.dao.impl.UserDaoJdbc"></bean>
```

##### 工厂方法注入

工厂方法

```java
public class DaoFactory {  
    //实例工厂  
    public FactoryDao getFactoryDaoImpl(){  
        return new FactoryDaoImpl();  
    }  
}  
```

要注入的类

```java
public class SpringAction {  
    //注入对象  
    private FactoryDao factoryDao;  
      
    public void factoryOk(){  
        factoryDao.saveFactory();  
    }  
  
    public void setFactoryDao(FactoryDao factoryDao) {  
        this.factoryDao = factoryDao;  
    }  
} 
```

xml

```java
<!--配置bean,配置后该类由spring管理-->  
    <bean name="springAction" class="com.bless.springdemo.action.SpringAction">  
        <!--(4)使用实例工厂的方法注入对象,对应下面的配置文件(4)-->  
        <property name="factoryDao" ref="factoryDao"></property>  
    </bean>  
      
    <!--(4)此处获取对象的方式是从工厂类中获取实例方法-->  
    <bean name="daoFactory" class="com.bless.springdemo.factory.DaoFactory"></bean>  
    <bean name="factoryDao" factory-bean="daoFactory" factory-method="getFactoryDaoImpl"></bean> 
```

#### 注解方式

注解方式声明Bean使用的是@Component、@Controller、@Service、@Repository

注解方式的注入采用的是@Resource、@Autowired

@Resource：先按名称来找，找不到按类型来找

@Autowired：先按类型来找，找不到按名称来找

##### 构造方法注入

```java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public DI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC) {
    this.dependencyA = dependencyA;
    this.dependencyB = dependencyB;
    this.dependencyC = dependencyC;
}
```

##### set方法注入

```java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public void setDependencyA(DependencyA dependencyA) {
    this.dependencyA = dependencyA;
}

@Autowired
public void setDependencyB(DependencyB dependencyB) {
    this.dependencyB = dependencyB;
}

@Autowired
public void setDependencyC(DependencyC dependencyC) {
    this.dependencyC = dependencyC;
}
```

##### 成员变量注入

```java
@Autowired
private DependencyA dependencyA;

@Autowired
private DependencyB dependencyB;

@Autowired
private DependencyC dependencyC;
```

