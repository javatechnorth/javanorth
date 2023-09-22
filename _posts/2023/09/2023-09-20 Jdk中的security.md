
---
layout: post
title:  2023-09-20 Jdk中的security
tagline: by 沉浮
categories: 
tags: 沉浮
---

<!--more-->

## JDK Security

前段时间再看关于JDK算法相关的知识时，看到很多与jdk中security包下的模块有一定关联，之前对这块一直没有关注，趁此机会先做个简单的了解。

本文旨在深入探讨Java的Security技术，包括其核心概念、关键模块以及具体应用。通过详细分析，希望帮助读者更好地理解如何在Java应用程序中实现安全防护，提高系统的可靠性和稳定性。

主要功能包括授权、访问控制、数据加密、身份验证等

### 核心模块

+ 授权与访问控制  
Java Security提供了一套完整的授权和访问控制机制，包括基于角色的访问控制、基于声明的访问控制、自定义访问控制等。这些机制可以有效地保护系统资源，防止未经授权的访问。

+ 数据加密与解密  
Java Security提供了丰富的加密算法，如AES、RSA等，用于对敏感数据进行加密和解密。通过使用这些算法，我们可以保护数据的安全性，防止数据泄露。

+ 身份验证与授权  
身份验证是确认用户身份的过程，授权则是确定用户是否有权执行某项任务。Java Security提供了各种身份验证和授权机制，如用户名密码验证、数字证书验证等，以满足不同场景的需求。

+ 安全套接字层（SSL）与传输层安全（TLS）  
SSL和TLS是用于在通信过程中保证数据传输安全的协议。Java Security提供了SSL和TLS的实现，可用于构建安全的网络通信。

### 授权与访问控制

Java提供了对敏感信息的访问控制功能，比如本地文件，类方法等的访问约束，以此来组建安全的代码框架

先看一个文件写入的示例：
1. 定义policy
```
grant {
    permission com.sucl.blog.security.jdk.control.UserResourcePermission "read";
}
```
2. 编写测试类
```java
public class FileAccessController {

    static {
//       -Djava.security.manager
        System.setSecurityManager( new SecurityManager() );
    }

    public static void main(String[] args) throws IOException {
        SecurityManager securityManager = System.getSecurityManager();
        if( securityManager == null ){
            System.out.println("执行文件写入1");
            writeHello("hello");
        }else{
            System.out.println("执行文件写入2");
            AccessController.doPrivileged(new PrivilegedAction<Object>() {
                @Override
                public Object run() {
                    try {
                        writeHello("world");
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                    return null;
                }
            }, AccessController.getContext());
        }
    }

    private static void writeHello(String text) throws IOException {
        FileOutputStream fos = new FileOutputStream("e:\\home\\file.txt", true);
        PrintWriter pw = new PrintWriter(fos);
        pw.println(text);
        pw.close();
        fos.close();
    }

}
```
3. 测试
cd 到target/classes
```bash
java -D"java.security.policy=path\file.policy"  com.sucl.blog.security.jdk.control.file.FileAccessController
```

上面的例子中如果没有定义policy时，如果调用文件写入方法，此时会抛出异常
```
java.security.AccessControlException: access denied ("java.io.FilePermission" "e:\home\file.txt" "write")
```
这样通过自定义polic编写权限策略，则可以实现对资源访问控制的效果。


> 如果看spring的源代码，你会发现有很多类似`AccessController.doPrivileged`这样的写法，不知道当时有没考虑过其目的，虽然在调试时发现其根本没有执行

### 数据加密与解密

Java Security API提供了可互操作的算法和安全服务的实现。服务以provider的形式实现，可以以插件的形式植入应用程序中。
程序员可以透明地使用这些服务，如此使得程序员可以集中精力在如何把安全组件集成到自己的应用程序中，而不是去实现这些安全功能。
此外，除了Java提供的安全服务外，用户可以编写自定义的security provider，按需扩展Java的security平台。

#### 算法

Message digest algorithms 【信息摘要算法, 如：MD5】
Digital signature algorithms 【数字签名算法，DSA】
Symmetric bulk encryption  【对称块加密， 如：DES】
Symmetric stream encryption 【对称流加密， 如：RC4】
Asymmetric encryption 【非对称加密， 如：RSA】
Password-based encryption (PBE) 【密码加密】
Elliptic Curve Cryptography (ECC) 【椭圆曲线加密】
Key agreement algorithms 【key协议算法】
Key generators 【key生成器】
Message Authentication Codes (MACs) 【消息认证码】
(Pseudo-)random number generators 【伪随机数生成器】

#### 示例

Java内置的Provider提供了许多通用的密码算法，比如：RSA, DSA, ECDSA等签名算法、DES, AES, ARCFOUR等加密算法、MD5, SHA-1, SHA-256等
信息摘要算法、还有Diffie-Hellman和ECDH这样的密钥协商算法。下面简单的实现了MD5、DES、RSA几个我们常用的算法

**MD5**
```java
public class MD5 {

   private static final Charset DEFAULT_CHARSET = Charset.defaultCharset();

   static MessageDigest messageDigest;

   static {
      try {
         messageDigest = MessageDigest.getInstance("MD5");
      } catch (NoSuchAlgorithmException e) {
         throw new RuntimeException(e);
      }
   }
}
```

**DES**
```java
public class DES {

    private static final String DES = "DES";

    private static final int DEFAULT_ENCRYPT_CHUNK = 245;

    private static final int DEFAULT_DECRYPT_CHUNK = 256;

    private String salt;

    private SecretKey secretKey;

    public DES(String salt){
        this.salt = salt;
        initKey();
    }

    public void initKey(){
        try {
            KeyGenerator keyGenerator = KeyGenerator.getInstance(DES);
            SecureRandom secureRandom = new SecureRandom(salt.getBytes());
            keyGenerator.init(secureRandom);
            this.secretKey = keyGenerator.generateKey();
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }

    public String encrypt(String text){
        try {
            Cipher cipher = Cipher.getInstance(DES);
            cipher.init(Cipher.ENCRYPT_MODE, this.secretKey);

            byte[] codes = text.getBytes();
            int length = codes.length;

            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int check = 0;
            byte[] cache ;
            while (check < length){
                int chunk = Math.min(DEFAULT_ENCRYPT_CHUNK, length-check);
                cache = cipher.doFinal(codes, check, chunk);
                check += chunk;
                out.write(cache, 0, cache.length);
            }
            return Base64.getEncoder().encodeToString(out.toByteArray());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public String decrypt(String text){
        try {
            Cipher cipher = Cipher.getInstance(DES);
            cipher.init(Cipher.DECRYPT_MODE, this.secretKey);

            byte[] base64Text = Base64.getDecoder().decode(text);

            int length = base64Text.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int check = 0;
            byte[] cache ;
            while (check < length){
                int chunk = Math.min(DEFAULT_DECRYPT_CHUNK, length-check);
                cache = cipher.doFinal(base64Text, check, chunk);
                check += chunk;
                out.write(cache, 0, cache.length);
            }
            return new String(out.toByteArray());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

**RSA**
```java
@Slf4j
public class RSA {

    private static final String KEY_ALGORITHM = "RSA";
    private static final int KEY_PAIR_SIZE = 2048;
    /**
     * 不大于245
     */
    private static final int DEFAULT_ENCRYPT_CHUNK = 245;
    /**
     * 不大于256，改成其他值时：Decryption error
     */
    private static final int DEFAULT_DECRYPT_CHUNK = 256;

    /**
     * NONEwithRSA
     * MD5withRSA
     * SHA256withRSA
     */
    private static final String SIGN_ALGORITHM = "SHA256withRSA";
    private static final String CIPHER_ALGORITHM = "RSA/ECB/PKCS1Padding";

    private KeyPair keyPair;

    public RSA() {
        this.keyPair = getKeyPair();
    }

    /**
     * 生成公钥、私钥
     * @param key
     * @return
     */
    private KeyPair getKeyPair(){
        try {
            KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance(KEY_ALGORITHM);
            keyPairGen.initialize(KEY_PAIR_SIZE);
            return keyPairGen.generateKeyPair();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 获取公钥
     * @param base64PublicKey base64编码的值 基于KeyPair公钥
     * @return
     */
    private PublicKey getPublicKey(String base64PublicKey){
        try {
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
            KeySpec keySpec = new X509EncodedKeySpec(hexToBytes(base64PublicKey)); //
            PublicKey publicKey = keyFactory.generatePublic(keySpec);
            return publicKey;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 获取私钥
     * @param base64PrivateKey base64编码的值 基于KeyPair私钥
     * @return
     */
    private PrivateKey getPrivateKey(String base64PrivateKey){
        try {
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
//            Security.addProvider(BouncyCastleProviderSingleton.getInstance());
            PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(hexToBytes(base64PrivateKey));
            PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
            return privateKey;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public String getPublicKey(){
        return bytesToHex(keyPair.getPublic().getEncoded());
    }

    public String getPrivateKey(){
        return bytesToHex(keyPair.getPrivate().getEncoded());
    }

    /**
     * 通过私钥签名
     * @param data
     * @param privateKeyStr
     * @return
     */
    public String sign(byte[] data){
        PrivateKey privateKey = getPrivateKey(getPrivateKey());
        try {
            Signature sig = Signature.getInstance(SIGN_ALGORITHM);
            sig.initSign(privateKey);
            sig.update(data);
            return bytesToHex(sig.sign());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 校验签名
     * @param data
     * @param sign
     * @param publicKeyStr
     * @return
     */
    public boolean verify(byte[] data, String sign){
        PublicKey publicKey = getPublicKey(getPublicKey());
        try {
            Signature sig = Signature.getInstance(SIGN_ALGORITHM);
            sig.initVerify(publicKey);
            sig.update(data);
            return sig.verify(hexToBytes(sign));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 通过公钥加密
     * @param text
     * @param publicKey
     * @return
     */
    public String encrypt(String text){
        byte[] codes = text.getBytes();
        PublicKey publicKey = getPublicKey(getPublicKey());
        try {
            Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
            int length = codes.length;

            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int check = 0;
            byte[] cache ;
            while (check < length){
                int chunk = Math.min(DEFAULT_ENCRYPT_CHUNK, length-check);
                cache = cipher.doFinal(codes, check, chunk);
                check += chunk;
                out.write(cache, 0, cache.length);
            }
            return bytesToHex(out.toByteArray());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 通过私钥解密
     * @param text 加密内容
     * @param privateKeyStr
     * @return
     */
    public String decrypt(String text){
        PrivateKey privateKey = getPrivateKey(getPrivateKey());
        try {
            byte[] base64Text = hexToBytes(text);
            Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
            cipher.init(Cipher.DECRYPT_MODE, privateKey);

            int length = base64Text.length;
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            int check = 0;
            byte[] cache ;
            while (check < length){
                int chunk = Math.min(DEFAULT_DECRYPT_CHUNK, length-check);
                cache = cipher.doFinal(base64Text, check, chunk);
                check += chunk;
                out.write(cache, 0, cache.length);
            }
            return new String(out.toByteArray());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public byte[] hexToBytes(String str){
        return Base64.getDecoder().decode(str);
    }

    public String bytesToHex(byte[] bytes){
        return Base64.getEncoder().encodeToString(bytes);
    }
}
```

### 身份验证与授权

客户端向服务器发送身份验证请求。
服务器随机生成一个挑战参数，发送给客户端。
客户端使用挑战参数和密码计算出响应参数，并将响应参数发送给服务器。
服务器使用挑战参数、用户名和密码计算出期望的响应参数，并将其与客户端发送的响应参数进行比较，以验证客户端的身份。

+ 身份认证
```java
import javax.security.auth.login.LoginContext;

public class App {

   public static void main(String[] args) {
      URL url = ClassLoader.getSystemClassLoader().getResource("jaas.config");
      System.setProperty("java.security.auth.login.config", url.getPath());

      Subject subject = new Subject();
      subject.getPrincipals().add(new User("admin"));
      
      LoginContext loginContext = new LoginContext("app", subject);
      
      loginContext.login();
   }

}
```

AppLoginModule
```java
@Slf4j
public class AppLoginModule implements LoginModule {

    @Override
    public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, Map<String, ?> options) {
        log.info("initialize");
        // 基于配置构建认证环境
    }

    @Override
    public boolean login() throws LoginException {
        log.info("login");
        return true;
    }

    @Override
    public boolean commit() throws LoginException {
        log.info("commit");
        return true;
    }

    @Override
    public boolean abort() throws LoginException {
        log.info("abort");
        return false;
    }

    @Override
    public boolean logout() throws LoginException {
        log.info("logout");
        return false;
    }
}
```

jaas.config，放到classpath即可
```
app {
    com.sucl.blog.security.jdk.auth.AppLoginModule required
    useTicketCache=true
    doNotPrompt=true;
};
```

> 相关这块知识在网上找了一圈，发现很少有人讲到，在编写示例准备让文心一言给出点提示，发现它只会傻傻的坑我，由于时间原因这块内容也没有过多的深入，
> 通过关键接口的实现，可以看到，在很多成熟的项目中都有被用到，可能jdk本身的模块更多是应用在构建基础系统架构中吧。

### 常用工具类

`java.security` 包是 Java 安全框架的核心部分，上面说到它提供了各种类和接口来支持加密、解密、签名、验证等安全操作。下面简单罗列了常用的工具类及其使用场景。

1. `KeyPairGenerator` 和 `KeyPair`：

    * `KeyPairGenerator` 是用于生成公钥和私钥对的类。它提供了生成 RSA、DSA 等算法的密钥对的接口。
    * `KeyPair` 是由公钥和私钥组成的键对类。它包含公钥和私钥的信息，并提供了操作密钥的方法。
    * 使用场景：用于实现基于公钥加密和数字签名的安全通信，保证数据传输的安全性和身份验证的准确性。
   
2. `Certificate` 和 `CertificateFactory`：

    * `Certificate` 是表示证书的类。它提供了证书的基本信息，如颁发者、持有者、有效期等。
    * `CertificateFactory` 是用于处理证书的类。它提供了解析证书的接口，可以将证书从各种格式转换为 `Certificate` 对象。
    * 使用场景：用于验证远程身份，确保数据传输的安全性。常用于 HTTPS 等安全协议中验证服务器证书。
  
3. `AlgorithmParameters` 和 `AlgorithmParameterGenerator`：

    * `AlgorithmParameters` 是用于处理加密算法参数的类。它提供了获取和设置加密算法参数的方法。
    * `AlgorithmParameterGenerator` 是用于生成加密算法参数的类。它提供了生成特定加密算法参数的接口。
    * 使用场景：用于增强加密算法的强度，提供更多的安全选项。例如，可以用于生成 RSA 加密算法中的密钥长度和填充方式。
  
4. `SecureRandom`：

    * `SecureRandom` 是用于生成随机数的类。它提供了加密强度的随机数生成器，生成的随机数更加安全可靠。
    * 使用场景：用于生成随机数验证码、随机密钥等，提高密码安全性。还可以用于初始化加密算法中的随机数生成器。
 
5. `KeyStore`：

	* `KeyStore` 是用于管理密钥和证书存储的类。它提供了存储和管理私钥、公钥证书等安全凭据的方法。
	* 使用场景：用于管理应用程序的安全凭据，如私钥、公钥证书等。可以用于存储和管理用户的密钥对，保证数据的安全性。

除了上述模块，`java.security` 包还包含其他与安全相关的类和接口，如权限、策略等。这些模块提供了更细粒度的安全控制和配置选项，适用于各种安全场景。
要了解全面的安全功能和用法，请参考 Java 官方文档或相关资源。


### 总结

Java Security技术提供了全面的安全防护机制，包括授权、访问控制、数据加密、身份验证等。通过深入了解和掌握这些技术，
我们可以构建更加安全、可靠的Java应用程序。本文通过详细介绍Java Security的核心模块和代码演示，旨在帮助读者更好地理解和应用这些技术。
未来，随着技术的不断发展，Java Security将会不断完善和改进，为我们的应用程序提供更加安全、可靠的保障。
