---
title: spring笔记day02(IOC案例实现)
date: 2019-04-25 20:14:31
tags: spring
---
# spring笔记day02(IOC案例实现)
本篇将实现一个连接mysql数据库的一个账号管理的实例。
首先导入依赖
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>commons-dbutils</groupId>
            <artifactId>commons-dbutils</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.7.14</version>
        </dependency>
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.1.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
```
## 编写业务层、持久层
接下来依次编写业务层、持久层的接口和实现类。在这之前，我们已经在本地mysql数据库中新建了一个account的表结构。首先实现账户的实体类。
```java
/**
*账户的实体类
*/
public class Account implements Serializable{
    private Integer id;
    private String name;
    private Float money;

    public void setId(Integer id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setMoney(Float money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", money=" + money +
                '}';
    }
}
```
然后是业务层的接口类
```java
public interface IAccountService {

    /**
     * 查询所有account
     * @return
     */
    List<Account> findAllAcccount();

    /**
     * 查询一个
     */
    Account findAccountByid(Integer id);

    void saveAccount(Account account);

    void updateAccount(Account account);

    void deleteAccount(Integer id);
}
```
持久层的接口类
```java
/**
 * 账户持久层接口
 */

public interface IAccountDao {

    /**
     * 查询所有account
     * @return
     */
    List<Account> findAllAcccount();

    /**
     * 查询一个
     */
    Account findAccountByid(Integer id);

    void saveAccount(Account account);

    void updateAccount(Account account);

    void deleteAccount(Integer id);
}
```
业务层接口的实现类
```java
public class AccountServiceImpl implements IAccountService {

    private IAccountDao accountDao;

    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public List<Account> findAllAcccount() {
        return accountDao.findAllAcccount();
    }

    public Account findAccountByid(Integer id) {
        return accountDao.findAccountByid(id);
    }

    public void saveAccount(Account account) {
        accountDao.saveAccount(account);

    }

    public void updateAccount(Account account) {
        accountDao.updateAccount(account);
    }

    public void deleteAccount(Integer id) {
        accountDao.deleteAccount(id);
    }
}
```
持久层接口的实现类。这其中的QueryRunner对象我们也使用spring进行注入。
使用query方法是要注意，必须增加抛出异常的方法，否则会报错。
```java
public class AccountDaoImpl implements IAccountDao {

    private QueryRunner runner;

    public void setRunner(QueryRunner runner) {
        this.runner = runner;
    }

    public List<Account> findAllAcccount() {
        try{
            return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
        }catch (Exception e){
            throw new RuntimeException(e);
        }

    }

    public Account findAccountByid(Integer id) {
        try{
            return runner.query("select * from account where id = ? ",new BeanHandler<Account>(Account.class),id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void saveAccount(Account account) {
        try{
            runner.update("insert into account(name,money)values(?,?)",account.getName(),account.getMoney());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void updateAccount(Account account) {
        try{
            runner.update("updae account set name = ?,money = ? where id = ?",account.getName(),account.getMoney(),account.getId());
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void deleteAccount(Integer id) {
        try{
            runner.update("delete account where id = ?",id);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
## 搭建spring开发环境
/resource目录下新建bean.xml并配置对象。若配置正确，idea会提示进行下一个对象的配置。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--配置service对象-->
    <bean id = "accoutService" class = "com.itheima.service.impl.AccountServiceImpl" >
        <!--注入Dao对象(set方法)-->
        <property name="accountDao" ref = "accountDao"></property>
    </bean>

    <!--配置dao-->
    <bean id = "accountDao" class = "com.itheima.dao.impl.AccountDaoImpl">
        <!--注入QueryRunner对象-->
        <property name="runner" ref = "runner"></property>
    </bean>

    <bean id = "runner" class = "org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源(使用构造函数)-->
        <constructor-arg name = "ds" ref = "dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id = "dataSource" class = "com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--注入连接数据库的必备信息-->
        <property name="driverClass" value = "com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value = "jdbc:mysql://localhost:3306/eesy"></property>
        <property name="user" value="root"></property>
        <property name="password" value = "123456"></property>
    </bean>
</beans>
```
## 测试对数据库的增删改查
```java
public class AccountServiceTest {
    @Test
    public void testFindAll() {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        //3.执行方法
        List<Account> accounts = as.findAllAcccount();
        for(Account account : accounts)
        {
            System.out.println(account);
        }
    }
    @Test
    public void testFindOne() {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        //3.执行方法
        Account account = as.findAccountByid(1);
            System.out.println(account);
    }
    @Test
    public void testUpdate() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);

        Account account = as.findAccountByid(4);
        account.setMoney(123456f);
        as.updateAccount(account);

    }
    @Test
    public void testSave() {
        Account account = new Account();
        account.setName("test");
        account.setMoney(12345f);

        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);

        as.saveAccount(account);
    }
    @Test
    public void testDelete() {
        //1.获取容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        //3.执行方法
        Account account = as.findAccountByid(4);
    }
}
```
## 基于注解的IOC改造案例
重新配置bean.xml。
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

    <!--告知spring在创建容器时要扫描的包-->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!--配置QueryRunner-->
    <bean id = "runner" class = "org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源(使用构造函数)-->
        <constructor-arg name = "ds" ref = "dataSource"></constructor-arg>
    </bean>

    <!--配置数据源-->
    <bean id = "dataSource" class = "com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--注入连接数据库的必备信息-->
        <property name="driverClass" value = "com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value = "jdbc:mysql://localhost:3306/eesy"></property>
        <property name="user" value="root"></property>
        <property name="password" value = "123456"></property>
    </bean>
</beans>
```
改造配置代码成为纯注解。
```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {

    @Autowired
    private IAccountDao accountDao;

@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {

    @Autowired
    private QueryRunner runner;
```
## 总结
以上即为spring的ioc案例实现。尽管我们将对象的创建和数据的注入都交给了spring，但依然存在几个问题：  
1.test类中代码冗余
2.无法脱离xml文件