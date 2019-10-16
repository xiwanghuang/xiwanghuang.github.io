---
title: spring笔记day02(spring的新注解)
date: 2019-04-26 14:53:29
tags: spring
---
# spring笔记day02(spring的新注解)
本篇解决无法脱离xml文件的问题
## Configuration和ComponentScan
* Configuration  
作用：指定当前类是一个配置类
* ComponentScan  
作用：通过注解指定spring在创建容器时要扫描的包  
属性：value = backPackages (要扫描的包)  
使用此注解就相当于使用了xml文件中的`<context:component-scan base-package="com.itheima"></context:component-scan>`
* Bean：把当前方法的返回值作为bean对象存入spring的ioc容器中  
    属性： name：指定bean的id。默认值为当前方法的名称。
* 细节：当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象,查找方式和Autowired注解是一样的。
```java
@Configuration
@ComponentScan(basePackages = "com.itheima")
public class SpringConfiguration {

    /**
     * 创建一个queryrunner对象
     * @param dataSource
     * @return
     */

    @Bean("runner")
    public QueryRunner createQueryRunner(DataSource dataSource){

        return new QueryRunner(dataSource);
    }

    //创建数据源对象
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass("com.mysql.jdbc.Driver");
            ds.setJdbcUrl("jdbc:mysql://localhost:3306/eesy");
            ds.setUser("root");
            ds.setPassword("123456");
            return ds;
        }catch(Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
在测试类中使用`AnnotationConfigApplicationContext(SpringConfiguration.class)`方法获取容器。这样我们就可以删掉bean.xml文件了。
```java
//1.获取容器
        //ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
        //2.得到业务层对象
        IAccountService as = ac.getBean("accountService",IAccountService.class);
        //3.执行方法
        List<Account> accounts = as.findAllAcccount();
```
注意，此时的runner已经是单例的，而我们之前使用的是多例对象，这样会造成如果有多个用户同时使用接口时会产生线程冲突。因此我们需要加上`@Scope`注解。
### 一些@Configuration注解的细节
我们会发现，在去掉@Configuration之后程序依然能够运行。仿佛没有任何存在的意义。当配置类作为AnnotationConfig。。。对象创建的参数时，该注解可以不写。如
```java
ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class, JdbcConfig.class);
```
如果没有在这个参数表中的话就要加上Configuration注解并配置扫描的包。这个问题可以使用下面的Import注解解决。
## Import
作用：用于导入其他的配置类  
属性：value：用于指定其他配置类的字节码。当使用Import注解后，有import注解的类就是主（父）配置类，导入的都是子配置类。
```java
@Import(JdbcConfig.class)
public class SpringConfiguration {}
```
到了这一步，我们发现数据源的配置还是写死的状态。接下来我们将使用另一个注解解决这个问题。
##  PropertySource
* 作用：用于指定properties文件的位置
* 属性：  
  value：文件名称和路径  
  关键字：classpath表示类路径下
我们首先新建一个.properites文件。在里面配置数据源。
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy
jdbc.username=root
jdbc.password=123456
```
随后在我们的代码中加入注解。再将数据源中的字符串更换成我们所书写的变量。通过变量存储.properties文件中的各个值。
```java
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean("runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(DataSource dataSource){

        return new QueryRunner(dataSource);
    }
    //创建数据源对象
    @Bean(name="dataSource")
    public DataSource createDataSource(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        }catch(Exception e){
            throw new RuntimeException(e);
        }
    }
}
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {}
```
这样我们就完全摆脱了.xml文件。
## 总结
我们使用了纯xml文件的方式和纯注解的方式以及xml和注解混用的方式来进行对象的创建和容器的注入。那么具体使用哪种方法呢。  
在我们有权利选择的情况下。最好使用两种方法混用的方式。如果该类是别人写好的封装成jar包的类，我们最好使用xml的方式来进行。如果是我们自己写的，使用注解更方便。