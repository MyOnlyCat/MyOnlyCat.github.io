
---
title: SpringBoot异常统一处理
date: 2017-12-18 11:40
categories: SpringBoot
tags: 
		- SpringBoot, 
		- 异常处理
---

**SpringBoot异常统一处理(初学记录下)**
<!--more-->

### 1.建立测试异常

```java
@RestController
@RequestMapping(value = "/ecpc")
public class ExceptionController {

    @RequestMapping(value = "/ecpc1")
    public String ecpcTest() throws Exception {
        throw new Exception("发生错误");
    }
}
```
不进行处理的话访问 `http://localhost:8080/ecpc/ecpc1`,会出下面的界面

![1.png-14.7kB][1]
### 2.统一异常处理类
```java
@RestControllerAdvice //定义统一的异常处理类
public class GlobalExceptionHandler {

    private Logger log = LoggerFactory.getLogger("GlobalExceptionHandler");

    @ResponseBody
    @ExceptionHandler(value = Exception.class ) //定义针对的异常类型,这里捕获Exception类型和其所用子异常
    public Object errorHandler(HttpServletRequest req, Exception e) throws Exception {
        log.error("---DefaultException Handler---Host {} invokes url {} ERROR: {}",req.getRemoteHost(),req.getRequestURL(),e.getMessage());
        return e.getMessage();
    }
}
```

- `@RestControllerAdvice`  表明GlobalExceptionHandler 是一个全局的异常处理器,也是是一个 RESTful Controller, 即它会以 RESTful 的形式返回回复.(类注解, 作用于 整个 Spring 工程. ControllerAdvice 注解定义了一个全局的异常处理器,如果你们的异常需要返回页面啊之类的，你可以使用@ControllerAdvice分别定制。)
- `ExceptionHandler(value = Exception.class )` 表示 defaultErrorHandler 会处理 Exception 异常和其所用子异常(作用于 Controller 级别. ExceptionHandler 注解为一个 Controler 定义一个异常处理器)
- 捕获效果如图:
![2.png-11.2kB][2]
- 控制台打印效果如图:
![3.png-6.7kB][3]


 


  [1]: http://static.zybuluo.com/pockadmin/3a1ofdtjiqrbfyhtz9t30vp6/1.png
  [2]: http://static.zybuluo.com/pockadmin/foyhnhfbsuri2fcfgbpxzakk/2.png
  [3]: http://static.zybuluo.com/pockadmin/dr3lpm17ew4ijfz3hmldblte/3.png