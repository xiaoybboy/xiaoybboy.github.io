---
title: SpringMVC自定义注解和拦截器实现用户行为验证
date: 2017-08-01 17:30:39
tags: [springmvc,自定义注解]
categories: spring
---
最近在进行项目开发的时候需要对接口做Session验证

# 自定义一个注解@AuthCheckAnnotation
```java
@Documented
@Target(ElementType.METHOD)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface AuthCheckAnnotation {
    boolean check() default false;//默认不需要进行验证
}
```
<!--more-->
# 定义一个相应的拦截器，在springMVC配置文件中进行配置
拦截器：
&emsp;spring为我们提供了org.springframework.web.servlet.handler.HandlerInterceptorAdapter这个适配器，继承此类，可以非常方便的实现自己的拦截器。可以根据我们的需要重写preHandle、postHandle、afterCompletio方法。
分别实现预处理、后处理（调用了Service并返回ModelAndView，但未进行页面渲染）、返回处理（已经渲染了页面）
1. 在preHandle中，可以进行编码、安全控制等处理；
2. 在postHandle中，有机会修改ModelAndView；
3. 在afterCompletion中，可以根据ex是否为null判断是否发生了异常，进行日志记录。
```java
public class AuthCheckInteceptor extends HandlerInterceptorAdapter {
    @Autowired
    UserInfoService userInfoService ;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        
        HandlerMethod methodHandler=(HandlerMethod) handler;
        AuthCheckAnnotation auth=methodHandler.getMethodAnnotation(AuthCheckAnnotation.class);
　　　　　//如果方法中添加了@AuthCheckAnnotation 这里的auth不为null
　　　　　//如果@AuthCheckAnnotation(check=false) 这里的auth为false,即不用进行拦截验证，@AuthCheckAnnotation默认为前面定义的true　　
        if(auth!=null&&!auth.check()){
           return true;  
        }
        UserInfo user=(UserInfo)request.getSession().getAttribute(Constants.SESSION_USER);
        try {
            userInfoService.login(request, user);
            return true;
        } catch (Exception e) {
            request.getRequestDispatcher("login.do").forward(request, response);
            return false;
        }
    }

}
```
在springMVC.xml文件中添加拦截器
```xml
    <mvc:interceptors>
            <mvc:interceptor>
            <mvc:mapping path="/*.do"  />
            <bean  class="com.party.common.interceptor.AuthCheckInteceptor"/>        
        </mvc:interceptor>
    </mvc:interceptors>
```
# 在springMVC controller中使用实例
```java
    @AuthCheckAnnotation(check=true)
    @RequestMapping("doLogin")
    @ResponseBody
    public Object doLogin(HttpServletRequest request,HttpServletResponse response){
        .......
        return RetMessage.toJson(responseBody);
    }
```
转：https://www.bbsmax.com/A/QV5Z1jwbJy/