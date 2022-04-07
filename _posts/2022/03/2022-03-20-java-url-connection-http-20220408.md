---
layout: post
title:  使用 HttpUrlConnection 在Java中做一个简单的HTTP请求 20220408
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

今天将介绍一种在 Java 中执行 HTTP 请求的方法 -- 通过使用 Java 内置的 HttpUrlConnection 类实现。

从 JDK 11 开始，Java 为执行 HTTP 请求提供了一个新的 API，它是用来替代 HttpUrlConnection 的，即HttpClient API。

<!--more-->

### HttpUrlConnection

HttpUrlConnection 类允许我们执行基本的 HTTP 请求，而无需使用任何额外的库。我们需要的所有类都是 java.net 包的一部分。

使用这种方法的缺点是，代码可能比其他的HTTP库更繁琐，而且它不提供更高级的功能，如添加头文件或认证的专用方法。

### 创建一个请求

我们可以使用 URL 类的 openConnection() 方法创建一个 HttpUrlConnection 实例。注意，这个方法只是创建一个连接对象，但还没有建立连接。

HttpUrlConnection 类通过将 requestMethod 属性设置为get, post, head, options, put, delete, trace其中一个值。

让我们使用GET方法创建一个与给定URL的连接:

```java
URL url = new URL("https://www.javanorth.cn");
HttpURLConnection con = (HttpURLConnection) url.openConnection();
con.setRequestMethod("GET");
```

### 添加请求参数

如果我们想向一个请求添加参数，我们必须将 doOutput 属性设置为 true，然后向 HttpUrlConnection 实例的OutputStream 写一个类似 `param1=value&m2=value` 的字符串。

```java
Map<String, String> parameters = new HashMap<>();
parameters.put("param1", "val");

con.setDoOutput(true);
DataOutputStream out = new DataOutputStream(con.getOutputStream());
out.writeBytes(ParameterStringBuilder.getParamsString(parameters));
out.flush();
out.close();
```

为了方便参数Map的转换，我们编写了一个名为ParameterStringBuilder的实用类，其中包含一个静态方法getParamsString()，可以将Map转换为所需格式的字符串。

```java
public class ParameterStringBuilder {
    public static String getParamsString(Map<String, String> params) 
      throws UnsupportedEncodingException{
        StringBuilder result = new StringBuilder();

        for (Map.Entry<String, String> entry : params.entrySet()) {
          result.append(URLEncoder.encode(entry.getKey(), "UTF-8"));
          result.append("=");
          result.append(URLEncoder.encode(entry.getValue(), "UTF-8"));
          result.append("&");
        }

        String resultString = result.toString();
        return resultString.length() > 0 ? resultString.substring(0, resultString.length() - 1) : resultString;
    }
}
```

### 设置请求头信息

通过使用 setRequestProperty() 方法可以实现在请求中添加头信息。

```java
con.setRequestProperty("Content-Type", "application/json");
```

要从一个连接中读取一个头的值，我们可以使用 getHeaderField() 方法。

```java
String contentType = con.getHeaderField("Content-Type");
```

### 配置超时

HttpUrlConnection 类允许设置连接和读取超时。这些值定义了等待与服务器的连接建立或数据可被读取的时间间隔。

为了设置超时值，我们可以使用 setConnectTimeout（）和 setReadTimeout（）方法。

```java
con.setConnectTimeout(5000);
con.setReadTimeout(5000);
```

在这个例子中，我们把两个超时值都设为5秒。

### 处理Cookie

java.net 包包含了便于处理 cookie 的类，如 CookieManager 和 HttpCookie。

首先，为了从响应中读取 cookie，我们可以检索 Set-Cookie 头的值，并将其解析为一个 HttpCookie 对象的列表。

```java
String cookiesHeader = con.getHeaderField("Set-Cookie");
List<HttpCookie> cookies = HttpCookie.parse(cookiesHeader);
```

接下来，我们将把cookie添加到cookieStore。

```java
cookies.forEach(cookie -> cookieManager.getCookieStore().add(null, cookie));
```

让我们检查一下是否有一个叫做 username 的 cookie，如果没有，我们将把它添加到cookieStore，其值为 "javanorth"。

```java
Optional<HttpCookie> usernameCookie = cookies.stream().findAny().filter(cookie -> cookie.getName().equals("username"));
if (usernameCookie == null) {
    cookieManager.getCookieStore().add(null, new HttpCookie("username", "javanorth"));
}
```

最后，为了在请求中加入 cookie，我们需要在关闭和重新打开连接后设置 Cookie 头。

```java
con.disconnect();
con = (HttpURLConnection) url.openConnection();

con.setRequestProperty("Cookie", StringUtils.join(cookieManager.getCookieStore().getCookies(), ";"));
```

### 处理重定向

我们可以通过使用参数为 true 或 false 的 setInstanceFollowRedirects() 方法，为一个特定的连接启用或禁用自动跟踪重定向。

```java
con.setInstanceFollowRedirects(false);
```

也可以启用或禁用所有连接的自动重定向。

```java
HttpUrlConnection.setFollowRedirects(false);
```

默认情况下，该行为是启用的。

当一个请求返回状态代码 301 或 302，表示重定向时，我们可以检索位置头并创建一个新的请求到新的URL。

```java
if (status == HttpURLConnection.HTTP_MOVED_TEMP || status == HttpURLConnection.HTTP_MOVED_PERM) {
    String location = con.getHeaderField("Location");
    URL newUrl = new URL(location);
    con = (HttpURLConnection) newUrl.openConnection();
}
```

### 读取响应

读取请求的响应可以通过解析 HttpUrlConnection 实例的 InputStream 来完成。

为了执行请求，我们可以使用 getResponseCode()、connect()、getInputStream() 或 getOutputStream() 方法。

```java
int status = con.getResponseCode();
```

最后，让我们读一下请求的响应，并把它放在一个内容字符串中。

```java
BufferedReader in = new BufferedReader(new InputStreamReader(con.getInputStream()));
String inputLine;
StringBuffer content = new StringBuffer();
while ((inputLine = in.readLine()) != null) {
    content.append(inputLine);
}
in.close();
```

要关闭连接，我们可以使用 disconnect() 方法。

```java
con.disconnect();
```

### 在失败的请求中读取响应

如果请求失败了，我们从 HttpUrlConnection 实例的 InputStream 读取是读取不到数据的。我们可以从 HttpUrlConnection.getErrorStream() 提供的流读取。

我们可以通过比较 HTTP 状态码来决定使用哪个 InputStream。

```java
int status = con.getResponseCode();

Reader streamReader = null;

if (status > 299) {
    streamReader = new InputStreamReader(con.getErrorStream());
} else {
    streamReader = new InputStreamReader(con.getInputStream());
}
```

最后，我们可以用与上一节相同的方式读取 streamReader。

### 总结

在这篇文章中，我们展示了如何使用 HttpUrlConnection 类来执行HTTP请求。