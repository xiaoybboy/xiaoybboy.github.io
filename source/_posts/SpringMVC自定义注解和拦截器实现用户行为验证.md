---
title: SpringMVC 使用HandlerMethodArgumentResolver自定义解析器实现请求数据绑定方法入参
date: 2017-08-07 18:59:23
tags: [springmvc,自定义参数解析器]
categories: spring
---
# 问题
首先，我们遇到的问题是，当我们需要在controller中频繁的从session中获取数据，比如向下面这样
在controller中需要从session中获取user对象，那么可能你会想到在controller里面或者其他类里面写这样的代码，然后在controller里面调用….
<!--more-->
```java
	public User getLoginUser(HttpServletRequest request) {  
	        HttpSession session = request.getSession();  
	        return (User) session.getAttribute("user");  
	}  
```
总感觉特别的不好…
现在如果我们看了下面介绍的HandlerMethodArgumentResolver自定义解析器实现的请求数据绑定方法入参，你就会看到像下面的代码只需要一个注解就能解决上面的问题↓
```java
	@RequestMapping("/index")  
	public String index(@MyUser User user,ModelMap modelMap){  
	    logger.info(user.getUsername()+"---------------------------");  
	    return "login";  
	}  
```
# 用HandlerMethodArgumentResolver解决
首先，我们需要知道一点的就是SpringMVC的工作流程，SpringMVC的DispatchServlet会根据请求来找到对应的HandlerMapping，最终spring会选择用RequestMappingHandlerMapping，然后根据RequestMappingHandlerMapping来获取HandlerMethod，然后来找支持的HandlerMethodArgumentResolver来处理对应controller的方法的入参。
首先，我们需要做的就是创建一个Annotation
```java
	@Target(ElementType.PARAMETER)  
	@Retention(RetentionPolicy.RUNTIME)  
	@Documented 
	public @interface MyUser {  
	  String value() default "";  
	}  
```
然后我们要做的就是创建一个MyUserMethodArgumentResolver这个类来实现HandlerMethodArgumentResolver这个接口
```java
	public class MyUserMethodArgumentResolver implements HandlerMethodArgumentResolver {  
	@Override  
	public boolean supportsParameter(MethodParameter methodParameter) {  
	      return methodParameter.hasParameterAnnotation(ManyUser.class);  
	}  
	@Override  
	public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer   modelAndViewContainer, NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {  
8.	         //直接返回request域对象的user
9.	        return nativeWebRequest.getAttribute("user", NativeWebRequest.SCOPE_REQUEST);;  
10.	        //或者这里你也可以直接返回自己创建的User对象 
11.	        /* 
12.	            User user = new User(); 
13.	            user.setUsername("yangpeng"); 
14.	            return user; 
15.	          */  
16.	         /**自定义实现，比如参数是很多用户：如    custom?names=lyncc,fly,ted&ids=1,2,3
17.			   List<User> users = new ArrayList<User>();  
18.	        String names = (String)webRequest.getParameter("names");  
19.	        String ids = (String)webRequest.getParameter("ids");  
20.	        if(null != names && null != ids){  
21.	            String[] nameStrs = names.trim().split(",");  
22.	            String[] idStrs = ids.trim().split(",");  
23.	            for(int i = 0;i<nameStrs.length;i++){  
24.	             User user = new User(Integer.parseInt(idStrs[i]), nameStrs[i]);
25.	                users.add(user);  
26.	            }  
27.	        }  
28.	        return users;  //返回list 例如注解式 @manyUser List<User> users
29.	    }         
30.	}  }  
```
Spring默认会注册多个HandlerMethodArgumentResolver来处理不同的请求，Spring会根据HandlerMethodArgumentResolver的supportsParameter()方法来判断是否支持处理当前请求。 
第一个supportsParameter方法是判断这个MyUserMethodArgumentResolver是否支持传入的MethodParameter对象。 
第二个resolveArgument方法是处理具体的需要绑定到方法入参，返回的对象就是需要绑定的对象，这里我是直接从session里面获取了一个user的对象直接返回，或者你也可以在这里直接创建一个User对象然后返回用于测试是一样的。
接下来就是在spring-mvc.xml中配置了
```xml
	<mvc:annotation-driven>  
	   <mvc:argument-resolvers>  
	        <bean class="com.yp.code.common.bind.method.MyUserMethodArgumentResolver"></bean>  
	    </mvc:argument-resolvers>  
	</mvc:annotation-driven> 
``` 
最后就是使用创建的@MyUser这个Annotation来让SpringMVC自动的帮你绑定到Controller的方法里面了
```java
	@RequestMapping("/index")  
	public String index(@MyUser User user,ModelMap modelMap){  
	    System.out.println(user.getUsername());  
	    return "login";  
	} 
``` 
这样就非常优雅的解决了上面的问题。
