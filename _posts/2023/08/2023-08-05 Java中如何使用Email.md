---
layout: post
title:  2023-08-05 Java中如何使用Email
tagline: by 沉浮
categories: 
tags: 沉浮
---


<!--more-->
## Email

电子邮件早已成为工作生活中不可缺少的部分，每个工作的人都会有自己的私人邮箱或企业邮箱，用来协助我们处理生活事务以及实现工作中的交流。

今天主要通过简单的示例，了解在Java中如何使用API来完成邮件的接收与发送。

通过该篇文章我们可以有如下收获：
1. 了解基于Java的电子邮件客户端的实现方式
2. 了解常见的邮箱如何集成
3. 认识邮箱中的IMAP与POP协议

### 适用场景

邮件和短信很像，将信息发送到目的用户，不需要用户在线，基于邮件服务器，完成消息的存储与转发。
一般公司都会有自己的企业邮箱，主要也是为了保证数据的安全性。可能你平时在注册网站时，需要通过邮件来接收验证消息完成认证流程；
或者每天打开邮箱收到的各种订阅消息等等。

1. 基于电子邮件的通信与交流
2. 接收验证消息，实现用户认证
3. 发送邮件提供消息通知

### 说明

电子邮件在Internet上发送和接收的原理与我们通过邮局发信件非常相类似：首先要找到任何一个邮局，填写邮件收件人姓名、地址等信息，
之后信件就会寄到收件人所在地的邮局，对方需要到相应的邮局才能取出信件。同样，在发送电子邮件时，邮件是由邮件发送服务器发出，
根据收信人的地址匹配目的邮件接收服务器，收信人收取邮件需要访问这个服务器才能取件。

邮件的发送与接收都需要基于特定的通信协议，发邮件时基于SMTP协议，收邮件时基于POP3、IMAP协议。

  + SMTP   
  SMTP 的全称是“Simple Mail Transfer Protocol”，即简单邮件传输协议，是用于发送电子邮件的协议。
  它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。SMTP 协议属于 TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。SMTP 服务器就是遵循 SMTP 协议的发送邮件服务器。

  + IMAP  
  IMAP（Internet Message Access Protocol）Internet邮件访问协议，是用于接收电子邮件的协议。
  IMAP不用对服务器上面的邮件进行全部下载（根据实际需要进行下载），可以通过邮件客户端对邮件进行操作；IMAP提供了WebMail与邮件客户端之间的双向通信，以及客户端上的操作（如阅读、删除、移动邮件等）。

  + POP3  
  POP3（Post Office Protocol version 3）邮局协议的第3个版本，同样用于接收电子邮件的协议。
  POP3可以让你下载邮件服务器上的邮件（下载所有未读邮件），在邮件从服务器发送到电脑的同时删除邮件服务器上的邮件（目前很多邮件服务器都支持“下载邮件，不删除邮件，或者发出提醒”）。

`
  POP允许电子邮件客户端下载服务器上的邮件，但是您在电子邮件客户端的操作（如：移动邮件、标记已读等），这是不会反馈到服务器上的，
比如：您通过电子邮件客户端收取了QQ邮箱中的3封邮件并移动到了其他文件夹，这些移动动作是不会反馈到服务器上的，也就是说，QQ邮箱服务器上的这些邮件是没有同时被移动的。
但是IMAP就不同了，电子邮件客户端的操作都会反馈到服务器上，您对邮件进行的操作（如：移动邮件、标记已读等），服务器上的邮件也会做相应的动作。也就是说，IMAP是“双向”的。
同时，IMAP可以只下载邮件的主题，只有当您真正需要的时候，才会下载邮件的所有内容。
`

*如果感兴趣可以深入了解这几个协议的具体实现与规范，这里我们只用知道，与邮箱服务器对接时，是基于这几个协议来实现通信，什么时候用什么协议即可。后面示例中会有用到。*

### 邮箱与协议

如果要完成邮件的发送，我们需要知道*用户*通过*服务器*将邮件发送给*谁*，这里的用户指的是发件方，需要明确我们的发件地址，
谁即对方的邮箱地址，邮箱地址主要邮3个部分组成，**用户名**@**邮件服务器域名**，比如123456@qq.com，tom@gmail.com等等，
上面说到的服务器与域名对应。

在编写示例前，需要先了解我们用到邮箱的一些信息，比如实现基于qq邮箱的邮件发送以及收取时，我们必须知道其邮箱服务器对应的协议服务地址以及端口，
下面是几个常见的协议信息：

+ 126邮箱

| 协议类型 | 协议功能 | 服务器地址        | 非SSL端口 | SSL端口号  |
|------|------|--------------|:------:|:-------:|
| SMTP | 发送邮件 | smtp.126.com |   25   | 465、994 |
| POP  | 接收邮件 | pop.126.com  |  110   |   995   |
| IMAP | 接收邮件 | imap.126.com |  143   |   993   |

+ 163邮箱

| 协议类型 | 协议功能 | 服务器地址        | 非SSL端口 | SSL端口号 |
|------|------|--------------|:------:|:------:|
| SMTP | 发送邮件 | smtp.163.com |   25   |  465   |
| POP  | 接收邮件 | pop.163.com  |  110   |  995   |
| IMAP | 接收邮件 | imap.163.com |  143   |  993   |

+ QQ邮箱

| 协议类型 | 协议功能 | 服务器地址       | 非SSL端口 | SSL端口号  |
|------|------|-------------|:------:|:-------:|
| SMTP | 发送邮件 | smtp.qq.com |   25   | 465、587 |
| POP  | 接收邮件 | pop.qq.com  |  110   |   995   |
| IMAP | 接收邮件 | imap.qq.com |  143   |   993   |

+ Gmail邮箱

| 协议类型 | 协议功能 | 服务器地址          | 非SSL端口 | SSL端口号  |
|------|------|----------------|:------:|:-------:|
| SMTP | 发送邮件 | smtp.gmail.com |        | 465、587 |
| POP  | 接收邮件 | pop.gmail.com  |        |   995   |
| IMAP | 接收邮件 | imap.gmail.com |        |   993   |

### 实例

在Java中我们可以基于JavaMail API实现邮件的发送与读取，由于我使用的是JDK17，所以选用的是jakarta.mail.jar完成今天的示例。

在Spring中同样提供了邮件的支持，我们可以在项目中通过引入*spring-boot-starter-mail*来集成，下面分别来看下如何实现邮件的收发功能。
示例以QQ邮件为例，比如我的邮箱地址为409835152@qq.com，下面来看看具体实现过程

+ 发送邮件

1. 引入依赖

```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
      <version>${spring-boot.version}</version>
  </dependency>
```
2. 添加application配置
```yaml
spring:
  mail:
    host: smtp.qq.com
    port: 25
    protocol: smtp
    username: 409835152@qq.com
    password: '******'
```
这里主要配置了邮箱地址，和上面说到的协议类型、服务地址以及端口，最后还有一个密码，注意这里不是邮箱登录密码，我们需要单独申请，这个在各个邮箱中都有申请入口，比如qq邮箱中：
![img.png](/assets/images/2023/sucls/08_05/qq-1.png)
点击“管理服务”在新的页面中通过“生成授权码”按流程申请即可，**注意不要泄露！！！**
![img.png](/assets/images/2023/sucls/08_05/qq-2.png)

1. 编写邮件发送服务
```java
@Service
public class EmailQQService {

    @Resource
    private JavaMailSender javaMailSender;

    @Resource
    private MailProperties mailProperties;

    public void sendEmail(Email email){
      SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
      simpleMailMessage.setFrom(mailProperties.getUsername()); //设置发送邮件账号
      simpleMailMessage.setTo(email.getTo()); //设置接收邮件的人，可以多个
      simpleMailMessage.setSubject(email.getSubject()); //设置发送邮件的主题
      simpleMailMessage.setText(email.getText()); //设置发送邮件的内容
      javaMailSender.send(simpleMailMessage);
    }
}    
```
主要指定发送目标对象的邮箱地址，邮件主题以及邮件内容等即可。可以看到，基于spring提供的工具，邮件的发送变得非常简单。

+ 邮件的接收

在Spring中没有提供这样的工具类，需要我们自己写：
```java
@Service
public class QqEmailService {
    
    public List<Email> receiveEmail() throws MessagingException, IOException {
        Properties properties = configProperties();
        Store store = createStore( properties );
        List<Email> emails = receive(store);
        store.close();
        return emails;
    }

}    
```
1. 添加接收服务相关的配置，包括协议、服务地址、端口
```
    private Properties configProperties(){
        // 配置邮件服务器
        Properties properties = new Properties();
        properties.setProperty("mail.store.protocol", receiveMailProperties.getProtocol());
        properties.setProperty("mail.imap.host", receiveMailProperties.getHost());
        properties.setProperty("mail.imap.port", receiveMailProperties.getPort());
        return properties;
    }
```
2. 创建Session与Store
```
    private Store createStore(Properties properties) throws MessagingException {
        // 创建Session实例对象
        Session session = Session.getInstance( properties );
        // 创建IMAP协议的Store对象
        Store store = session.getStore("imap");
        // 连接邮件服务器
        store.connect(mailProperties.getUsername(), mailProperties.getPassword());
        return store;
    }
```
3. 从服务器读取邮件
```
    private List<Email> receive(Store store) throws MessagingException, IOException {
        // 获得收件箱
        Folder folder = store.getFolder("INBOX");
        // 以读写模式打开收件箱
        folder.open(Folder.READ_WRITE);
        // 各状态邮件数量
        System.out.println(String.format("收件箱邮件总数：%s，其中，新邮件数：%s，未读邮件数：%s，",folder.getMessageCount(), folder.getUnreadMessageCount(), folder.getNewMessageCount()));
        // 获得收件箱的邮件列表
        Message[] messages = folder.getMessages(folder.getMessageCount()-5, folder.getMessageCount());
        System.out.println("------------------------开始解析邮件----------------------------------");
        List<Email> emailList = new ArrayList<>();
        for (Message message : messages) {
            Email email = new Email()
                    .setFrom(Arrays.stream(message.getFrom()).map(address -> ((InternetAddress)address).getAddress()).collect(Collectors.joining()))
                    .setSubject(message.getSubject())
                    .setContentType(message.getContentType())
                    .setSendDate(message.getSentDate())
                    .setReceiveDate(message.getReceivedDate());


            System.out.println(String.format(">>>>> 邮件来自：%s，主题：%s，接收时间：%s", email.getFrom(),
                    email.getSubject(),
                    DateFormatUtils.format(email.getReceiveDate(), DateFormatUtils.ISO_8601_EXTENDED_DATETIME_FORMAT.getPattern()))
            );
            email.setEmailContents(resolveMessage(message.getContentType(), message));
            System.out.println(String.format("邮件内容：%s" , email.getEmailContents()));
            emailList.add(email);
        }
        // 关闭资源
        folder.close(false);
        return emailList;
    }
```
4. 解析邮件内容，邮件除了文字，还有图片，需要根据消息内容类型进行解析，当然发送消息的时候，同样支持各种类型的消息，具体可以**JavaMailSender**的实现类
```
    private List<EmailContent> resolveMessage(String contentType, Message message) throws MessagingException, IOException {
        List<EmailContent> emailContents = new ArrayList<>();
        resolveMessageContent( message.getContent(), message, emailContent->{
            emailContents.add(emailContent);
        } );
//        return content.toString();
        return emailContents;
    }

    private void resolveMessageContent(Object content, Object parent, Consumer<EmailContent> emailContentConsumer) throws MessagingException, IOException {
        if( content instanceof String ){
            emailContentConsumer.accept( new EmailContent(EmailContent.Type.TEXT, (String) content) );
        }else if( content instanceof MimeMultipart){
            MimeMultipart multipart = (MimeMultipart) content;
            int count = multipart.getCount(), index = -1;
            while ( count > ++index ){//  0：纯文本；1： html内容
                BodyPart bodyPart = multipart.getBodyPart(index);
                Object partContent = bodyPart.getContent();
                resolveMessageContent( partContent, bodyPart, emailContentConsumer);
            }
        }else if( content instanceof BASE64DecoderStream){
            File file = new File(((IMAPBodyPart) parent).getFileName());
            ((BASE64DecoderStream) content).transferTo( new FileOutputStream( file ) );
            emailContentConsumer.accept( new EmailContent(EmailContent.Type.FILE, file.getAbsolutePath()) );
        }else {
            System.out.println(">>>>>>>>>>>>>>>> 邮件内容类型： "+ content.getClass() );
            emailContentConsumer.accept( new EmailContent(EmailContent.Type.TEXT, content.toString()) );
        }
    }
```
5. 关闭store
```
    store.close();
```

代码有点多，但是流程不复杂且比较清晰。到这里一个简单的针对qq邮箱的邮件发送与接收示例就完成了。不管是收邮件还是发邮件其关键点是：
1) 邮件收发对应的协议类型、服务地址、服务端口
2) 发送邮件用户的邮箱地址与授权码
3) 目标邮箱地址

剩下的都是些简单API调用的过程

### 结束语

最近学习些桌面端应用，看到了网易邮箱大师，顺便了解了下载Java中如何写一个客户端实现与各个邮箱服务器间完成邮件的读取与接收。
这样通过其他语言实现时也知道了主要流程与必要的相关配置信息。