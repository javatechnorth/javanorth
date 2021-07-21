---
layout: post
title:  3个案例让你掌握 Spring Boot 全局异常处理
tagline: by feng
categories: "Spring-Boot"
tags: 
    - feng
---

大家好，我是指北君。

### 前言

在平时的 API 开发过程中，总会遇到一些错误异常没有捕捉到的情况。那有的小伙伴可能会想，这还不简单么，我在 API 最外层加一个 `try...catch` 不就完事了。

哈哈哈，没错。这种方法简单粗暴。指北君曾经也是这么干的，但是你转过来想一想，你会在每一个 API 入口，都去做 `try...catch` 吗？这样不是代码非常丑陋的。小伙伴开始思考，突然灵光一现，说我们实现一个 AOP 来做这事不久完了。没错，使用 AOP 来实现是最佳的选择。

现在指北君就给大家来介绍介绍 `Spring Boot` 怎么通过注解来实现全局异常处理的。

<!--more-->

### 主角 `@ControllerAdvice` 和 `@ExceptionHandler`

我们先来介绍一下今天的主角，分别是 `@ControllerAdvice` 和 `@ExceptionHandler` 。

+ `@ControllerAdvice` 相当于 `controller` 的切面，主要用于 `@ExceptionHandler`,  `@InitBinder` 和 `@ModelAttribute`，使注解标注的方法对每一个 `controller` 都起作用。默认对所有 `controller` 都起作用，当然也可以通过 `@ControllerAdvice` 注解中的一些属性选定符合条件的 `controller` 。
+ `@ExceptionHandler` 用于异常处理的注解，可以通过 `value` 指定处理哪种类型的异常还可以与 `@ResponseStatus` 搭配使用，处理特定的 `http` 错误。标记的方法入参与返回值都有很大的灵活性，具体可以看注释也可以后边的深度探究。

### 案例分析

今天我们就通过几种案例的方式，来给大家分析分析，怎么通过全局异常处理的方式玩转 Spring Boot 的全局异常处理。

#### 案例一

一般的异常处理，所有的API都需要有相同的异常结构。

![exception1](http://www.javanorth.cn/assets/images/2021/feng/exception1.png)

在这种情况下，实现是非常简单的，我们只需要创建 `GeneralExceptionHandler` 类，用 `@ControllerAdvice` 注解来注解它，并创建所需的 `@ExceptionHandler` ，它将处理所有由应用程序抛出的异常，如果它能找到匹配的 `@ExceptionHandler`，它将相应地进行转换。

```java
@ControllerAdvice
public class GeneralExceptionHandler {
    @ExceptionHandler(Exception.class)
    protected ResponseEntity<Error> handleException(Exception ex) {
       MyError myError = MyError.builder()
                         .text(ex.getMessage())
                         .code(ex.getErrorCode()).build();
       return new ResponseEntity(myError,
                               HttpStatus.valueOf(ex.getErrorCode()));
    }
}
```

#### 案例二

我们有一个API，它需要有一个或多个异常以其他格式处理，与其他应用程序的 API 不同。

![exception2](http://www.javanorth.cn/assets/images/2021/feng/exception2.png)

我们可以采取两种方式来实现这种情况。我们可以在 `OtherController` 内部添加 `@ExceptionHandler` 来处理 `OtherException` ，或者为 `OtherController` 创建新的`@ControllerAdvice`，以备我们也想在其他 API 中处理 `OtherException`。

在 `OtherController` 中添加 `@ExceptionHandler` 来处理 `OtherException` 的代码示例。

```java
@RestController
@RequestMapping("/other")
public class OtherController {
    @ExceptionHandler(OtherException.class)
    protected ResponseEntity<Error> handleException(OtherException ex) {
      MyOtherError myOtherError = MyOtherError.builder()
                         .message(ex.getMessage())
                         .origin("Other API")
                         .code(ex.getErrorCode()).build();
      return new ResponseEntity(myOtherError,
                               HttpStatus.valueOf(ex.getErrorCode()));
    }
}
```

只针对 `OtherController` 控制器的 `@ControllerAdvice` 的代码示例

```java
@ControllerAdvice(assignableTypes = OtherController.class)
public class OtherExceptionHandler {
    @ExceptionHandler(OtherException.class)
    protected ResponseEntity<Error> handleException(OtherException ex) {
      MyOtherError myOtherError = MyOtherError.builder()
                         .message(ex.getMessage())
                         .origin("Other API")
                         .code(ex.getErrorCode()).build();
      return new ResponseEntity(myOtherError,
                               HttpStatus.valueOf(ex.getErrorCode()));
    }
}
```

#### 案例三

与案例二类似，我们有一个 API 需要以不同于应用程序中其他 API 的方式对异常进行格式化，但这次所有的异常都需要进行不同的转换。

![exception3](http://www.javanorth.cn/assets/images/2021/feng/exception3.png)

为了实现这个案例，我们将不得不使用两个 `@ControllerAdvice`，并加上 `@Order` 注解的注意事项。因为现在我们需要告诉 `Spring`，在处理同一个异常时，哪个 `@ControllerAdvice` 的优先级更高。如果我们没有指定 `@Order`，在启动时，其中一个处理程序将自动注册为更高的顺序，我们的异常处理将变得不可预测。例如，我最近看到一个案例，如果你使用 `mvn springboot:run` 任务启动一个应用程序，`OtherExceptionHandler` 是主要的，但是当以jar形式启动时，`GeneralExceptionHandler` 是主要的。

```java
@ControllerAdvice
public class GeneralExceptionHandler {
    @ExceptionHandler(Exception.class)
    protected ResponseEntity<Error> handleException(Exception ex) {
       MyError myError = MyError.builder()
                         .text(ex.getMessage())
                         .code(ex.getErrorCode()).build();
       return new ResponseEntity(myError,
                               HttpStatus.valueOf(ex.getErrorCode()));
    }
}
@ControllerAdvice(assignableTypes = OtherController.class)
@Order(Ordered.HIGHEST_PRECEDENCE)
public class OtherExceptionHandler {
    @ExceptionHandler(Exception.class)
    protected ResponseEntity<Error> handleException(Exception ex) {
       MyError myError = MyError.builder()
                         .message(ex.getMessage())
                         .origin("Other API")
                         .code(ex.getErrorCode()).build();
       return new ResponseEntity(myError,
                               HttpStatus.valueOf(ex.getErrorCode()));
    }
}
```

### 总结

经过上述的几个案例，指北君觉得大家应该已经能够轻松应对 Spring Boot 中大部分的全局异常处理的情况。

细心的同学也许会觉得为什么不使用 `@RestControllerAdvice` 呢？ 如果是用的 `@RestControllerAdvice` 注解，它会将数据自动转换成JSON格式，不再需要 `ResponseEntity` 的处理来。这种于 `Controller` 和 `RestController` 类似，本质是一样的，所以我们在使用全局异常处理的之后可以进行灵活的选择处理。

有任何问题可以在公众号后台留言，指北君会第一时间回复大家。欢迎关注公众号【Java技术指北】，第一时间获取更多精彩内容。
