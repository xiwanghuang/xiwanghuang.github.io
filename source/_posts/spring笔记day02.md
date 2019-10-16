---
title: sping学习笔记day2(上)
date: 2019-04-21 18:33:26
tags: spring
---
# sping学习笔记day2(上)
## 主要内容
* spring基于注解的IOC以及IOC的案例
* spring中IOC的常用注解
* 案例使用xml方式和注解方式时间单表的CRUD操作  
持久层技术选择：dbutils
* 改造基于注解的IOC案例，使用纯注解的方式实现  
spring的一些新注解使用
* spring和Junit整合
## 1 基于注解的IOC配置
注解和xml对应分为四类
* 1.用于创建对象的注解。相当于在xml文件中编写的 &#60;bean&#62;标签。
* 2.用于注入数据的注解。相当于在xml文件中的&#60;bean&#62;标签中写一个&#60;property&#62;标签。
* 3.用于改变作用范围的注解。相当于在xml文件中的&#60;bean&#62;标签中使用scope属性。
* 4.和生命周期相关。相当于在&#60;bean&#62;标签中使用init-method和destroy-method。
```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

<!--告知spring在创建容器时要扫描的包，配置所需要的标签不是在beans的约束中，而是一个名称为context名称空间和约束中-->
    <context:component-scan base-package="com.itheima"></context:component-scan>

</beans>
```
### 1.1 用于创建对象的注解
| 注解名 | 作用 | 属性 | 作用 |
| --- | --- | --- | --- |
|*@Component*|把当前类对象存入spring容器中|*value*|用于指定bean的id。默认值是当前类名且首字母小写|
|*@Controller*|用于表现层|同上|同上|
|*@Service*|用于业务层|同上|同上|
|*@Repository*|用于持久层|同上|同上|
```java
@Component(value="accountservice")
public class AccountServiceImpl implements IAccountService {}

@Service(value="accountservice")
public class AccountServiceImpl implements IAccountService {}
```
```java
    public class Client {
    public static void main(String[] args) {
        //1.获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
       // 2.根据id获取bean对象
        IAccountService as =(IAccountService)ac.getBean("accountservice");
        IAccountDao adao =ac.getBean("accountDao",IAccountDao.class);
        System.out.println(as);
        System.out.println(adao);
    }
}
```
### 1.2 用于注入数据的注解
* Autowired  
作用：自动按照**类型**注入，只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功。  
若IOC容器中没有任何bean类型和注入的变量类型匹配则报错。  
若有多个匹配时,首先按照类型圈定匹配的对象，接下来使用变量名称作为bean的id继续查找，若都不一样即报错。  
出现位置：可以是变量上，也可以是方法上。  
细节：使用注解注入时，set方法就不是必须的了。
```java
@Service(value="accountService")
public class AccountServiceImpl implements IAccountService {

    @Autowired
    private IAccountDao accountDao ;

    public void saveAccount()
    {
        accountDao.saveAccount();
    }
}
```
* Qualifier  
作用：在按照类型注入的基础上再按照名称注入，它给类成员注入时不能单独使用，但给方法参数注入时可以。  
属性：value：用于指定注入bean的id。  
```java
    @Autowired
    @Qualifier("accountDao1")
```
* Resource:  
作用：直接按照bean的id注入。可以直接使用。  
属性：name：用于指定bean的id  
以上三个注解都只能注入其他bean类型的数据，基本类型和string类型无法使用上述注解实现。另外，集合类型的注入只能通过xml来实现。
```java
    @Resource(name="accountDao1")
    private IAccountDao accountDao = null ;
```
* Value：  
作用：用于注入基本类型和String类型的数据。  
属性：Value：用于指定数据的值。它可以使用Spring中的SpEL（spring的el表达式）SpEL的写法：${表达式}。
### 1.3 用于改变作用范围的注解
scope作用：用于指定bean的作用范围  
属性：value：指定范围的取值。常用取值：singleton prototype。若不写scope，默认取值是singleton。
### 1.4和生命周期相关(了解)
和bean标签中使用init/destroy-method是一样的  
* Predestroy：指定销毁方法
* Postconstruct：指定初始化方法












