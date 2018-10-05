---
title: SpringMVC拦截器
date: 2018-10-5 14:42:35
tags: [spring,拦截器]
categories: spring
---
{% note success %} Spring MVC中的拦截器（Interceptor）类似于Servlet中的过滤器（Filter），它主要用于拦截用户请求并作相应的处理。通过拦截器可以进行权限验证、记录请求信息的日志、判断用户是否登录等。 {% endnote %}
<!--more-->
{% note info %} SpringMVC 中的Interceptor 拦截请求是通过HandlerInterceptor 来实现的。
在SpringMVC 中定义一个Interceptor 主要有两种方式.
1. 第一种方式是要定义的Interceptor类要实现了Spring 的HandlerInterceptor 接口，或者是这个类继承实现了HandlerInterceptor 接口的类，如Spring已经提供的实现了HandlerInterceptor 接口的抽象类HandlerInterceptorAdapter;
2. 第二种方式是实现Spring的WebRequestInterceptor接口，或者是继承实现了WebRequestInterceptor的类。{% endnote %}
## 一.实现HandlerInterceptor接口
HandlerInterceptor 接口中定义了三个方法，我们就是通过这三个方法来对用户的请求进行拦截处理的。流程：
![](http://i.imgur.com/GGUBVhn.png)
![](https://i.imgur.com/8i84ykU.jpg)
注意postHandle方法的执行顺序与定义顺序正好相反。
接口实现如下：
```java
public class SpringMVCInterceptor implements HandlerInterceptor {  
    /** 
     * preHandle方法是进行处理器拦截用的，该方法将在Controller处理之前进行调用，SpringMVC中的Interceptor拦截器是链式的，可以同时存在 
     * 多个Interceptor，然后SpringMVC会根据声明的前后顺序一个接一个的执行，而且所有的Interceptor中的preHandle方法都会在 
     * Controller方法调用之前调用。SpringMVC的这种Interceptor链式结构也是可以进行中断的，这种中断方式是令preHandle的返 
     * 回值为false，当preHandle的返回值为false的时候整个请求就结束了。 
     */  
    @Override  
    public boolean preHandle(HttpServletRequest request,  
            HttpServletResponse response, Object handler) throws Exception {  
        // TODO Auto-generated method stub  
        return false;  
    }  
      
    /** 
     * 这个方法只会在当前这个Interceptor的preHandle方法返回值为true的时候才会执行。postHandle是进行处理器拦截用的，它的执行时间是在处理器进行处理之 
     * 后，也就是在Controller的方法调用之后执行，但是它会在DispatcherServlet进行视图的渲染之前执行，也就是说在这个方法中你可以对ModelAndView进行操 
     * 作。这个方法的链式结构跟正常访问的方向是相反的，也就是说先声明的Interceptor拦截器该方法反而会后调用。 
     */  
    @Override  
    public void postHandle(HttpServletRequest request,  
            HttpServletResponse response, Object handler,  
            ModelAndView modelAndView) throws Exception {  
        // TODO Auto-generated method stub   
    }  
  
    /** 
     * 该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行。该方法将在整个请求完成之后，也就是DispatcherServlet渲染了视图执行， 
     * 这个方法的主要作用是用于清理资源的，当然这个方法也只能在当前这个Interceptor的preHandle方法的返回值为true时才会执行。 
     */  
    @Override  
    public void afterCompletion(HttpServletRequest request,  
            HttpServletResponse response, Object handler, Exception ex)  
    throws Exception {  
        // TODO Auto-generated method stub   
    }  
      
} 
```
1. preHandle：预处理回调方法，实现处理器的预处理（如登录检查），第三个参数为响应的处理器（如我们上一章的Controller实现）；
返回值：true表示继续流程（如调用下一个拦截器或处理器）;
false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；
2. postHandle：后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。
3. afterCompletion：整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion。
{% note primary %}在实际应用中，一般都是通过实现HandlerInterceptor接口或者继承HandlerInterceptorAdapter抽象类，复写preHandle()、postHandle()和afterCompletion()这 3 个方法来对用户的请求进行拦截处理的。{% endnote %}

## 二.WebRequestInterceptor 接口
在WebRequestInterceptor接口中也定义了 3 个方法，同HandlerInterceptor接口完全相同，也是通过复写这 3 个方法来用户的请求进行拦截处理的。而且这 3 个方法都传递了同一个参数 WebRequest，这个 WebRequest 是 Spring 中定义的一个接口，它里面的方法定义跟 HttpServletRequest 类似，在WebRequestInterceptor中对 WebRequest 进行的所有操作都将同步到 HttpServletRequest 中，然后在当前请求中依次传递。
在 Spring 框架之中，还提供了一个和WebRequestInterceptor接口长的很像的抽象类，那就是：WebRequestInterceptorAdapter，其实现了AsyncHandlerInterceptor接口，并在内部调用了WebRequestInterceptor接口。
接下来，咱们主要讲一下WebRequestInterceptor接口的 3 个函数：
1. preHandle(WebRequest request)方法，该方法在请求处理之前进行调用，即在 Controller 中的方法调用之前被调用。这个方法跟 HandlerInterceptor 中的 preHandle 不同，主要区别在于该方法的返回值是void 类型的，也就是没有返回值，因此我们主要用它来进行资源的准备工作。在这里，进一步说说 setAttribute 方法的第三个参数 scope ，该参数是一个Integer 类型的。在 WebRequest 的父层接口 RequestAttributes 中对它定义了三个常量，分别为： 
- SCOPE_REQUEST ，它的值是 0，表示只有在 request 中可以访问。
- SCOPE_SESSION，它的值是1，如果环境允许的话，它表示的是一个局部的隔离的 session，否则就代表普通的 session，并且在该 session 范围内可以访问。
- SCOPE_GLOBAL_SESSION，它的值是 2，如果环境允许的话，它表示的是一个全局共享的 session，否则就代表普通的 session，并且在该 session 范围内可以访问。
2. postHandle(WebRequest request, ModelMap model)方法，该方法在请求处理之后，也就是在 Controller 中的方法调用之后被调用，但是会在视图返回被渲染之前被调用，所以可以在这个方法里面通过改变数据模型 ModelMap 来改变数据的展示。该方法有两个参数，WebRequest 对象是用于传递整个请求数据的，比如在 preHandle 中准备的数据都可以通过 WebRequest 来传递和访问；ModelMap 就是 Controller 处理之后返回的 Model 对象，咱们可以通过改变它的属性来改变返回的 Model 模型。
3. afterCompletion(WebRequest request, Exception ex)方法，该方法会在整个请求处理完成，也就是在视图返回并被渲染之后执行。因此可以在该方法中进行资源的释放操作。而 WebRequest 参数就可以把咱们在 preHandle 中准备的资源传递到这里进行释放。Exception 参数表示的是当前请求的异常对象，如果在 Controller 中抛出的异常已经被 Spring 的异常处理器给处理了的话，那么这个异常对象就是是 null.
```java
import org.springframework.ui.ModelMap;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.context.request.WebRequestInterceptor;

public class WrongCodeInterceptor implements WebRequestInterceptor {

    @Override
    public void preHandle(WebRequest request) throws Exception {
        System.out.println("WrongCodeInterceptor, preHandle......");
    }

    @Override
    public void postHandle(WebRequest request, ModelMap model) throws Exception {
        System.out.println("WrongCodeInterceptor, postHandle......");
    }

    @Override
    public void afterCompletion(WebRequest request, Exception ex) throws Exception {
        System.out.println("WrongCodeInterceptor, afterCompletion......");
    }
} 
```
## 拦截器的配置
```xml
<mvc:interceptors>  
    <!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
    <bean class="com.host.app.web.interceptor.AllInterceptor"/>  
    <mvc:interceptor>  
        <mvc:mapping path="/test/number.do"/>  
        <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->  
        <bean class="com.host.app.web.interceptor.LoginInterceptor"/>  
    </mvc:interceptor>  
</mvc:interceptors>
```
## 实例1：SpringMVC拦截器实现登录认证
控制器
```java
/** 
 * 登录认证的控制器 
 */  
@Controller  
public class LoginControl {  
  
    /** 
     * 登录 
     * @param session 
     *          HttpSession 
     * @param username 
     *          用户名 
     * @param password 
     *          密码 
     * @return 
     */  
    @RequestMapping(value="/login")  
    public String login(HttpSession session,String username,String password) throws Exception{        
        //在Session里保存信息  
        session.setAttribute("username", username);  
        //重定向  
        return "redirect:hello.action";   
    }  
      
    /** 
     * 退出系统 
     * @param session 
     *          Session 
     * @return 
     * @throws Exception 
     */  
    @RequestMapping(value="/logout")  
    public String logout(HttpSession session) throws Exception{  
        //清除Session  
        session.invalidate();  
          
        return "redirect:hello.action";  
    }       
} 
```   
拦截器：
```java
/** 
 * 登录认证的拦截器 
 */  
public class LoginInterceptor implements HandlerInterceptor{  
  
    /** 
     * Handler执行完成之后调用这个方法 
     */  
    public void afterCompletion(HttpServletRequest request,  
            HttpServletResponse response, Object handler, Exception exc)  
            throws Exception {  
          
    }  
  
    /** 
     * Handler执行之后，ModelAndView返回之前调用这个方法 
     */  
    public void postHandle(HttpServletRequest request, HttpServletResponse response,  
            Object handler, ModelAndView modelAndView) throws Exception {  
    }  
  
    /** 
     * Handler执行之前调用这个方法 
     */  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,  
            Object handler) throws Exception {  
        //获取请求的URL  
        String url = request.getRequestURI();  
        //URL:login.jsp是公开的;这个demo是除了login.jsp是可以公开访问的，其它的URL都进行拦截控制  
        if(url.indexOf("login.action")>=0){  
            return true;  
        }  
        //获取Session  
        HttpSession session = request.getSession();  
        String username = (String)session.getAttribute("username");  
          
        if(username != null){  
            return true;  
        }  
        //不符合条件的，跳转到登录界面  
        request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);  
          
        return false;  
    }    
}```
在spring的配置文件中配置这个拦截器
```xml
<!-- 拦截器 -->  
        <mvc:interceptors>  
            <!-- 多个拦截器，顺序执行 -->  
            <mvc:interceptor>  
                <mvc:mapping path="/**"/>  
                <bean class="com.mvc.interceptor.LoginInterceptor"></bean>  
            </mvc:interceptor>  
        </mvc:interceptors>
```  
此外，也可在postHandle和afterCompletion中定义拦截逻辑，其中postHandle处理尚未渲染的ModelAndView数据，afterCompletion中处理后置的一些验证等操作.
## 实例2 性能监控
如记录一下请求的处理时间，得到一些慢请求（如处理时间超过500毫秒），从而进行性能改进，一般的反向代理服务器如apache都具有这个功能，但此处我们演示一下使用拦截器怎么实现。
实现分析：
1、在进入处理器之前记录开始时间，即在拦截器的preHandle记录开始时间；
2、在结束请求处理之后记录结束时间，即在拦截器的afterCompletion记录结束实现，并用结束时间-开始时间得到这次请求的处理时间。
问题：
我们的拦截器是单例，因此不管用户请求多少次都只有一个拦截器实现，即线程不安全，那我们应该怎么记录时间呢？
解决方案是使用ThreadLocal，它是线程绑定的变量，提供线程局部变量（一个线程一个ThreadLocal，A线程的ThreadLocal只能看到A线程的ThreadLocal，不能看到B线程的ThreadLocal）。
```java
public class StopWatchHandlerInterceptor extends HandlerInterceptorAdapter {  
    private NamedThreadLocal<Long>  startTimeThreadLocal =   
new NamedThreadLocal<Long>("StopWatch-StartTime");  
    @Override  
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,   
Object handler) throws Exception {  
        long beginTime = System.currentTimeMillis();//1、开始时间  
        startTimeThreadLocal.set(beginTime);//线程绑定变量（该数据只有当前请求的线程可见）  
        return true;//继续流程  
    }  
      
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,   
Object handler, Exception ex) throws Exception {  
        long endTime = System.currentTimeMillis();//2、结束时间  
        long beginTime = startTimeThreadLocal.get();//得到线程绑定的局部变量（开始时间）  
        long consumeTime = endTime - beginTime;//3、消耗的时间  
        if(consumeTime > 500) {//此处认为处理时间超过500毫秒的请求为慢请求  
            //TODO 记录到日志文件  
            System.out.println(  
String.format("%s consume %d millis", request.getRequestURI(), consumeTime));  
        }          
    }  
}
```  
NamedThreadLocal：Spring提供的一个命名的ThreadLocal实现。
在测试时需要把stopWatchHandlerInterceptor放在拦截器链的第一个，这样得到的时间才是比较准确的。

