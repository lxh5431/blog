---
title: 面向切面的AOP编程
date: 2016-6-15
tags: 编程
categories: 面向对象
---


面向切面
##面向切面编程
###定义
Aspect Oriented Programming 面向切面编程，使业务和逻辑的解耦，让程序员更加专注于业务，其他交给切面编程来实现
要点
1） Aspect ：切面，方法调用的横切面，切入系统的一个切面。比如事务管理是一个切面，权限管理也是一个切面
静态切入点
动态切入点
自定义切入点
2） Join point ：连接点，也就是可以进行横向切入的位置
3） Advice ：通知，切面在某个连接点执行的操作
<!--more-->
(分为: Before advice , After returning advice , After throwing advice , After (finally) advice , Around advice )
Before advice
public class LogBefore implements MethodBeforeAdvice {
private Logger logger = Logger.getLogger(this.getClass().getName());
public void before(Method method, Object[] args, Object target) throws Throwable {
    logger.log(Level.INFO, args[0] + " 开始审核数据....");
}
}
After returning advice
ublic class LogAfter implements AfterReturningAdvice {
private Logger logger=Logger.getLogger(this.getClass().getName());
@Override
public void afterReturning(Object object, Method method, Object[] args, Object target) throws Throwable {
// TODO Auto-generated method stub
logger.log(Level.INFO, args[0]+”审核数据完成…”);
}
4） Pointcut ：切点，符合切点表达式的连接点，也就是真正被切入的地方
AOP是OOP的有效补充点，让系统的设计更加完善和更加容易维护
AOPd 两种代理

``` bash
$ java动态代理
```

public class LogProxy implements InvocationHandler {
private Logger logger = Logger.getLogger(this.getClass().getName());
private Object delegate;
public Object bind(Object delegate) {
    this.delegate = delegate;
    return Proxy.newProxyInstance(delegate.getClass().getClassLoader(), delegate.getClass().getInterfaces(), this);
}

public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    Object result = null;
    try {
        logger.log(Level.INFO, args[0] + " 开始审核数据....");
        result = method.invoke(delegate, args);
        logger.log(Level.INFO, args[0] + " 审核数据结束....");
    } catch (Exception e){
        logger.log(Level.INFO, e.toString());
    }
    return result;
}
CGLIB代理

``` bash
$ CGLIB代理
```

注入
package net.aazj.aop;
import java.lang.reflect.Method;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
public class CGProxy implements MethodInterceptor{
private Object target; // 被代理对象
public CGProxy(Object target){
this.target = target;
}
public Object intercept(Object arg0, Method arg1, Object[] arg2, MethodProxy proxy) throws Throwable {
System.out.println(“do sth before….”);
Object result = proxy.invokeSuper(arg0, arg2);
System.out.println(“do sth after….”);
return result;
}
public Object getProxyObject() {
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(this.target.getClass()); // 设置父类
// 设置回调
enhancer.setCallback(this); // 在调用父类方法时，回调 this.intercept()
// 创建代理对象
return enhancer.create();
}
}
自动代理






<bean id="exceptionHandlereAdvisor"
    class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="exceptionHandler" />
    </property>
    <property name="patterns">
        <value>.*.*</value>
    </property>
</bean>
动态代理
## java动态代理
主要使用到 InvocationHandler 接口和 Proxy.newProxyInstance() 方法。 JDK动态代理要求被代理实现一个接口，只有接口中的方法才能够被代理 。其方法是将被代理对象注入到一个中间对象，而中间对象实现InvocationHandler接口，在实现该接口时，可以在 被代理对象调用它的方法时，在调用的前后插入一些代码。而 Proxy.newProxyInstance() 能够利用中间对象来生产代理对象。插入的代码就是切面代码。所以使用JDK动态代理可以实现AOP。
#### 例子
被代理对象实现的接口，只有接口中的方法才能够被代理：
public interface UserService {
public void addUser(User user);
public User getUser(int id);
}
实现类
public class UserServiceImpl implements UserService {
public void addUser(User user) {
System.out.println(“add user into database.”);
}
public User getUser(int id) {
User user = new User();
user.setId(id);
System.out.println(“getUser from database.”);
return user;
}
}

``` bash
$ 代理中间类
```

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
public class ProxyUtil implements InvocationHandler {
private Object target; // 被代理的对象
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
System.out.println(“do sth before….”);
Object result = method.invoke(target, args);
System.out.println(“do sth after….”);
return result;
}
ProxyUtil(Object target){
this.target = target;
}
public Object getTarget() {
return target;
}
public void setTarget(Object target) {
this.target = target;
}
}
#### 测试
``` bash
$ 测试
```

import java.lang.reflect.Proxy;
import net.aazj.pojo.User;
public class ProxyTest {
public static void main(String[] args){
Object proxyedObject = new UserServiceImpl(); // 被代理的对象
ProxyUtil proxyUtils = new ProxyUtil(proxyedObject);
// 生成代理对象，对被代理对象的这些接口进行代理：UserServiceImpl.class.getInterfaces()
UserService proxyObject = (UserService) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
UserServiceImpl.class.getInterfaces(), proxyUtils);
proxyObject.getUser(1);
proxyObject.addUser(new User());
}
}
### 总结：
通过对面向切面编程的学习，对动态代理的使用有更加深入的了解，对解耦的设计和软件设计的思想有进一步的了解，对动态代理的设计模式的重要性进一步的了解
面向切面的编程就是对日记和时间等非业务的编程在切面插入，运用代理的设计模式更好的把业务和逻辑的解耦，改变前一个的日记并不会影响业务的运行。
