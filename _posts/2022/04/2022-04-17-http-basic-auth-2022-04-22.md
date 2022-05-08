---
layout: post
title:  Java HTTP 基本认证 -- 2022-04-22
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

今天我们将看一下 HTTP 基本认证。指北君将会讲讲它是如何工作的，并且一步步的教大家如何使用。

### 什么是 HTTP 基本认证

HTTP 基本认证是一种简单的认证方法。客户端可以通过用户名和密码进行认证。这些凭证以特定的格式在 Authorization HTTP Header 中发送。一般它以 Basic 关键字开始，后面是一个 base64 编码的用户名:密码值。冒号字符在这里很重要。头部应该严格遵循这个格式。

<!--more-->

例如，要用 javanorth 用户名和 http 密码进行认证，我们必须发送这个头。

```java
Basic amF2YW5vcnRoOmh0dHA=
```

我们可以通过使用 base64 解码器和检查解码的结果来验证。

### Java HttpClient

在 Java 9 中引入了一个新的 HttpClient 模块，它在 Java 11 中得到了标准化。下面代码部分指北君我将使用 Java 17，我们可以简单地从 `java.net.http` 包中导入它，无需任何额外的配置或依赖。

让我们从执行一个简单的GET请求开始，现在不需要任何认证。

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder().GET()
  .uri(new URI("https://www.javanorth.cn/get")).build();

HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

logger.info("Status {}", response.statusCode());
```

- 首先，我们创建一个 HttpClient 对象，它可以用来执行 HTTP 请求。
- 其次，我们使用构建器设计模式创建一个 HttpRequest 对象。GET 方法设置请求的 HTTP 方法。uri 方法设置了我们想发送请求的 URL。
- 之后，我们使用我们的客户端发送该请求。发送方法的第二个参数是一个响应体处理程序。这告诉客户端，我们想把响应体当作一个字符串。

让我们运行我们的应用程序并检查日志。输出应该是这样的。

```java
INFO cn.javanorth.httpclient.basicauthentication.HttpClientBasicAuthentication - Status 200
```

我们看到 HTTP 状态是 200，意味着我们的请求是成功的。在这之后，让我们看看我们如何处理认证。

### 使用 HttpClient 认证器

在我们配置认证之前，我们需要一个URL来测试它。让我们使用一个需要认证的Postman Echo端点。首先，将之前的URL改为这个，然后再次运行应用程序。

```java
HttpRequest request = HttpRequest.newBuilder().GET()
  .uri(new URI("https://www.javanorth.cn/basic-auth")).build();
```

我们检查一下日志，找找状态代码。这次我们收到 HTTP 状态 401 "未授权"。这个响应代码意味着端点需要认证，但客户端没有发送任何凭证。

我们修改一下代码，使其发送所需的认证数据。我们可以通过配置 HttpClient Builder 来做到这一点，我们的客户端将使用我们设置的凭证。这个端点接受用户名 "javanorth"和密码 "password"。让我们为我们的客户端添加一个认证器。

```java
HttpClient client = HttpClient.newBuilder()
  .authenticator(new Authenticator() {
      @Override
      protected PasswordAuthentication getPasswordAuthentication() {
          return new PasswordAuthentication("javanorth", "password".toCharArray());
      }
  })
  .build();
```

让我们再次运行该应用程序。现在请求成功了，我们收到HTTP状态200。

### 使用 HTTP 头文件进行认证

我们可以使用另一种方法来访问需要认证的端点。我们从前面的章节中了解到授权头是如何构建的，所以我们可以手动设置其值。尽管这必须在每个请求中进行，而不是通过认证器设置一次。

我们删除认证器，看看如何设置请求头。我们需要使用base64编码来构建头文件的值。

```java
private static final String getBasicAuthenticationHeader(String username, String password) {
    String valueToEncode = username + ":" + password;
    return "Basic " + Base64.getEncoder().encodeToString(valueToEncode.getBytes());
}
```

让我们为授权头设置这个值，并运行该应用程序。

```java
HttpRequest request = HttpRequest.newBuilder()
  .GET()
  .uri(new URI("https://www.javanorth.com/basic-auth"))
  .header("Authorization", getBasicAuthenticationHeader("postman", "password"))
  .build();
```

我们的请求是成功的，这意味着我们正确地构建和设置了头信息值。

### 总结

在这个简短的文章中，我们看到了什么是 HTTP 基本认证以及它如何工作。我们通过为Java HttpClient 设置一个认证器，使用了基本认证。另外我们使用了不同的方法，通过手动设置HTTP头来进行认证。
