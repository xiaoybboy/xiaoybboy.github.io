---
title: SpringMVC注解@ControllerAdvice
date: 2020-03-09 23:34:08
tags: [SpringMVC]
---

# 介绍

**@ControllerAdvic**e顾名思义，用于Controller增强，是一个在类上声明的注解。可以用于定义**@ExceptionHandler**、**@InitBinder**、**@ModelAttribute**，并应用到所有**@RequestMapping**注解中（用于增强上述三个注解的作用范围）。使用方法常见有下面3种：

1. 结合注解**@ExceptionHandler**，捕获制定类型的异常，从而对不同类型的异常区分处理。
2. 结合注解**@InitBinde**r，用于绑定全局的属性编辑器或用于全局的参数预处理。
3. 结合注解**@ModelAttribute**，设置全局数据，供所有的@**RequestMapping**使用。

<!--more-->

总体使用：

```java
import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Map;

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

  **@ExceptionHandler**的作用在于声明一个或多个类型的异常，当符合条件的Controller抛出这些异常之后将会对这些异常进行捕获，然后按照其标注的方法的逻辑进行处理，改变返回的视图信息。通过 **@ControllerAdvice** 结合 **@ExceptionHandler** 可以全局异常捕获机制，所有的Controller抛出的异常会被捕获，并对应到相应的处理方法中。

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
}
```

# 2.@InitBinder

## 2.1 绑定属性编辑器

**@InitBinder**可以绑定**请求参数到指定的属性编辑器**。

前端传递给后端的值是String类型的，当向Model中Set值时，如果Set的属性是一个对象，Spring会找对应的Editor进行转换之后再Set进去。Spring本身提供了很多Editor，如CustomDateEditor ，CustomBooleanEditor，CustomNumberEdito等。有时候对于一些特殊类型的参数，我们需要手动注册一个编辑器。例如：前端出入的日期为一个String，后端接收为Date类型，这时候通过**@InitBinder**可以注册一个Editor用于这个转换（当类上加上**@ControllerAdvice**时，表示注册全局的Editor）。

 如下是使用**@InitBinder**注册Date类型参数转换器的实现：

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
    return view;
  }
}
```

 在浏览器输入http://localhost:8080/user/detail?id=1&date=2018-10-2

可以看到控制台进行了如下打印：

```
Tue Oct 02 00:00:00 CST 2018
```

​    可以看到，这里我们对request参数进行了转换，并且在接口中成功接收了该参数。

## 2.2  请求参数预处理



# 3. @ModelAttribute

当**@ModelAttribute**在方法上使用时，该方法会在`Controller`中每个方法执行之前执行，可以用于添加一个或多个属性到model上，填充一些公共需要的属性或数据，其他**@RequestMapping**就可以从model中获取这些属性。

1）使用`@ModelAttribute`注解无返回值的方法

```java
@Controller
@RequestMapping(value = "/modelattribute")
public class ModelAttributeController {

    @ModelAttribute
    public void myModel(@RequestParam(required = false) String abc, Model model) {
        model.addAttribute("attributeName", abc);
    }

    @RequestMapping(value = "/method")
    public String method() {
        return "method";
    }
}
```

2）使用`@ModelAttribute`注解带有返回值的方法

```java
@ModelAttribute
public String myModel(@RequestParam(required = false) String abc) {
    return abc;
}

@ModelAttribute
public Student myModel(@RequestParam(required = false) String abc) {
    Student student = new Student(abc);
    return student;
}
```

当`@ModelAttribute`结合**@ControllerAdvice**时，**@ModelAttribute**注解的方法会在**@ControllerAdvice**所指定的范围内的所有**@RequestMapping**方法之前执行，并且**@RequestMapping**标注的方法添加的属性可以在所有**@RequestMapping**方法中获取。

  使用`@ModelAttribute`进行方法标注的一个例子：

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

​    该方法提供了一个String类型的返回值，而`@ModelAttribute`中指定了该属性名称为message，这样在Controller层就可以接收该参数了，如下是Controller层的代码：

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

​    可以看到，这里使用`@ModelAttribute`注解接收名称为message的参数，从而获取了前面绑定的参数。运行上述代码并且访问http://localhost:8080/user/detail?id=1

可以看到页面进行了正常的展示，控制台也进行了如下打印：

```
global model attribute.
this is from model attribute
```

​    可以看到，这里使用@ModelAttribute注解标注的方法确实在目标接口执行之前执行了。

# @ControllerAdvice指定作用范围

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {

	@AliasFor("basePackages")
	String[] value() default {};

	@AliasFor("value")
	String[] basePackages() default {};

	Class<?>[] basePackageClasses() default {};

	Class<?>[] assignableTypes() default {};

	Class<? extends Annotation>[] annotations() default {};
}
```

**@ControllerAdvice**默认是全局作用范围，即对所有的**@RequestMapping**有效。可以使用属性值来指定**@ControllerAdvice**注解的作用范围。

```java
//对com.xiao.app.controller包下RequestMapping起作用
@ControllerAdvice(value="com.xiao.app.mvc.controller")
```

```java
//对TestController及其子类型的Controller起作用
@ControllerAdvice(assignableTypes=TestController.class)
```

```java
//对标注了RestController注解的RequestMapping起作用
@ControllerAdvice(annotations=RestController.class)
```

上述属性也可以联合使用。

本文参考：https://my.oschina.net/zhangxufeng/blog/2222434

​			 	https://blog.csdn.net/junlon2006/article/details/91997607

