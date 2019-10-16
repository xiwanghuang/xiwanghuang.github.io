---
title: Spring笔记01
date: 2019-04-19 21:07:05
tags: java
---
# **spring笔记（一）——程序的耦合和解耦**
耦合：程序之间的依赖关系  
第一步：通过反射来创建对象，而避免使用new关键字  
第二部：通过读取配置文件来获取要创建的对象全限定类名  
## 工厂模式解耦 
一个创建Bean对象的工厂  
Bean：可重用组件  
JavaBean！=实体类：实体类是Bean的一部分。JavaBean是使用java语言编写的可重用组件。
工厂就是创建serveice和dao对象的  
第一：需要一个配置文件来配置service和dao。  
　　　　配置的内容：全限定类名=唯一标识。（key=value）  
第二：通过读取配置文件中配置的内容，反射创建对象。  
配置文件可以是xml和properties  
```java
public class BeanFactory {

    private static Properties props;

    static{
        try {

            //实例化对象
            props = new Properties();
            //获取properties文件的流对象
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
        }catch(Exception e) {
            throw new ExceptionInInitializerError("初始化properties失败");
        }
    }

    /**
     * 根据bean的名称获取bean对象
     * @param beanName
     * @return
     */
    public static Object getBean(String beanName){//此处使用静态方法。因为无法在静态类中引用非静态对象
        Object bean = null;
        try{
            String beanPath = props.getProperty(beanName);
            bean = Class.forName(beanPath).newInstance();
        }catch(Exception e){
            e.printStackTrace();
        }
        return bean;
    }
}
```
## IOC
控制反转（IOC）  
降低程序间的依赖
spring中的IOC容器是是Map结构
### 使用spring中的IOC解决程序耦合
　　导入beanfactory的工程，delete掉factory。在resources下新建xml文件，该文件可自由命名。  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--吧对象的创建交给spring管理-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>

    <bean id = "accountDao" class = "com.itheima.dao.impl.AccountDaoImpl"></bean>

</beans>
```
获取spring的IOC核心容器，并根据id获取对象。
```java
public class Client {

    /**
     * 获取spring的IOC核心容器，并根据id获取对象
     * @param args
     */
    public static void main(String[] args) {
        //IAccountService as =new AccountServiceImpl();
        //1.获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        /**
            applicationcontext三个常用实现类
     *      CkassPathXmlApplicationContext:加载类路径下的配置文件，要求配置文件必须在类路径下
     *      FileSystemXml：加载磁盘任一路经下的配置文件
     *      AnnotationConfig：读取注解创建容器

        */
        //2.根据id获取bean对象
        IAccountService as =(IAccountService)ac.getBean("accountService");
        IAccountDao adao =ac.getBean("accountDao",IAccountDao.class);

        System.out.println(as);
        System.out.println(adao);
        
//        as.saveAccount();
    }
}
```
核心容器的两个接口引发出的问题
* ApplicationContext：  适用场景（单例对象创建）  
     它在构建核心容器时，创建对象采取的思想是采用立即加载的方式。也就是说，只要一读取完配置文件马上就创建配置文件中配置的对象。
* Beanfactory：    适用场景（多例对象）  
    创建对象时采取的策略是延迟加载的方式。什么时候根据id获取对象，什么时候才真正地创建对象。
```java
public static void main(String[] args) {
        //---------------BeanFactory----------------
        Resource resource = new ClassPathResource("bean.xml");
        BeanFactory factory = new XmlBeanFactory(resource);

        IAccountService as =(IAccountService)factory.getBean("accountService");
        System.out.println(as);
    }
}
```
spring框架可以从配置上的不同选择不同的对象创建模式。实例开发中更多的是采用ApplicationContext接口创建对象。
## spring对bean对象的管理细节
### 创建bean的三种方式
* 使用默认构造函数创建   
  　　在spring配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时。采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。并在xml文件中就会报错。
* 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"></bean>
        <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
    </beans>
```
* 使用工厂的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器）
```xml
  <bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"></bean>
  <bean id="accountService" class="com.itheima.factory.StaticFactory" factory-method="getAccountService"></bean>
```
```java
  package com.itheima.factory;
  import com.itheima.service.IAccountService;
  import com.itheima.service.impl.AccountServiceImpl;
  public class StaticFactory {
    public static  IAccountService getAccountService() {
        return new AccountServiceImpl();
    }
  }
```
### bean的作用范围
bean标签的scope属性用于用于指定bean的作用范围，取值：常用单例或者多例 
* singleton：单例（默认值）
* prototype：多例
* request：作用于web应用的请求范围
* session：作用于web应用的会话范围
* global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，它就是session

### bean的生命周期
* 单例对象：
            出生：容器创建时  
            活着：容器存在  
            死亡：容器销毁  
            总结：单例对象的生命周期与容器相同
```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"
          scope="singleton" init-method="init" destroy-method="destroy"></bean>
```
```java
public class AccountServiceImpl implements IAccountService {
    /**
     * 模拟保存账户
     */

    public AccountServiceImpl(){
        System.out.println("对象创建了");
    }

    public void saveAccount(){
        System.out.println("service中的saveAccount方法执行了");
    }
    public void init(){
        System.out.println("对象初始化");
    }
    public void destroy(){
        System.out.println("对象销毁");
    }
}
```
```java
public class Client {

    /**
     *
     * @param args
     */
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

        IAccountService as =(IAccountService)ac.getBean("accountService");
        as.saveAccount();
        ac.close();
    }
}

```
* 多例对象：
            出生：使用对象时spring框架创建  
            活着：对象在使用过程中  
            死亡：当对象长时间不用，且没有别的对象引用时，由Java的辣鸡回收器回收
## spring的依赖注入
依赖注入：dependency injection。  
IOC的作用： 降低程序间的耦合。  
依赖关系的管理：交给spring的维护。  
在当前类需要用到其他类的对象，有spring为我们提供，我们只需要在配置文件中说明依赖关系的维护：就称之为依赖注入。  
* 能够注入的类型：三类  
　　基本类型和String  
　　其他bean类型（在配置文件中或者注解配置过的bean）  
　　复杂类型  
* 注入的方式：  
**使用构造函数注入**  
使用的标签：constructor-org  
标签出现的位置：bean标签的内部  
标签中的属性:  
　　type：用于指定注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型  
　　index:用于指定要注入的数据给构造函数中指定索引位置的参数赋值，索引的位置从0开始  
　　name：用于指定给构造函数中指定名称的参数赋值（常用）  
　　value：用于给基本类型和string类型的数据  
　　ref：引用关联的bean对象，用于指定其他的bean类型数据。（在spring的IOC核心容器中出现过的bean对象）    
优势：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。  
弊端：改变了bean对象的实例化方式，使我们在创建对象时如果用不到这些数据也必须提供。
```java
public class AccountServiceImpl implements IAccountService {
    /**
     * 模拟保存账户
     */
    //如果是经常变化的数据，并不适用于注入的方式
    private String name;
    private Integer age;
    private Date birthday;

    public AccountServiceImpl(String name,Integer age,Date birthday){
        this.name=name;
        this.age=age;
        this.birthday=birthday;
    }

    public void saveAccount() {
        System.out.println("service中的saveAccount方法执行了"+name+","+age+","+birthday);
    }
}
```
```xml
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <constructor-arg name="name" value="test"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
        <constructor-arg name="birthday" ref="now"></constructor-arg>
    </bean>

    <!--配置一个日期对象-->
    <bean id="now" class="java.util.Date"></bean>
```
**使用set方法提供(较上面的方法更加常用)**     
　　涉及的标签；property  
　　出现的位置：bean标签的内部  
　　标签的属性：  
　　　　name:用于指定注入时所调用的set方法名称  
　　　　value:同上   
　　　　ref:同上    
　　优势：创建对象时没有明确的限制，可直接使用默认构造函数  
　　弊端：若有某个成员必须有值，则获取对象时有可能set方法没有执行  

```java
public class AccountServiceImpl2 implements IAccountService {
    /**
     * 模拟保存账户
     */

    //如果是经常变化的数据，并不适用于注入的方式
    private String name;
    private Integer age;
    private Date birthday;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public void saveAccount() {
        System.out.println("service中的saveAccount方法执行了"+name+","+age+","+birthday);
    }
}
```
```xml
<bean id="accountService2" class="com.itheima.service.impl.AccountServiceImpl2">
        <property name="name" value="test"></property>
        <property name="age" value="21"></property>
        <property name="birthday" ref="now"></property>
    </bean>
```
**使用注解提供**  
此处不做介绍  
**复杂类型/集合类型的注入**  
于给List结构集合注入的标签：list,array,set    
用于给map结构集合注入的标签：map，property  
　　**结构相同，标签可以互换**  
常用set方法进行注入。
```java
public class AccountServiceImpl3 implements IAccountService {

    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,String> myMap;
    private Properties myPops;

    public void setMyStrs(String[] myStrs) {
        this.myStrs = myStrs;
    }

    public void setMyList(List<String> myList) {
        this.myList = myList;
    }

    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }

    public void setMyMap(Map<String, String> myMap) {
        this.myMap = myMap;
    }

    public void setMyPops(Properties myPops) {
        this.myPops = myPops;
    }


    public void saveAccount() {
        System.out.println(Arrays.toString(myStrs));
        System.out.println(myList);
        System.out.println(mySet);
        System.out.println(myMap);
        System.out.println(myPops);
    }
}

```
```xml
<bean id="accountService3" class="com.itheima.service.impl.AccountServiceImpl3">
        <property name="myStrs">
            <array>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </array>
        </property>
        <property name="myList">
            <list>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </list>
        </property>
        <property name="mySet">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>
        <property name="myMap">
            <map>
                <entry key="testA" value="aaa"></entry>
                <entry key="testB" >
                    <value>
                        bbb
                    </value>
                </entry>
            </map>
        </property>
        <property name="myPops">
            <props>
                <prop key="testC" >ccc</prop>
                <prop key="testD" >ddd</prop>
            </props>
        </property>
    </bean>
```
