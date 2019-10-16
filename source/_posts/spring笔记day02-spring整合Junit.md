---
title: spring笔记day02-spring整合Junit
date: 2019-04-26 20:03:48
tags:
---
## spring笔记day02-spring整合Junit
应用程序的入口为main()方法。junit单元测试中，没有main()方法也能执行，因为junit集成了一个main()方法。该方法可以判断当前测试类中哪些方法有`@Test`注解。junit就会让有注解的方法执行。junit并不会馆管我们是否采用spring框架。因此在执行测试方法时，junit不会为我们读取配置文件/配置类创建spring核心容器。所以就算写了`@Autowired`注解，也不会为我们注入数据。  
解决思路：使用能加载spring容器的main()方法。  
解决方法：
 * 1.导入spring-test的jar包  
 * 2.使用junit提供的注解把原有的main（）方法替换成spring提供的@Runwith注解  
 * 3.告知spring的运行器（基于xml还是注解），并说明位置  
  @ContextConfiguration:（当使用spring5.x版本时，要求junit的jar包必须是4.1.2及以上） 
 location:指定xml文件的位置，加上classpath关键字，表示在类路径下  
 classes:指定注解类所在的位置  
```xml
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
```
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")

public class AccountServiceTest {

    @Autowired
//    private ApplicationContext ac;
    private IAccountService as = null;

    @Test
    public void testFindAll() {

        //3.执行方法
        List<Account> accounts = as.findAllAcccount();
        for(Account account : accounts)
        {
            System.out.println(account);
        }
    }
```