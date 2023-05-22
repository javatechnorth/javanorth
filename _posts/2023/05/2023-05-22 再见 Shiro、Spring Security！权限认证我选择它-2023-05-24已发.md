---
layout: post
title:  再见 Shiro、Spring Security！权限认证我选择它
tagline: by 付义帆
categories: Java开源项目
tags:
- 付义帆
---

哈喽，大家好，我是了不起。  

Java有很多优秀的权限认证框架，如`Apache Shiro`、`Spring Security`等，但是集成起来实在是有些复杂；今天给大家介绍一个轻量级的权限认证框架：Sa-Token，只需引入依赖即可使用，接下来让我们进一步了解它。
<!--more-->

### 初识sa-token

**Sa-Token** 是一个轻量级 Java 权限认证框架，旨在以简单、优雅的方式完成系统的权限认证部分，主要解决：**登录认证**、**权限认证**、**单点登录**、**OAuth2.0**、**分布式Session会话**、**微服务网关鉴权** 等一系列权限相关问题。

![sa-token](https://www.javanorth.cn/assets/images/2023/fu/sa-token-js4.png)

### Sa-Token 功能

Sa-Token 目前主要五大功能模块：登录认证、权限认证、单点登录、OAuth2.0、微服务鉴权。

1. **登录认证** —— 单端登录、多端登录、同端互斥登录、七天内免登录
2. **权限认证** —— 权限认证、角色认证、会话二级认证
3. **Session会话** —— 全端共享Session、单端独享Session、自定义Session
4. **踢人下线** —— 根据账号id踢人下线、根据Token值踢人下线
5. **账号封禁** —— 登录封禁、按照业务分类封禁、按照处罚阶梯封禁
6. **持久层扩展** —— 可集成Redis、Memcached等专业缓存中间件，重启数据不丢失
7. **分布式会话** —— 提供jwt集成、共享数据中心两种分布式会话方案
8. **微服务网关鉴权** —— 适配Gateway、ShenYu、Zuul等常见网关的路由拦截认证
9. **单点登录** —— 内置三种单点登录模式：无论是否跨域、是否共享Redis，都可以搞定
10. **OAuth2.0认证** —— 轻松搭建 OAuth2.0 服务，支持openid模式
11. **二级认证** —— 在已登录的基础上再次认证，保证安全性
12. **Basic认证** —— 一行代码接入 Http Basic 认证
13. **独立Redis** —— 将权限缓存与业务缓存分离
14. **临时Token认证** —— 解决短时间的Token授权问题
15. **模拟他人账号** —— 实时操作任意用户状态数据
16. **临时身份切换** —— 将会话身份临时切换为其它账号
17. **前后端分离** —— APP、小程序等不支持Cookie的终端
18. **同端互斥登录** —— 像QQ一样手机电脑同时在线，但是两个手机上互斥登录
19. **多账号认证体系** —— 比如一个商城项目的user表和admin表分开鉴权
20. **Token风格定制** —— 内置六种Token风格，还可：自定义Token生成策略、自定义Token前缀
21. **注解式鉴权** —— 优雅的将鉴权与业务代码分离
22. **路由拦截式鉴权** —— 根据路由拦截鉴权，可适配restful模式
23. **自动续签** —— 提供两种Token过期策略，灵活搭配使用，还可自动续签
24. **会话治理** —— 提供方便灵活的会话查询接口
25. **记住我模式** —— 适配[记住我]模式，重启浏览器免验证
26. **密码加密** —— 提供密码加密模块，可快速MD5、SHA1、SHA256、AES、RSA加密
27. **全局侦听器** —— 在用户登陆、注销、被踢下线等关键性操作时进行一些AOP操作
28. **开箱即用** —— 提供SpringMVC、WebFlux等常见web框架starter集成包，真正的开箱即用

### 简单示例

1、引入依赖

> 注：如果你使用的是 `SpringBoot 3.x`，只需要将 `sa-token-spring-boot-starter` 修改为 `sa-token-spring-boot3-starter` 即可。

```xml
<!-- Sa-Token 权限认证，1.34.0 已是最新版本 -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot-starter</artifactId>
    <version>1.34.0</version>
</dependency>
```

yaml配置

```yaml
server:
    # 端口
    port: 8081
    

sa-token: 
    # token名称 (同时也是cookie名称)
    token-name: satoken
    
# 用 sa-token 默认的配置即可，也可根据业务自行修改
```



2、编写测试代码

```java
@RestController
@RequestMapping("/user/")
public class UserController {

    @RequestMapping("doLogin")
    public String doLogin(String username, String password) {
        if("zhang".equals(username) && "123456".equals(password)) {
            StpUtil.login(10001);
            return "登录成功";
        }
        return "登录失败";
    }

    @RequestMapping("isLogin")
    public String isLogin() {
        return "当前会话是否登录：" + StpUtil.isLogin();
    }
    
}
```

3、测试

启动代码，从浏览器依次访问上述测试接口

![](https://www.javanorth.cn/assets/images/2023/fu/test-do-login.png)

![](https://www.javanorth.cn/assets/images/2023/fu/test-is-login.png)

### 小结

看得出来，比起Shiro、SpringSecurity这些被广泛使用的鉴权项目，这个项目的集成使用方式可以简单到令人发指。

今天就介绍到这里了，感兴趣的小伙伴们可以去看官方文档，进一步了解它。

> Sa-Token 官方文档
>
> https://sa-token.cc/doc.html

