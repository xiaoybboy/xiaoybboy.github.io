---
title: SpringMVC注解@ControllerAdvice
date: 2020-03-09 23:34:08
tags:[SpringMVC]
---

# 介绍

@ControllerAdvice顾名思义，用于Controller增强，是一个在类上声明的注解，可以用于定义@ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping、@PostMapping， @GetMapping注解中。

使用方法常见有下面3种：

1. 结合方法型注解@ExceptionHandler，用于捕获Controller中抛出的指定类型的异常，从而达到不同类型的异常区别处理的目的；

2. 结合方法型注解@InitBinder，用于request中自定义参数解析方式进行注册，从而达到自定义指定格式参数的目的。

3. 结合方法型注解@ModelAttribute，表示其标注的方法将会在目标Controller方法执行之前执行。

   <!--more-->

总体使用：

```java
package com.sam.demo.controller;
 
import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Map;
 
/**
 * controller 增强器
 * @author sam
 * @since 2017/7/17
 */
@ControllerAdvice
public class MyControllerAdvice {
 
    /**
     * 应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器
     * @param binder
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {}
 
    /**
     * 把值绑定到Model中，使全局@RequestMapping可以获取到该值
     * @param model
     */
    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("author", "Magical Sam");
    }
 
    /**
     * 全局异常捕捉处理
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public Map errorHandler(Exception ex) {
        Map map = new HashMap();
        map.put("code", 100);
        map.put("msg", ex.getMessage());
        return map;
    }
}
```

# 1.@ExceptionHandler

  @ExceptionHandler的作用主要在于声明一个或多个类型的异常，当符合条件的Controller抛出这些异常之后将会对这些异常进行捕获，然后按照其标注的方法的逻辑进行处理，从而改变返回的视图信息。

```java
@ControllerAdvice
@ResponseBody
public class WebExceptionHandle {
    private static Logger logger = LoggerFactory.getLogger(WebExceptionHandle.class);
    /**
     * 400 - Bad Request
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ServiceResponse handleHttpMessageNotReadableException(HttpMessageNotReadableException e) {
        logger.error("参数解析失败", e);
        return ServiceResponseHandle.failed("could_not_read_json");
    }
    
    /**
     * 405 - Method Not Allowed
     */
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ServiceResponse handleHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {
        logger.error("不支持当前请求方法", e);
        return ServiceResponseHandle.failed("request_method_not_supported");
    }

    /**
     * 415 - Unsupported Media Type
     */
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)
    public ServiceResponse handleHttpMediaTypeNotSupportedException(Exception e) {
        logger.error("不支持当前媒体类型", e);
        return ServiceResponseHandle.failed("content_type_not_supported");
    }

    /**
     * 500 - Internal Server Error
     */
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(Exception.class)
    public ServiceResponse handleException(Exception e) {
        if (e instanceof BusinessException){
            return ServiceResponseHandle.failed("BUSINESS_ERROR", e.getMessage());
        }
        
        logger.error("服务运行异常", e);
        e.printStackTrace();
        return ServiceResponseHandle.failed("server_error");
    }
}
```

# 2.@InitBinder

@InitBinder的主要作用是绑定一些自定义的参数。对于一些特殊类型参数，比如Date，它们的绑定Spring是没有提供直接的支持的，我们只能为其声明一个转换器，将request中字符串类型的参数通过转换器转换为Date类型的参数，从而供给@RequestMapping标注的方法使用。

 如下是使用@InitBinder注册Date类型参数转换器的实现：

```java
@ControllerAdvice(basePackages = "mvc")
public class SpringControllerAdvice {
  @InitBinder
  public void globalInitBinder(WebDataBinder binder) {
    binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
  }
}
```

这里@InitBinder标注的方法注册的转换器在每次request请求进行参数转换时都会调用，用于判断指定的参数是否为其可以转换的参数。

```java
@Controller
@RequestMapping("/user")
public class UserController {

  @Autowired
  private UserService userService;

  @RequestMapping(value = "/detail", method = RequestMethod.GET)
  public ModelAndView detail(@RequestParam("id") long id, Date date) {
    System.out.println(date);
    ModelAndView view = new ModelAndView("user");
    User user = userService.detail(id);
    view.addObject("user", user);
    return view;
  }
}
```

  在浏览器输入http://localhost:8080/user/detail?id=1&date=2018-10-2，可以看到控制台进行了如下打印：

```
Tue Oct 02 00:00:00 CST 2018
```

​    可以看到，这里我们对request参数进行了转换，并且在接口中成功接收了该参数。

# 3. @ModelAttribute

关于@ModelAttribute的用法，处理用于接口参数可以用于转换对象类型的属性之外，其还可以用来进行方法的声明。如果声明在方法上，并且结合@ControllerAdvice，该方法将会在@ControllerAdvice所指定的范围内的所有接口方法执行之前执行，并且@ModelAttribute标注的方法的返回值还可以供给后续会调用的接口方法使用。

  这里@ModelAttribute的各个属性值主要是用于其在接口参数上进行标注时使用的，如果是作为方法注解，其name或value属性则指定的是返回值的名称。如下是使用@ModelAttribute进行方法标注的一个例子：

```java
@ControllerAdvice(basePackages = "mvc")
public class SpringControllerAdvice {
  @ModelAttribute(value = "message")
  public String globalModelAttribute() {
    System.out.println("global model attribute.");
    return "this is from model attribute";
  }
}
```

​    这里需要注意的是，该方法提供了一个String类型的返回值，而@ModelAttribute中指定了该属性名称为message，这样在Controller层就可以接收该参数了，如下是Controller层的代码：

```java
@Controller
@RequestMapping("/user")
public class UserController {

  @Autowired
  private UserService userService;

  @RequestMapping(value = "/detail", method = RequestMethod.GET)
  public ModelAndView detail(@RequestParam("id") long id, 
       @ModelAttribute("message") String message) {
    System.out.println(message);
    ModelAndView view = new ModelAndView("user");
    User user = userService.detail(id);
    view.addObject("user", user);
    return view;
  }
}
```

​    可以看到，这里使用@ModelAttribute注解接收名称为message的参数，从而获取了前面绑定的参数。运行上述代码并且访问http://localhost:8080/user/detail?id=1，可以看到页面进行了正常的展示，控制台也进行了如下打印：

```
global model attribute.
this is from model attribute
```

​    可以看到，这里使用@ModelAttribute注解标注的方法确实在目标接口执行之前执行了。需要说明的是，@ModelAttribute标注的方法的执行是在所有拦截器的preHandle()方法执行之后才会执行。

