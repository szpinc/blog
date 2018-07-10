---
title: Spring注解@Async和@Transactional失效问题究竟是什么原因，强势解释一波
date: 2017-07-01 21:53:39
comments: false
tags:
    - java
    - spring
categories:
    - 学习笔记
    - java学习笔记
---

## 提前说说

项目中涉及到的代码我都会上传到码云(gitee)或者github上，提供给大家下载参考，文中就以最简单的方式说明执行过程。源码的地址在文末哦！



## 问题场景重现

**场景一：**

Spring的异步执行注解@Async，在调用这个方法的时候发现，不对劲，耗时的逻辑我已经加入到异步取做了，怎么接口请求的响应这么慢，赶紧看日志，懵X，加了异步注解，却没有异步执行。

**场景二：**

在项目中用到@Transactional注解实现事务是必须滴，如果你还在用xml配置，那我只能说……。

但是有时候我们会发现在方法上加了@Transactional注解却出现灵异事件，在方法内出现异常，数据还是插入到数据库，没有回滚，事务哪里去了，明明是加了的。

**@Async注解实现原因分析和解决方案**

在看下面的内容之前，对动态代理不是很熟悉的可以看一下我之前的一篇文章：[http://blog.zdydoit.com/blogs/2018/06/dynamic-static-proxy/](http://blog.zdydoit.com/blogs/2018/06/dynamic-static-proxy/)。
这里添加的注解是通过Spring AOP对方法的一种增强，而Spring AOP的原理就是动态代理，他的代理有两种，分别是CGLB和JDK自带的代理，Spring AOP会根据具体的实现不同，采用不同的代理方式。
动态代理的原理了解了，下面的问题就可以很好的理解。

**异步测试接口**


``` java
public interface AsyncAopService {
   void addOrder();
   void sendMsg(int result);
}
```

**异步测试接口实现**
``` java
@Service
@Slf4j
public class AsyncAopServiceImpl implements AsyncAopService {

   @Autowired
   private OrderDao orderDao;
   @Autowired
   private MsgDao msgDao;

   /**
    * 添加订单后会给用户异步的推送信息
    */
   @Transactional //这里为了让该被代理，加此注解
   public void addOrder() {
       int result = orderDao.insert(OrderModel.builder()
               .amount(10000L)
               .orderId("ORDER_2018042601")
               .phone("15600001212")
               .userId("U_001")
               .build());
       String currentThreadName = Thread.currentThread().getName();
       sendMsg(result);
       System.out.println(currentThreadName + "------>下单结束：mark");
   }

   @Async
   public void sendMsg(int result) {
       try {
           Thread thread = Thread.currentThread();
           thread.sleep(3000);//停留3秒
           String currentThreadName = thread.getName();
           if (result == 1) {
               msgDao.insert(MsgModel.builder().msgContent("下单成功！").receiver("15600001212").build());
               System.out.println(currentThreadName + "------>发送信息成功");
           } else {
               msgDao.insert(MsgModel.builder().msgContent("下单失败！").receiver("15600001212").build());
               System.out.println(currentThreadName + "------>发送信息失败");
           }
       } catch (Exception e) {
           e.printStackTrace();
       }
   }
}
```

**测试类**
``` java
@Test
public void AsyncTest() throws InterruptedException {
 System.out.println("=======async test start=======");
 asyncAopService.addOrder();
 System.out.println("=======async test end=======");
 /**
  * 在这里让线程睡5秒的原因是为了能够看到异步执行的结果日志
  * 小知识点：在Junit测试中，如果主线程执行结束，
  *整个测试过程也结束了，存在的异步逻辑如果没有执行完就不会执行啦!
  * 测试方式：把这行代码去掉，执行测试，t_order_info表中会插入数据，
  *但是t_msg_info表中无数据插入。
  */
 Thread.sleep(5000);
}
```

**测试类上的注解：**
``` java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
```

**Spring Boot启动类的注解：**
``` java
@SpringBootApplication
@MapperScan("com.minuor.aop.dao")
@EnableAspectJAutoProxy
@EnableAsync //开启异步功能
```

**测试结果**
``` java
=======async test start=======
main------>发送信息成功
main------>下单结束：mark
=======async test end=======
```

运行的过程很慢，两行日志的线程名称相同，并且mark日志是在发送信息成功后再输出的，回到代码可以知道发送信息逻辑是异步的执行，为什么会和下单过程的线程名称相同，并且异步执行发送消息是延迟的，为何日志还在mark日志前。种种迹象表明，这不是一个异步执行，而是顺序执行。但是这里是加了异步的注解的，实际没有生效。
在`addOrder()`方法里面直接调用`sendMsg(……)`方法，这里还隐含一个关键字，那就是`this`，实际上这里调用是这样的：`this.sendMsg()`,`this`是当前对象。而`addOrder()`是被代理的，在代理对象中执行结束增强后，通过`invoke`，用实际`AsyncAopServiceImpl`对象来调用`addOrder()`方法执行业务逻辑。在业务逻辑内又调用了`sendMsg(……)`方法，调用的对象是当前对象，当前对象是`AsyncAopServiceImpl`，问题就出在这里，因为要想用异步执行`sendMsg(……)`，必须用代理对象执行，因为代理对象要做异步相关的增强，但是此时却直接用AsyncAopServiceImpl对象调用，绕过了代理对象增强的部分，也就是说代理增强部分失效，`@Async`注解失效。原来想异步执行的逻辑，变成了顺序执行。

**解决方案**

没有用代理对象执行`sendMsg(……)`，被`AsyncAopServiceImpl`对象抢占了先机。那么解决就是要让代理对象来执行`sendMsg(……)`。
在调用`sendMsg(……)`之前添加下面的代码

``` java
AsyncAopService service = (AsyncAopService) AopContext.currentProxy(); 
//获取代理对象
service.sendMsg(result); 
//通过代理对象调用sendMsg，做异步增强
```
这里还不算完，如果就这样运行，那肯定会报错。
**在@EnableAspectJAutoProxy添加属性值。**
> @EnableAspectJAutoProxy(exposeProxy = true)

运行结果
``` java
=======async test start=======
main------>下单结束：mark
=======async test end=======
SimpleAsyncTaskExecutor-1------>发送信息成功
```

结果也是想要的结果，下单结束，整个测试结束，在测试结束后等待5秒，等待异步日志打印。主线程和异步线程是不同的两个。

如果对代理对象和当前对象有点懵的话，可以加上下面的两行代码
``` java
System.out.println("------>代理对象："+service.getClass());
System.out.println("------>当前对象："+this.getClass());
```
得到的结果：
``` java
------>代理对象：class com.minuor.aop.impl.AsyncAopServiceImpl$$EnhancerBySpringCGLIB$$9de92f4b //可以看出来是CGLB动态代理
------>当前对象：class com.minuor.aop.impl.AsyncAopServiceImpl
```

**@Transactional注解失效的原因分析**

这个原因和上面的是相同的，代理被绕过，直接当前对象执行应该被增强的方法，导致方法没有被增强成功。但是可以说一下两个情况。

情况一：在非代理增强方法中调用加了`@Transactional`增强的方法
这个过程容易理解，不解释。

业务代码

``` java
@Service
public class TransactionalAopServiceImpl implements TransactionalAopService {

   @Autowired
   private OrderDao orderDao;
   @Autowired
   private UserDao userDao;

   public void addOrder() {
       orderDao.insert(OrderModel.builder()
               .userId("YK_002") //游客编号
               .phone("13522203330")
               .orderId("ORDER_2018042602")
               .amount(10000L)
               .build());
       //默开用户
       System.out.println("--->"+this.getClass());
       addUser("13522203330");
   }

   @Transactional
   public void addUser(String phone) {
       userDao.insert(UserModel.builder().userName("zhangsan").userPhone(phone).build());
       throw new RuntimeException();
   }
}
```

运行结果是order订单信息添加成功，同时user用户信息也添加成功，数据库都有数据，没有回滚。按照表面理解应该是order添加成功，user添加失败，因为addUser上加了事务，会回滚。原理参照@Async失效的原理解释。

**情况二：addOrder和addUser方法上都添加@Transactional**

这种情况下，是可以回滚的，但是不太清楚是在哪个事务回滚，也不太清楚`@Transactional`是都有效，还是其中一个有效。但是可以模拟，那就是定义三个异常，分别是`OrderException`、`UserException`、`OtherException`，然后在两个方法上指定回滚异常类。通过抛出不同的异常来看具体的结果。

`@Transactional`修改
``` java
@Transactional(rollbackFor = OrderException.class, noRollbackFor = RuntimeException.class) 
//addOrder方法上
@Transactional(rollbackFor = UserException.class, noRollbackFor = RuntimeException.class) 
//addUser方法上
```

**执行结果分析**

1、抛`OtherException`异常，没有回滚，`order`、`user`数据都成功录入到数据库中；
2、抛`UserException`异常，没有回滚，`order`、`user`数据都成功录入到数据库中，这里可以看的出来`addUser`方法上的`@Transactional`注解是无效的；
3、抛`OrderException`异常，回滚成功，`order`、`user`数据都没有录入到数据库中，`addOrder`方法上的`@Transactional`有效。

这样的结果加上动态代理原理的分析不难得出结果，`addUser`方法的代理增强被绕过，只是普通的一个方法调用，而且这个方法是包含在`addOrder`方法事务内的。

