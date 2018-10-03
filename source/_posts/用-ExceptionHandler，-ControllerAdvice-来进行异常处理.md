---
title: 用@ExceptionHandler，@ControllerAdvice 来进行异常处理
date: 2017-08-05 19:03:25
tags: [springmvc,异常处理]
categories: spring
---
有时候我们想统一处理一个Controller中抛出的异常怎么搞呢？
直接在Controller里面加上用@ExceptionHandler标注一个处理异常的方法像下面这样子，@ExceptionHandler只在当前controller里面有效。当前controller中所有抛出指定异常的方法。
<!--more-->
```java
1.	@ExceptionHandler(MissingServletRequestParameterException.class)  
2.	@ResponseStatus(HttpStatus.BAD_REQUEST)  
3.	public void processMethod(MissingServletRequestParameterException ex,HttpServletRequest request ,HttpServletResponse response) throws IOException {  
4.	    System.out.println("抛异常了！"+ex.getLocalizedMessage());  
5.	    logger.error("抛异常了！"+ex.getLocalizedMessage());  
6.	    response.getWriter().printf(ex.getMessage());  
7.	    response.flushBuffer();  
```  
这样，Controller里面的方法抛出了MissingServletRequestParameterException异常就会执行上面的这个方法来进行异常处理。
像我下面的代码`
```java
1.	@RequestMapping("/index")  
2.	public String index(@RequestParam String id,ModelMap modelMap){  
3.	    if (id==null){
4.		   throw new MissingServletRequestParameterException();
5.	}
6.	    return "login";  
7.	} 
``` 
如果我没有传入id值，那么就会抛出MissingServletRequestParameterException的异常，就会被上面的异常处理方法处理。
上面的@ExceptionHandler(MissingServletRequestParameterException.class)这个注解的value的值是一个Class[]类型的，这里的ExceptionClass是你自己指定的，你也可以指定多个需要处理的异常类型，比如这样
@ExceptionHandler(value = {MissingServletRequestParameterException.class,BindException.class})，这样就会处理多个异常了。
另一种更加通用的方法时：
首先定义这样一个统一异常处理类
```java
ExceptionController.Java
1.	import net.sf.json.JSONObject;  
2.	public abstract class ExceptionController {  
3.	    @ExceptionHandler  
4.	    public @ResponseBody String exceptionProcess(HttpServletRequest request, HttpServletResponse  
5.	            response, RuntimeException ex) {  
6.	        JSONObject json = new JSONObject();  
7.	        json.put("isError", true);  
8.	        json.put("msg", ex.getMessage());  
9.	        return json.toString();  
10.	    }  
11.	}  
```
在需要进行统一处理的类中继承这个class
```java
1.	import net.sf.json.JSONObject;  
2.	@RequestMapping("/auth/customer")  
3.	@Controller  
4.	@LoginAuth  
5.	public class CustomerController extends ExceptionController {  
6.	    @Resource  
7.	    private CustomerService customerService;  
8.	      
9.	    /** 
10.	     *  
11.	     * @param page 第几页，从1开始 
12.	     * @param rows 每页多少记录的行数 
13.	     * @return JSON String 
14.	     */  
15.	    @RequestMapping("/querybypage")  
16.	    @ResponseBody  
17.	    public String QueryByPage(int page, int rows) {  
18.	        //当为缺省值的时候进行赋值  
19.	        int currentpage = ( page == 0) ? 1: page;   //第几页  
20.	        int pageSize = (rows == 0) ? 5: rows;   //每页显示数量  
21.	        int index = (currentpage - 1) * pageSize;   //从第几条开始显示  
22.	        JSONObject map = customerService.queryByPage(index, pageSize);  
23.	        return map.toString();  
24.	    }  
25.	      
26.	    @RequestMapping("/test")  
27.	    public @ResponseBody Object test() {  
28.	        JSONObject json = new JSONObject();  
29.	        json.put("status", "success");  
30.	        String str = null;  
31.	        str.toString();  
32.	        return json.toString();  
33.	    }  
34.	} 
``` 
其中重要的一行是
1.	public class CustomerController extends ExceptionController  
这样CustomController里的方法被访问的时候， 如果有异常，就会被exceptionProces()处理.因为继承之后，子类中都有这个标注的方法了。
但这个注解只会是在当前的Controller里面起作用，如果想在所有的Controller里面统一处理异常的话，可以用@ControllerAdvice来创建一个专门处理全局的的类。
```java
1.	@ControllerAdvice    
2.	    public class SpringExceptionHandler{    
3.	      /**  
4.	         * 全局处理Exception  
5.	         * 错误的情况下返回500  
6.	         * @param ex  
7.	         * @param req  
8.	         * @return  
9.	         */    
10.	        @ExceptionHandler(value = {Exception.class})    
11.	        public ResponseEntity<Object> handleOtherExceptions(final Exception ex, final WebRequest req) {    
12.	            TResult tResult = new TResult();  //处理结果  
13.	            tResult.setStatus(CodeType.V_500);    
14.	            tResult.setErrorMessage(ex.getMessage());    
15.	            return new ResponseEntity<Object>(tResult,HttpStatus.OK);    
16.	        }    
17.	    }
```    
注意，这种注解在springmvc的配置文件中扫描是，要把@ControllerAdvice也包含进来，否则不起作用。
此外，@ControllerAdvice还可以用于全局的requestMapping，较少用。
参考：http://jinnianshilongnian.iteye.com/blog/1866350
     http://blog.csdn.net/u013632755/article/details/49908621
