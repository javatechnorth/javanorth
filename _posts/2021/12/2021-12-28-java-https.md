---
layout: post
title: Java 是如何做 HTTPS 双向认证的？ -- 2022-01-06
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

想必大家对 HTTPS 都有一定的了解吧。今天指北君将给大家聊聊 HTTPS 是如何做安全认证的。 HTTPS 是 HTTP 的一个扩展，允许计算机网络中的两个实体之间进行安全通信。HTTPS 使用TLS（传输层安全）协议来实现安全连接。

TLS 可以通过单向或双向的证书验证来实现。在单向中，服务器分享其公共证书，以便客户可以验证它是一个受信任的服务器。另一个选择是双向验证。客户端和服务器都分享他们的公共证书以验证对方的身份。

<!--more-->

指北君将重点介绍双向证书验证，服务器也将检查客户的证书。

### Java 和 TLS 版本

TLS 1.3是该协议的最新版本。这个版本的性能和安全性更高。它有一个更有效的握手协议，并使用现代加密算法。

Java 在Java 11中开始支持这个版本的协议。我们将使用这个版本来生成证书，并实现一个简单的客户端-服务器，使用TLS来验证对方。

### 在 Java 中生成证书

由于我们正在进行双向TLS认证，我们需要为客户端和服务器生成证书。

在生产环境中，我们建议从证书颁发机构购买这些证书。然而，对于测试或演示的目的，使用自签名的证书就足够了。在这篇文章中，我们将使用Java的keytool来生成自签名证书。

### 服务器证书

首先，我们生成服务器的密钥存储。

```java
keytool -genkey -alias serverkey -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -keystore serverkeystore.p12 -storepass password -ext san=ip:127.0.0.1,dns:localhost
```

我们使用keytool -ext选项来设置 Subject Alternative Names （SAN），以定义识别服务器的本地主机名/IP地址。一般来说，我们可以用这个选项指定多个地址。然而，客户将被限制在使用这些地址中的一个来连接到服务器。

接下来，我们把证书导出到文件server-certificate.pem中。

```java
keytool -exportcert -keystore serverkeystore.p12 -alias serverkey -storepass password -rfc -file server-certificate.pem
```

最后，我们将服务器证书添加到客户端的信任存储中。

```java
keytool -import -trustcacerts -file server-certificate.pem -keypass password -storepass password -keystore clienttruststore.jks
```

### 客户端证书

同样地，我们生成客户端的密钥存储并导出其证书。

```java
keytool -genkey -alias clientkey -keyalg RSA -keysize 2048 -sigalg SHA256withRSA -keystore clientkeystore.p12 -storepass password -ext san=ip:127.0.0.1,dns:localhost

keytool -exportcert -keystore clientkeystore.p12 -alias clientkey -storepass password -rfc -file client-certificate.pem

keytool -import -trustcacerts -file client-certificate.pem -keypass password -storepass password -keystore servertruststore.jks
```

在最后一条命令中，我们把客户的证书添加到服务器的信任存储中。

### Java 服务端实现

指北君在这里使用Java socket 来实现。SSLSocketEchoServer类得到了一个SSLServerSocket，以轻松支持TLS认证。我们只需要指定密码和协议，剩下的只是一个标准的应答服务器，回复与客户端发送的相同的信息。

```java
public class SSLSocketEchoServer {

    static void startServer(int port) throws IOException {

        ServerSocketFactory factory = SSLServerSocketFactory.getDefault();
        try (SSLServerSocket listener = (SSLServerSocket) factory.createServerSocket(port)) {
            listener.setNeedClientAuth(true);
            listener.setEnabledCipherSuites(new String[] { "TLS_AES_128_GCM_SHA256" });
            listener.setEnabledProtocols(new String[] { "TLSv1.3" });
            System.out.println("listening for messages...");
            try (Socket socket = listener.accept()) {
                
                InputStream is = new BufferedInputStream(socket.getInputStream());
                byte[] data = new byte[2048];
                int len = is.read(data);
                
                String message = new String(data, 0, len);
                OutputStream os = new BufferedOutputStream(socket.getOutputStream());
                System.out.printf("server received %d bytes: %s%n", len, message);
                String response = message + " processed by server";
                os.write(response.getBytes(), 0, response.getBytes().length);
                os.flush();
            }
        }
    }
}
```

服务器监听客户端的连接。listener.setNeedClientAuth(true) 的调用要求客户端与服务器共享其证书。在后台 SSLServerSocket 实现使用TLS协议对客户端进行认证。

在我们的例子中，自签的客户证书在服务器的信任存储中，因此 socket 将接受连接。服务器继续使用InputStream 来读取消息。然后，它使用OuputStream来回传传入的消息，并附加一个确认信息。

### Java客户端实现

与我们处理服务器的方式相同，客户端的实现是一个简单的SSLScocketClient类。

```java
public class SSLScocketClient {

    static void startClient(String host, int port) throws IOException {

        SocketFactory factory = SSLSocketFactory.getDefault();
        try (SSLSocket socket = (SSLSocket) factory.createSocket(host, port)) {
            
            socket.setEnabledCipherSuites(new String[] { "TLS_AES_128_GCM_SHA256" });
            socket.setEnabledProtocols(new String[] { "TLSv1.3" });
            
            String message = "Hello World";
            System.out.println("sending message: " + message);
            OutputStream os = new BufferedOutputStream(socket.getOutputStream());
            os.write(message.getBytes());
            os.flush();
            
            InputStream is = new BufferedInputStream(socket.getInputStream());
            byte[] data = new byte[2048];
            int len = is.read(data);
            System.out.printf("client received %d bytes: %s%n", len, new String(data, 0, len));
        }
    }
}
```

首先，我们创建一个SSLSocket，与服务器建立一个连接。在后台，该socket将设置TLS连接建立握手。作为握手的一部分，客户端将验证服务器的证书并检查它是否在客户端的信任库中。

一旦连接成功建立，客户端将使用输出流向服务器发送一个消息。然后，它用输入流读取服务器的响应。

### 运行应用程序

要运行服务器，请打开一个命令窗口并运行。
```java
java -Djavax.net.ssl.keyStore=/path/to/serverkeystore.p12 \ 
  -Djavax.net.ssl.keyStorePassword=password \
  -Djavax.net.ssl.trustStore=/path/to/servertruststore.jks \ 
  -Djavax.net.ssl.trustStorePassword=password \
  cn.javanorth.httpsclientauthentication.SSLSocketEchoServer
```

我们指定javax.net.ssl.keystore和javax.net.ssl.trustStore的系统属性指向我们之前用keytool创建的serverkeystore.p12和servertruststore.jks文件。

为了运行客户端，我们打开另一个命令窗口并运行。
```java
java -Djavax.net.ssl.keyStore=/path/to/clientkeystore.p12 \ 
  -Djavax.net.ssl.keyStorePassword=password \
  -Djavax.net.ssl.trustStore=/path/to/clienttruststore.jks \ 
  -Djavax.net.ssl.trustStorePassword=password \
  cn.javanorth.httpsclientauthentication.SSLScocketClient	
```

同样，我们设置javax.net.ssl.keyStore和javax.net.ssl.trustStore系统属性，使其指向我们之前用keytool生成的clientkeystore.p12和clienttruststore.jks文件。

### 总结

今天指北君写了一个简单的客户端-服务器的Java实现，来演示使用服务器和客户端证书做双向的TLS认证。
