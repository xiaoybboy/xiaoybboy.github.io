---
title: SpringMVC注解@ControllerAdvice
date: 2020-03-09 23:34:08
tags:[SpringMVC]
---

# 介绍

@ControllerAdvice是一个在类上声明的注解，可以用于定义@ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping、@PostMapping， @GetMapping注解中。使用方法常见有下面3种：

1. 结合方法型注解@ExceptionHandler，用于捕获Controller中抛出的指定类型的异常，从而达到不同类型的异常区别处理的目的；
2. 结合方法型注解@InitBinder，用于request中自定义参数解析方式进行注册，从而达到自定义指定格式参数的目的；
3. 结合方法型注解@ModelAttribute，表示其标注的方法将会在目标Controller方法执行之前执行。

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

  @ExceptionHandler的作用主要在于声明一个或多个类型的异常，当符合条件的Controller抛出这些异常之后将会对这些异常进行捕获，然后按照其标注的方法的逻辑进行处理，从而改变返回的视图信息。如下是@ExceptionHandler的属性结构：