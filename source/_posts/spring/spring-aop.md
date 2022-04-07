---
title: spring-aop实例(1)
date: 2022-03-28 17:32:08
tags: spring
---
## Spring aop的实现原理(1)
### 1 创建Spring的aop实例
1 依据Spring源码 创建一个debug项目
  1.1创建一个Aspect
```xml
@Component
@Aspect
public class UserAspect {

	@Pointcut("execution(* com.debug.service.*.*(..))")
	public void pointCutExecution(){}

	@Before("com.debug.aop.UserAspect.pointCutExecution()")
	public void before(){
		System.out.println("--------before--------");
	}

	@After("com.debug.aop.UserAspect.pointCutExecution()")
	public void after(){
		System.out.println("--------after--------");
	}
}

```
 1.2 创建一个service 去执行对应的方法
```xml
   @Service("UserServiceImpl")
public class UserServiceImpl {
	
	public void add() {
		System.out.println("================");
	}

	public String get(String name) {
		return name;
	}
}
```

 1.3 在resource 目录下创建一个application.xml文件
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

	<bean id="student" class="com.debug.bean.Student">
		<property name="id" value="1"/>
		<property name="name" value="zhangsan"/>
	</bean>

	<bean id="UserServiceImpl" class="com.debug.service.UserServiceImpl">

	</bean>

	<aop:aspectj-autoproxy proxy-target-class="true"/>
	<bean class="com.debug.aop.UserAspect"></bean> // 这边需要创建一个bean 不然BeanFactory加载不到 

</beans>

 ```

 1.4 创建一个main方法去调用实现aop的效果
 ```xml
  public class MyApplication {
	public static void main(String[] args) {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath*:application.xml");
		UserServiceImpl userService =(UserServiceImpl) context.getBean("UserServiceImpl");
		userService.add();
	}
}

 ```
 1.5 至此运行main方法就可以看见aop的效果了

![](/../../static/spring/spring-aop.jpg)



