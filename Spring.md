# Spring

中文文档https://www.docs4dev.com/docs/zh/spring-framework/5.1.3.RELEASE/reference

Maven

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>

```



# IoC理论

1.UserDao接口

2.UserDaoImpl 实现类

3.UserService 业务接口

4.UserServiceImpl 业务实现类

传统实现

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao=new UserDaoOracleImpl();

    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

```java
@Test
public void getUser(){
    UserService userService =new UserServiceImpl();
    userService.getUser();
}
```

![image-20210513180959466](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210513180959466.png)

IOC实现

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao;
    //利用set进行动态实现值的注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

```java
@Test
public void getUser(){
    UserService userService =new UserServiceImpl();
    ((UserServiceImpl) userService).setUserDao(new UserDaoOracleImpl());
    userService.getUser();
}
```

**在之前的代码中，业务层的实现类由程序员指定，即在new UserDao时就由程序员指定了，但是在IoC中，我们将业务层的实现类交给用户指定，即将new UserDao的选择交给用户，这样就能实现按照用户需求来实现代码。**

在之前的业务中，用户需求的转变可能会影响代码，需要根据用户的需求去业务层修改代码，如果程序代码量非常大，修改一次的成本十分昂贵。

我们使用一个Set接口实现，已经发生了革命性的变化

```java
private UserDao userDao;
//利用set进行动态实现值的注入
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
```
+ 之前，程序时主动创建对象！控制权在程序员手上
+ 使用set注入之后，程序不再具有主动性，而是变成了被动的接受对象

![image-20210513181214088](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210513181214088.png)

这种思想，从本质上解决了问题，我们程序员不用再去管理对象的创建了。系统耦合性大大降低，可以更专注在业务的实现上，这是IOC的原型！

控制反转IoC，是一种设计思想，DI（依赖注入）是实现IoC的一种方法，也有人认为DI只是IoC的另一种说法。没有IoC的程序中，我们使用面向对象编程，对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后，将对象的创建转移给第三方，个人认为所谓的控制反转就是：获得依赖对象的方式反转了。



采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把二者合二为一，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。

**控制反转是一种通过描述（XML或注解）并通过第三方取生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入（DI）**

# HelloSpring

## 简单的Spring

**首先创建实体类pojo**

```java
public class Hello {
    private String str;

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }
}
```

**创建XML**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<!--使用Spring来创建对象，在Spring这些都称为bean
    类型 变量名 =new 类型();
    Hello hello = new Hello();

    id =变量名
    class=new 的对象;
    property 相当于给对象中的属性设置一个值！
-->
    <bean id="Hello" class="com.leo.pojo.Hello">
        <property name="str" value="Spring"/>
    </bean>
</beans>
```

创建XML是把pojo类托管给Spring，以后我们的操作大部分是针对XML进行的。

**测试类**

```java
public class MyTest {
    public static void main(String[] args) {
        //获取Spring的上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        //我们的对象现在都在Spring中管理了，我们要使用，直接取里面取出来就行了
        Hello hello = (Hello) context.getBean("Hello");
        System.out.println(hello);
    }
}
```

`ApplicationContext`是获取全部托管对象的接口，`ApplicationContext`实现类有很多，我们这里是取XML的方式。

`context.getBean("id")`这里的id是`<bean id`里面的id

## 上一个IoC程序改造成Spring

配置XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="MySqlImpl" class="com.leo.dao.UserDaoMySqlImpl"/>
    <bean id="OracleImpl" class="com.leo.dao.UserDaoOracleImpl"/>
    <bean id="UserServiceImpl" class="com.leo.service.UserServiceImpl">
        <!--ref：引用Spring容器中创建好的容器
          value：具体的值，基本数据类型
          -->
        <property name="userDao" ref="MySqlImpl"/>
    </bean>

</beans>
```

测试类

```java
public class MyTest {
    public static void main(String[] args) {
        //获取ApplicationContext：拿到Spring容器
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserServiceImpl  userServiceImpl = (UserServiceImpl) context.getBean("UserServiceImpl");
        userServiceImpl.getUser();
    }
}
```

通过配置XML中的UserServiceImpl的property，可以实现类似**set进行动态实现值的注入**的效果（实际上底层就是用setter来实现的）

## 思考问题

+  Hello对象是谁创建的？

	Hello对象是由Spring创建的

+ Hello对象的属性是怎么设置的？

	Hello对象的属性是由Spring容器设置的

这个过程就叫控制反转：

控制：谁来控制对象的创建，传统应用程序的对象是由程序本身控制创建的，使用Spring后，对象是由Spring来创建的

反转：程序本身不创建对象，而是被动地接收对象

依赖注入：就是利用set方法来进行注入

IOC是一种编程思想，由主动的编程变成被动的接收

可以通过new ClassPathXmlApplicationContext去查看底层源码

![image-20210513200453782](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210513200453782.png)

到了现在，我们完全不用在程序中去改动了，要实现不同的操作，只需要在XML配置文件中进行修改，所谓的IOC，一句话就是：对象由Spring来创建，管理，装配。

# IOC创建对象的方法

1.使用无参构造来创建

​	默认会调用无参构造

2.有参构造

![image-20210513205349184](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210513205349184.png)

**注意在new Context的时候就会把beans.xml中的所有实例全部创建，默认是单例模式，即每个类创建一个实例，我们后面的getbean取的都是同一个实例**



# Spring配置

## 别名

```xml
 <!--别名，如果添加了别名，我们也可以使用别名获取到这个对象-->
<alias name="User" alias="adadsad"/>
```

## Bean的配置

```xml
<!--
    id:bean的唯一标识符，也就相当于我们学的对象名
    class:bean对象所对应的全限定名，包名+类名
    name也是别名，可以同时取多个别名
-->
    <bean id="User2" class="com.leo.pojo.User2" name="user2 u2,u3">
        <property name="name" value="牛牛"/>
```

在bean中利用name取别名，逗号，空格都能分割

## import

import一般用于团队开发，他可以将多个配置文件导入合并为一个

假设，现在项目中有多个人开发，这三个人复制不同的类开发，不同的类需要注册在不同的bean中，我们可以利用import将所有人的beans.xml合并为一个总的

+ beans1.xml
+ beans2.xml
+ beans3.xml
+ applicationContext.xml

```xml
<import resource="beans.xml"/>
<import resource="beans2.xml"/>
<import resource="beans3.xml"/>
```

使用的时候，直接使用总的配置就可以了

# DI依赖注入

## 构造器注入

即利用构造函数注入

```xml
<bean id="User" class="com.leo.pojo.User">
    <constructor-arg name="name" value="leo"/>
</bean>
```

## set注入（重点）

+ 依赖注入：set注入
	+ 依赖：bean对象的创建依赖于容器
	+ 注入：bean对象中的所有属性，由容器来注入

【环境搭建】

1.复杂类型（已省去getter，setter）

```java
public class Address {
    private String address;
}
```

2.真实测试对象

```java
public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbys;
    private Map<String,String> card;
    private Set<String> games;
    private String wife;
    private Properties info;
}
```

3.beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="student" class="com.leo.pojo.Student">
        <!--第一种，普通值注入，value-->
        <property name="name" value="leo"/>
    </bean>
</beans>
```

4.测试类

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student);
    }
}
```

完善注入信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="address" class="com.leo.pojo.Address"/>
    <bean id="student" class="com.leo.pojo.Student">
        <!--第一种，普通值注入，value-->
        <property name="name" value="leo"/>
        <!--第二种，bean注入，ref引用其他bean-->
        <property name="address" ref="address"/>
        <!--数组-->
        <property name="books">
            <array>
                <value>红楼梦</value>
                <value>三国演义</value>
                <value>水浒传</value>
                <value>三国演义</value>
            </array>
        </property>
        <!--List-->
        <property name="hobbys">
            <list>
                <value>抽烟</value>
                <value>喝酒</value>
                <value>吃肥肉</value>
                <value>和异性交朋友</value>
            </list>
        </property>
        <!--map-->
        <property name="card">
            <map>
                <entry key="身份证" value="213121313131"/>
                <entry key="银行卡" value="213141341"/>
            </map>
        </property>
        <property name="games">
            <array>
                <value>LOL</value>
                <value>warThunder</value>
            </array>
        </property>
        <property name="wife">
            <null></null>
        </property>
        <property name="info">
            <props>
                <prop key="学号">20190525</prop>
                <prop key="性别">男</prop>
            </props>
        </property>
    </bean>
</beans>
```

运行结果

![image-20210514203627467](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210514203627467.png)

## 拓展方式注入

我们可以使用p命名空间和c命名空间进行注入

### p命名空间

官方实例

![image-20210515113229346](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515113229346.png)

### c命名空间

官方实例

![image-20210515113431349](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515113431349.png)

### 使用p和c标签

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--p(property)命名空间注入，可以直接注入属性的值，property-->
    <bean id="user" class="com.leo.pojo.User" p:name="leo"/>
    <!--c命名空间注入，通过构造器注入：constructor-arg-->
    <bean id="user2" class="com.leo.pojo.User" c:name="牛牛"/>
</beans>
```

### 测试p和c标签

```java
@Test
public void test2(){
    ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
    User user = (User) context.getBean("user2");
    User user = (User) context.getBean("user");
    System.out.println(user);
}
```

## bean 的作用域

![image-20210515123048742](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515123048742.png)

1.单例模式（Spring默认机制）

```xml
<bean id="user2" class="com.leo.pojo.User" c:name="牛牛" scope="singleton"/>
```

![image-20210515123322629](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515123322629.png)

2.原型模式：每次从容器中get的时候，都会产生一个新对象

```xml
<bean id="user2" class="com.leo.pojo.User" c:name="牛牛" scope="prototype"/>
```

![image-20210515123553175](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515123553175.png)

3.其余的request，session，application，这些个只能在web开发中使用到

# bean的自动装配

+ 自动装配是Spring满足bean依赖的一种方式
+ Spring会在上下文中自动寻找，并自动给bean装配属性



在Spring中有三种装配方式

1.在xml中显示的配置

2.在java中显示配置

3.隐式地自动装配bean【重要】

## 测试

测试类

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext contextt = new ClassPathXmlApplicationContext("beans.xml");
       People people = (People) contextt.getBean("people");
       people.getDog().shout();
       people.getCat().shot();
    }
}
```

## byName

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="cat" class="com.leo.pojo.Cat"/>
    <bean id="dog" class="com.leo.pojo.Dog"/>
    <!--
    byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的bean id
    byType：会自动在容器上下文中查找，和自己对象属性相同的bean，但是必须保证全局只是用一次这种类型的bean，不然会报错
    -->
    <bean id="people" class="com.leo.pojo.People" autowire="byName">
        <property name="name" value="leo"/>
    </bean>
</beans>
```

**byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的bean id**

![image-20210515130828601](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515130828601.png)

无法运行

## byType

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="cat" class="com.leo.pojo.Cat"/>
    <bean id="dog22222" class="com.leo.pojo.Dog"/>
    <!--
    byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的bean id
    byType：会自动在容器上下文中查找，和自己对象属性相同的bean，但是必须保证全局只是用一次这种类型的bean，不然会报错
    -->
    <bean id="people" class="com.leo.pojo.People" autowire="byType">
        <property name="name" value="leo"/>
    </bean>
</beans>
```

**byType：会自动在容器上下文中查找，和自己对象属性相同的bean，但是必须保证全局只是用一次这种类型的bean，不然会报错**

byType可以不利用bean id 进行装配，但是要保证bean的class唯一

![image-20210515131213669](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515131213669.png)

## 使用注解自动装配

要使用注解须知

1.导入约束,context约束

2.配置注解的支持<context:annotation-config/>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

使用注解装配不需要设置setter就能直接注入

**【重点】`@autowired`首先执行的是`byType`注入，当发现存在class不唯一后就会使用`byName`，有多个name的时候默认选择和对象成员名一致的bean id。如果所有的bean id和对象成员名不一致时，则会报错，这时需要使用`@Qualifier`来指定所用的bean id。**

## @autowired@Qualifier

```java
public class People {
    @Autowired
    @Qualifier(value = "cat2")
    private Cat cat;
    @Autowired
    private Dog dog;
    private String name;
```

## @Resource注解

```java
public class People {
    @Resource(name = "cat2")
    private Cat cat;
    @Resource
    private Dog dog;
    private String name;
```

小结：@Resource和@autowired的区别：

+ 都是用来自动装配的，都可以放在属性字段上
+ @Autowired默认通过byType方式实现，如果不唯一，则通过byType，而且必须要求这个对象存在【常用】
+ @Resource默认通过byName方式实现，如果找不到名字，则通过byType 实现，如果两个都找不到的情况下，就报错【常用】

## @component实现bean的注入

@component就是为了简化下面的配置

```xml
<bean id="cat" class="com.leo.pojo.Cat"/>
<bean id="cat2" class="com.leo.pojo.Cat"/>
<bean id="dog22222" class="com.leo.pojo.Dog"/>
<bean id="people" class="com.leo.pojo.People"/>
```

**使用**

首先在beans.xml中开启标签

```xml
<context:component-scan base-package="com.leo.pojo"/>
```

然后在实体类中使用@component标签

```java
@Component()
public class Cat {
    public void shot(){
        System.out.println("喵");
    }
}
```

@Component("指定bean id")可以指定bean id,默认是实体类小写，但貌似通过Component只能创建一个实体类

![image-20210515142516274](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515142516274.png)

# 注解开发

在Spring4之后，要使用注解开发，必须要保证aop的包导入了

![image-20210515144928008](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515144928008.png)

使用注解需要导入Context约束，增加注解支持

1.bean

2.属性如何注入

```java
@Component
public class User {

//    @Value("leo")
    public String name;
    //   相当于<property name="name" value="leo"/>
    @Value("leo")
    public void setName(String name) {
        this.name=name;
    }
}
```



3.衍生的注解

@Component有几个衍射注解，我们在web开发中，会按照mvc三层架构分层

+ dao【@Repository】
+ service【@Service】
+ controller【@Controller】

这四个注解功能是一样的，都是代表将某个类注册到Spring中，

4.自动装配

```
@Autowired：自动装配通过类型，名字
	如果Autowire的不能唯一自动装配上属性，则需要通过@Qualifier(value="XX")
@Nullable 字段标记了这个注解，说明这个字段可以为null
@Resource ：自动装配通过名字，类型	
```

5.作用域

@Scope,可以指定单例模式或者prototype

```java
@Component
@Scope("singleton")
public class User {

//    @Value("leo")
    public String name;
    //   相当于<property name="name" value="leo"/>
    @Value("leo")
    public void setName(String name) {
        this.name=name;
    }
}
```

6.小结

xml与注解：

+ xml更加万能，适用于任何场合，维护简单方便
+ 注解 不是自己类使用不了，维护相对复杂

xml和注解最佳实践：

+ xml用来管理bean
+ 注解只负责完成属性的注入
+ 在使用过程中，要注意，让注解生效，需要开启注解支持

```xml
<context:component-scan base-package="com.leo.pojo"/>
<context:annotation-config/>
```

# 使用Java的方式配置Spring

我们现在要完全不使用Spring的xml配置了，全权交给java来做！

JavaConfig是Spring的一个子项目，在Spring4之后，成为了一个核心功能

![image-20210515184524688](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515184524688.png)

实体类

```java
public class User {
    private String name;

    public String getName() {
        return name;
    }
    @Value("leo")
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

配置类

```java
@Configuration
//这个也会被Spring容器托管，注册到容器中，因为他本来就是一个@Component
// @Configuration代表这是一个配置类，就和bean.xml一样
public class MyConfig {
    //注册一个bean，相当于我们之前写的一个bean标签
    //这个方法的名字，就相当于bean标签中的id属性
    //这个方法的返回值，就相当于bean标签中的class属性
    @Bean
    public User getUser(){
        return new User();//就是返回要注入到bean的对象
    }
}
```

注意我们在这里new就已经相当于注册了一个bean，所以不需要在User类中加@Component标签

测试类

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        User getUser = (User) context.getBean("getUser");
        System.out.println(getUser.getName());
    }
}
```

# 代理模式

为什么要学习代理模式？因为这就是SpringAOP的底层！【SpringAOP和SpringMVC】

代理模式分类：

+ 静态代理
+ 动态代理

## 静态代理

角色分析：

+ 抽象角色：一般会使用接口或者抽象类来解决
+ 真实角色：被代理的角色
+ 代理角色：代理真实角色，代理真实角色后，我们会一般会做一些附属操作

代理模式的好处：

+ 可以使真实角色的操作更加纯粹！不用取关注一些公共的业务
+ 公共业务交给代理角色，实现了业务的分工
+ 公共业务发生扩展的时候，方便集中管理

缺点：

+ 一个真实角色就会产生一个代理角色：代码会翻倍，开发效率会变低

代码步骤

1.接口

```java
public interface Rent {
    public void rent();
}
```

2.真实角色

```java
public class Host implements Rent {
    @Override
    public void rent() {
        System.out.println("房东要出租房子");
    }
}
```

3.代理角色

```java
public class proxy {
    private Host host;
    public proxy() {
    }

    public proxy(Host host) {
        this.host = host;
    }
    public void rent(){
        seeHouse();
        host.rent();
        deal();
        fee();
    }
    //看房
    public void  seeHouse(){
        System.out.println("中介带你看房");
    }
    //合同
    public void deal(){
        System.out.println("签租赁合同");
    }
    //收中介费
    public void fee(){
        System.out.println("收中介费");
    }
}
```

4.客户端访问代理角色

```java
public class MyTest {
    public static void main(String[] args) {
        Host host = new Host();
        proxy proxy = new proxy(host);
        proxy.rent();
    }
}
```

![image-20210515201606751](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210515201606751.png)

## 动态代理

+ 动态代理和静态代理角色一样
+ 动态代理的代理类是动态生成的，不是直接写好的
+ 动态代理分为两大类：基于接口的动态代理，基于类的动态代理
	+ 基于接口---JDK动态代理【我们在这里使用】
	+ 基于类的---cglib
	+ java字节码实现：javasist

需要了解两个类：Proxy：代理，invocationhandler：调用处理程序

动态代理的好处：

+ 可以使真实角色的操作更加纯粹，不用去关注一些公共的业务
+ 公共也就交给代理角色，实现了业务的分工
+ 公共业务发生扩展时候，方便集中管理
+ 一个动态代理类代理的是一个接口，一般就是对应的一类业务
+ 一个动态代理类可以代理多个类，只要是实现了

### 提醒：代码里有丰富的注释，一定要看

**接口（抽象角色）**

```java
public interface Rent {
    public void rent();
    public void talk();
}
```

**真实角色**

```java
public class Host implements Rent{
    public void rent() {
        System.out.println("房东出租房子");
    }

    public void talk() {
        System.out.println("房东谈话");
    }
}
```

**InvocationHandler接口的实现（代理角色）**

```java
public class ProxyInvocationHandler implements InvocationHandler {
    //我们传入的是真实角色，但是注意，在整个ProxyInvocationHandler中，只有invoke方法里的method.invoke会调用真实角色，getProxy用的是target.getClass().getInterfaces()，即真实角色在其中的唯一作用就是获得其实现的接口
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }

    //生成得到代理类
    public Object getProxy(){
        //target其实是真实角色进行代理，但是这个代理类是取了实体类的接口类，也就是说为了和实体类解耦(避免代理类和具体的实体类产生联系)，选择更抽象的接口来创建代理类
         return Proxy.newProxyInstance(this.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //动态代理的本质，就是使用反射机制实现
        //调用抽象角色的方法，这里的method会随proxy.rent()或者proxy.talk()来产生不同的method
        //在这里由于调用了target即真实角色，就可以执行真实角色实现类里面的方法了，事实上，也正是这一步让我们的proxy.rent();proxy.talk();能够执行出真实角色的方法
        //这里的invoke就是注解与反射里的method.invoke方法，是用来执行某一个方法的，method是方法类，target是实体类，args是方法的参数
        Object result = method.invoke(target, args);
        return result;
    }
}
```

![image-20210516233405176](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210516233405176.png)

**测试**

```java
public class Client {
    public static void main(String[] args) {
        //选择要被代理的真实角色
        Rent host = new Host();
        //创建ProxyInvocationHandler类，类似于Factory,我们这一步的目的是创建出Proxy代理类
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        //设置真实对象，有两个目的，第一，给getProxy提供接口，第二，给invoke提供真实角色以便执行真实角色中实现的方法
        pih.setTarget(host);
        //获得代理类，将其强转为Rent来使用rent中的方法，
        Rent proxy = (Rent) pih.getProxy();
        //执行方法利用代理类执行方法，其实这个方法调用的本质还是invoke方法，.rent,和.talk只是赋值给method而已，能执行出rent也只是因为我们在invoke中有一段 Object result = method.invoke(target, args);而已
        proxy.rent();
        proxy.talk();
    }
}
```

### 我的理解

补充一下这段话：执行方法利用代理类执行方法，其实这个方法调用的本质还是invoke方法，.rent,和.talk只是赋值给method而已，能执行出rent也只是因为我们在invoke中有一段 Object result = method.invoke(target, args);而已。

innvoke是负责携带method的，即它是能否执行真实角色方法的必要条件，而Object result = method.invoke(target, args);是是否执行和具体去执行的代码。

之所以能实现解耦，是因为我们代理类的方法执行的时候，执行的只是ProxyInvocationHandler下的invoke方法，真实角色的方法只是由.rent和.talk注入到method中，然后在 Object result = method.invoke(target, args);中执行了，也就是说，我们的代理对象并不会受真实角色约束，可以说完全不受约束，他会去执行真实角色的方法也只是因为我们添加了一行代码Object result = method.invoke(target, args);让他执行了真实角色中的方法，我们甚至可以直接这样写

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    return null;
}
```

执行测试代码同样不会报错，我们知道了之所以会执行真实角色的方法只是因为Object result = method.invoke(target, args);，于是我们就可以把真实角色方法的执行**定位**到了Object result = method.invoke(target, args);这一行代码（即**切片操作**），也就是说如果我们要在这个方法执行的前，后，环绕来添加任意方法，我们就可以自由地在这行代码的前后添加我们想要的代码和功能。

动态代理中的ProxyInvocationHandler的目的本质上只是创建一个代理对象，而真正能实现对真实角色方法的调用，则是在invoke方法中进行，而invoke方法中的 Object result = method.invoke(target, args);则是实际的对真实角色方法进行执行的代码。通过一番操作，我们将真实角色中的某个方法的执行切片在了Object result = method.invoke(target, args);，到这里，我们就可以任意地在不破坏改变原真实角色的前提下，在真实角色的某个方法前后加上某些代码和功能。

**代理模式的本质就是对真实角色的方法进行定位切片，将其限定到某行代码中，然后在这行代码的前后左右添加代码和功能。这样就能实现在不改变真实角色的前提下，添加代码和功能**

PS：有点啰嗦了，但是我把我能想到的话都打上了，可能很多重复的，但我希望宁愿多重复，也不要漏过一点现在理解到的东西，毕竟睡一觉估计又忘了：）

# AOP

【重点】使用AOP要导入包

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

## 原生的Spring API（AfterReturningAdvice， MethodBeforeAdvice等API）实现AOP

**环绕增加的方法**

AfterLog

```java
public class AfterLog implements AfterReturningAdvice {
    //method:要执行的目标对象的方法
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了"+"返回值为"+returnValue);
    }
}
```

Log

```java
public class Log implements MethodBeforeAdvice {

    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了");
    }
}
```

抽象角色，即接口类

```java
public interface UserService {
    public void add();
    public void delete();
    public void update();
    public void select();
}
```

真实角色

```java
public class UserServiceImpl implements UserService{

    public void add() {
        System.out.println("增加一个用户");
    }

    public void delete() {
        System.out.println("删除一个用户");
    }

    public void update() {
        System.out.println("更新一个用户");
    }

    public void select() {
        System.out.println("查询一个用户");
    }
}
```

**配置ApplicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--注册bean-->
    <bean id="userService" class="com.leo.Service.UserServiceImpl"/>
    <bean id="log" class="com.leo.log.Log"/>
    <bean id="afterLog" class="com.leo.log.AfterLog"/>
    <!--配置aop：需要导入aop的约束-->
    <!--使用原生Spring API-->
    <aop:config>
        <!--选择需要进行代理的真实角色，同时确定在哪执行切片操作，在这里我们选择了UserSerImpl这个真实角色的所有方法-->
        <!--execution(修饰符  返回值  包名.类名/接口名.方法名(参数列表))注意老师忽略掉修饰符了 自己可以写上修饰符试试
(..)可以代表所有参数,(*)代表一个参数,(*,String)代表第一个参数为任何值,第二个参数为String类型.-->
        <aop:pointcut id="pointcut" expression="execution(* com.leo.Service.UserServiceImpl.*(..))"/>
        <!--执行MethodBeforeAdvice类，执行log中的before方法，他将处于我们注入的method前面-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <!--执行AfterReturningAdvice类，执行AfterLog中的before方法，它将处于我们注入的method前面-->
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

**测试类**

```java
public class MyTest {
    public static void main(String[] args) {
//        加载applicationContext.xml
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
//        得到Bean类，注意返回的类型是接口
        UserService userService = context.getBean("userService", UserService.class);
//        调用方法，这里调用的真实角色方法在源码中会利用反射注入到method类中
        userService.add();
        userService.delete();
    }
}

```

## 自定义实现AOP（切面实现）

相比之前添加一个切面方法类，即其中的方法用于在真实角色的方法前后执行

**diy**

```java
//这里我换成了注解注入bean
@Component
public class diy {
    public void before(){
        System.out.println("===========在方法执行前============");
    }
    public void after(){
        System.out.println("============方法执行后============");
    }
}
```

**配置ApplicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--注册bean-->
    <bean id="userService" class="com.leo.Service.UserServiceImpl"/>
    <bean id="log" class="com.leo.log.Log"/>
    <bean id="afterLog" class="com.leo.log.AfterLog"/>
    <context:component-scan base-package="com.leo.diy"/>
    <aop:config>
        <!--引用diy类-->
        <aop:aspect ref="diy" >
            <!--取得切点-->
            <aop:pointcut id="pointcut" expression="execution(* com.leo.Service.UserServiceImpl.*(..))"/>
            <!--在切点的不同的位置执行diy的方法-->
            <aop:before method="before" pointcut-ref="pointcut"/>
            <aop:after method="after" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

测试类不变

## 注解实现AOP

### 实现代码

注解实现极其简单

切面方法类AnnotationPointCut

```java
@Component
@Aspect
public class AnnotationPointCut {
    @Before("execution(* com.leo.Service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("===========在方法执行前============");
    }
    @After("execution(* com.leo.Service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("============方法执行后============");
    }
}
```

然后我们在xml中开启注解实现就完全ok了

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--注册bean-->
    <bean id="userService" class="com.leo.Service.UserServiceImpl"/>
    <bean id="log" class="com.leo.log.Log"/>
    <bean id="afterLog" class="com.leo.log.AfterLog"/>
    <context:component-scan base-package="com.leo.diy"/>
    <aop:aspectj-autoproxy/>
</beans>
```

测试类也完全不用改呢

### 拓展

这里介绍一种类似我们的动态代理实现的方式

```java
    @Around("execution(* com.leo.Service.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("===========环绕前============");
        //这里的就是我们之前在动态代理里说的，Object result = method.invoke(target, args);
        //既然拿到了方法的定位，我们自然可以自由地在其周围放方法了
        System.out.println(joinPoint.getClass());
        System.out.println(joinPoint.getSignature());
        System.out.println(joinPoint.getTarget());
        System.out.println(joinPoint.getThis());
        Object proceed = joinPoint.proceed();
        System.out.println(proceed);
        System.out.println("===========环绕后============");
    }
```

执行结果

![image-20210517195803504](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210517195803504.png)

可以看出，这里的joinpoint实际上就是真实角色的方法类

# Spring整合Mybatis

## 第一种方式

官方中文文档http://mybatis.org/spring/zh/index.html

创建pojo实体类

```java
package com.leo.pojo;

import lombok.Data;

@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

创建接口类

```java
public interface userMapper {
    public List<User> selectUser();
}
```

userMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.leo.Dao.userMapper">
    <select id="selectUser" resultType="User">
        select * from mybatis.user
    </select>
</mapper>
```

最重要的Spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--DaraSource:使用Spring的数据源替换Mybatis的配置
         我们这里提供Spring提供的jdbc-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTime=UTC&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="###164"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">

        <property name="dataSource" ref="dataSource" />
        <!--绑定Mybatis的配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <!--注册了Mapper，需要把mybatis-config.xml里的mapper注册删掉-->
        <property name="mapperLocations" value="classpath:com/leo/Dao/*.xml"/>
    </bean>
    <!--SqlSessionTemplate就是我们使用的sqlSession-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <!--这里只能用构造器注入sqlSessionFactory，因为查看源码可以知道它没有set方法-->
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
    <bean id="userMapper" class="com.leo.Dao.userMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```

mybatis-config.xml,一般用来进行基础配置

![image-20210517235752652](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210517235752652.png)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <typeAliases>
        <typeAlias type="com.leo.pojo.User" alias="User"/>
    </typeAliases>
</configuration>
```

测试类

```java
public class MyTest {
    public static void main(String[] args) {
       ApplicationContext context = new ClassPathXmlApplicationContext("Spring-dao.xml");
        userMapper userMapper = context.getBean("userMapper", userMapper.class);
        List<User> users = userMapper.selectUser();
        for (User user : users) {
            System.out.println(user);
        }
    }
}
```

## 第二种方式

第二种方式实际上就是把第一种中的创建sqlSession环节简化了

![image-20210518003832052](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518003832052.png)

我们只需要

```java
public class userMapperImpl2 extends SqlSessionDaoSupport implements userMapper {
    public List<User> selectUser() {
        return getSqlSession().getMapper(userMapper.class).selectUser();
    }
}
```

就可以实现对sqlSession的获取

# 声明式事务

## 回顾事务

+ 把一组业务当成一个业务来做，要么都成功，要么都失败

+ 事务在项目开发中，十分的重要，设计到数据的一致性，不能马虎
+ 确保完整性和一致性

事务ACID原则：

+ 原子性
+ 一致性
+ 隔离性
	+ 多个业务
+ 持久性

## Spring中的事务管理

+ 声明式事务：AOP
+ 编程式事务：需要在代码中，进行事务的管理

开启事务配置的时候注意选这种

![image-20210518214037603](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518214037603.png)

为了做测试，我们增加两个方法add和delete

其中delete方法我们故意打错，造成其无法执行

```java
public interface userMapper {
    public List<User> selectUser();
    @Update("insert into user (id,name,pwd) values (#{id},#{name},#{password})")
    public void addUser(User user);
    @Delete("deletes from user where id=#{id}")
    public void deleteUser(@Param("id") int id);
}
```

userMapperImpl

```java
public class userMapperImpl extends SqlSessionDaoSupport implements userMapper {
    public List<User> selectUser() {
        User user=new User(8,"leo","aaaad");
        SqlSession sqlSession = getSqlSession();
        userMapper mapper = sqlSession.getMapper(userMapper.class);
        mapper.addUser(user);
        mapper.deleteUser(8);
        return mapper.selectUser();
    }

    public void addUser(User user) {
        getSqlSession().getMapper(userMapper.class).addUser(user);
    }

    public void deleteUser(int id) {
        getSqlSession().getMapper(userMapper.class).deleteUser(id);
    }
}
```

 我们在不开启事务的时候，执行改造了的selectUser，可以看到，在不开启事务的时候，虽然delete有问题，但是add仍然正常执行

![image-20210518220731273](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518220731273.png)

![image-20210518220758479](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518220758479.png)

我们开启事务，这里采用AOP的方法开启事务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
        <!--DaraSource:使用Spring的数据源替换Mybatis的配置
         我们这里提供Spring提供的jdbc-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTime=UTC&amp;useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="###164"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">

        <property name="dataSource" ref="dataSource" />
        <!--绑定Mybatis的配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <!--注册了Mapper，需要把mybatis-config.xml里的mapper注册删掉-->
        <property name="mapperLocations" value="classpath:com/leo/Dao/*.xml"/>
    </bean>

    <!--配置声明式事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注意这里既可以用构造器注入，也可以set注入，官方用构造器，我们就用set-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--结合AOP实现事务的织入-->
    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* com.leo.Dao.userMapperImpl.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

![image-20210518221128741](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518221128741.png)

![image-20210518221150966](C:\Users\Lenovo\Desktop\笔记\Spring.assets\image-20210518221150966.png)

可以看见，开启事务后，delete执行失败后，add也失败了

思考：

为什么需要事务？

+ 如果不配置事务，可能存在数据提交不一致的情况
+ 如果我们不在Spring中去配置声明式事务，我们就需要在代码中手动配置事务
+ 事务在项目的开发中十分重要，涉及到数据的一致性和完整性问题，不容小觑