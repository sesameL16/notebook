**AOP** **的开发中的相关术语****:** 

Joinpoint(连接点):所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是方法,因为 spring 只 支持方法类型的连接点. 

Pointcut(切入点):所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义. 

Advice(通知/增强):所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知.通知分为前置通知,后置 通知,异常通知,最终通知,环绕通知(切面要完成的功能) 

Introduction(引介):引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类 动态地添加一些方法或 Field. 

Target(目标对象):代理的目标对象 

Weaving(织入):是指把增强应用到目标对象来创建新的代理对象的过程. spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装在期织入 

Proxy（代理）:一个类被 AOP 织入增强后，就产生一个结果代理类 Aspect(切面): 是切入点和通知（引介）的结合 

**Spring** **使用 AspectJ 进行 AOP 的开发：XML 的方式** 

引入相应的 jar 包 

\* spring 的传统 AOP 的开发的包 spring-aop-4.2.4.RELEASE.jar com.springsource.org.aopalliance-1.0.0.jar

\* aspectJ 的开发包: com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar spring-aspects-4.2.4.RELEASE.jar 

 

引入 Spring 的配置文件 

引入 AOP 约束: 

<beans xmlns="http://www.springframework.org/schema/beans"   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">  

 

</beans> 

 

编写目标类 

创建接口和类: 

public interface OrderDao { 

public void save(); 

public void update(); 

public void delete(); 

public void find(); 

} 

 

public class OrderDaoImpl implements OrderDao { 

 

 @Override public void save() { 

​     System.out.println("保存订单..."); 

 } 

 

 @Override public void update() { 

​     System.out.println("修改订单..."); 

 } 

 

 @Override public void delete() { 

​     System.out.println("删除订单..."); 

 } 

 

 @Override public void find() { 

​     System.out.println("查询订单..."); 

 } 

} 

 

目标类的配置 

 <!-- 目标类=== --> 

 <bean id="orderDao" class="cn.itcast.spring.demo3.OrderDaoImpl">  </bean> 

 

整合 Junit 单元测试 

引入 spring-test.jar

 

@RunWith(SpringJUnit4ClassRunner.class)

@ContextConfiguration("classpath:applicationContext.xml")

public class SpringDemo3 { 

@Resource(name="orderDao") 

private OrderDao orderDao;  

 

@Test 

public void demo1(){  

orderDao.save();  

orderDao.update();  

orderDao.delete();  

orderDao.find(); 

} 

} 

 

通知类型

前置通知 ：在目标方法执行之前执行. 

后置通知 ：在目标方法执行之后执行 

环绕通知 ：在目标方法执行前和执行后执行 

异常抛出通知：在目标方法执行出现 异常的时候 执行 

最终通知 ：无论目标方法是否出现异常 最终通知都会 执行. 1.2.7.7 切入点表达式 

 

execution(表达式) 表达式: [方法访问修饰符] 方法返回值 包名.类名.方法名(方法的参数) 

public * cn.itcast.spring.dao.*.*(..) 

\* cn.itcast.spring.dao.*.*(..) 

\* cn.itcast.spring.dao.UserDao+.*(..) 

\* cn.itcast.spring.dao..*.*(..) 

 

编写一个切面类 

public class MyAspectXml { 

// 前置增强 

public void before(){ 

​    System.out.println("前置增强==========="); 

} 

} 

 

配置完成增强 

 <!-- 配置切面类 --> 

<bean id="myAspectXml" class="cn.itcast.spring.demo3.MyAspectXml"></bean>  

 <!-- 进行 aop 的配置 --> 

<aop:config> 

 <!-- 配置切入点表达式:哪些类的哪些方法需要进行增强 -->  

<aop:pointcut expression="execution(* cn.itcast.spring.demo3.OrderDao.save(..))" id="pointcut1"/> 

 <!-- 配置切面 -->  

<aop:aspect ref="myAspectXml">  

<aop:before method="before" pointcut-ref="pointcut1"/>  

</aop:aspect> 

</aop:config> 

 

其他的增强的配置： 

 <!-- 配置切面类 --> 

<bean id="myAspectXml" class="cn.itcast.spring.demo3.MyAspectXml"></bean>  

 <!-- 进行 aop 的配置 --> 

<aop:config> 

 <!-- 配置切入点表达式:哪些类的哪些方法需要进行增强 -->  

<aop:pointcut expression="execution(* cn.itcast.spring.demo3.*Dao.save(..))" id="pointcut1"/>  

<aop:pointcut expression="execution(* cn.itcast.spring.demo3.*Dao.delete(..))" id="pointcut2"/>  

<aop:pointcut expression="execution(* cn.itcast.spring.demo3.*Dao.update(..))" id="pointcut3"/>  

<aop:pointcut expression="execution(* cn.itcast.spring.demo3.*Dao.find(..))" id="pointcut4"/> 

 <!-- 配置切面 -->  

<aop:aspect ref="myAspectXml">  

<aop:before method="before" pointcut-ref="pointcut1"/>  

<aop:after-returning method="afterReturing" pointcut-ref="pointcut2"/>  

<aop:around method="around" pointcut-ref="pointcut3"/>  

<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut4"/>  

<aop:after method="after" pointcut-ref="pointcut4"/>  

</aop:aspect> 

</aop:config> 

 

 

 

**Spring** **使用 AspectJ 进行 AOP 的开发：注解的方式** 

 

配置目标类： 

​     <!-- 目标类============ -->   

<bean id="productDao" class="cn.itcast.spring.demo4.ProductDao"></bean>  

 

开启 aop 注解的自动代理: 

<aop:aspectj-autoproxy/>

 

AspectJ 的 AOP 的注解: 

@Aspect:定义切面类的注解 

 

通知类型:   

\* @Before  :前置通知   

\* @AfterReturing :后置通知   

\* @Around  :环绕通知   

\* @After  :最终通知   

\* @AfterThrowing :异常抛出通知. 

 

@Pointcut:定义切入点的注解 1.2.1.7 编写切面类: 

@Aspect 

public class MyAspectAnno { 

 

 @Before("MyAspectAnno.pointcut1()") 

public void before(){ 

 System.out.println("前置通知==========="); 

}  

 

@Pointcut("execution(* cn.itcast.spring.demo4.ProductDao.save(..))") 

private void pointcut1(){} }

 

配置切面: 

​     <!-- 配置切面类 -->   

<bean id="myAspectAnno" class="cn.itcast.spring.demo4.MyAspectAnno"></bean>  

 

其他通知的注解: 

@Aspect 

public class MyAspectAnno { 

 

 @Before("MyAspectAnno.pointcut1()") 

public void before(){ 

​    System.out.println("前置通知==========="); 

 }  

 

@AfterReturning("MyAspectAnno.pointcut2()") 

public void afterReturning(){

​    System.out.println("后置通知==========="); 

}  

 

@Around("MyAspectAnno.pointcut3()") 

public Object around(ProceedingJoinPoint joinPoint) throws Throwable{ 

 System.out.println("环绕前通知==========");  

 Object obj = joinPoint.proceed(); 

 System.out.println("环绕后通知==========");  

 return obj; 

}  

 

@AfterThrowing("MyAspectAnno.pointcut4()") 

public void afterThrowing(){

​     System.out.println("异常抛出通知========"); 

}  

 

@After("MyAspectAnno.pointcut4()") 

public void after(){ 

​     System.out.println("最终通知=========="); 

}  

 

@Pointcut("execution(* cn.itcast.spring.demo4.ProductDao.save(..))") 

private void pointcut1(){} 

@Pointcut("execution(* cn.itcast.spring.demo4.ProductDao.update(..))") 

private void pointcut2(){} 

@Pointcut("execution(* cn.itcast.spring.demo4.ProductDao.delete(..))") 

private void pointcut3(){} 

@Pointcut("execution(* cn.itcast.spring.demo4.ProductDao.find(..))") 

private void pointcut4(){} 

} 

 

 