---
layout: post
title: Session、Cookie、JWT
tagline: by 无花
categories: sql
tags:
- 无花
---

哈喽，大家好，我是了不起。  

日常开发中或者面试的时候经常会遇到身份验证的问题，下面这些都是关于身份验证和授权的常用技术。

Session、Cookie、JWT、Token、SSO和OAuth 2.0 这些都是什么，在我们的应用程序中有什么用，我们来看看！



<!--more-->

### 前言

互联网应用的构建中，确保用户身份的安全性和便捷性是至关重要的。Session、Cookie、JWT、Token、SSO和OAuth 2.0都是用于身份验证和授权的技术.

找到一张图，形象的展示他们的作用。

![image-20240220215307192](D:\JavaNorth\javanorth\assets\images\2024\wuhua\0220-01.png)



#### Session

**Session**是一种在多个请求之间保持状态的方法。当用户登录成功时，服务器会创建一个Session，生成一个唯一的Session ID，并将其保存在服务器端。同时，服务器会将这个Session ID作为Cookie发送给客户端浏览器存储起来。当用户再次发起请求时，浏览器会自动带上这个Session ID，服务器据此识别用户并恢复其会话状态。

优点：

- 安全性较高，因为Session ID可设置过期时间，并且存储在服务器端。
- 可以跨域共享Session信息。

缺点：

- 需要服务器资源来维护Session数据。
- 分布式系统下Session共享问题复杂。

##### 应用场景：

- 适用于需要维护用户状态的场景，如在线购物网站，用户在浏览商品时，服务器可以通过Session来记录用户的购物车信息。
- 适用于服务端渲染(SSR)的应用，因为Session信息存储在服务器端，适合服务端控制用户会话。
- 适用于对安全性要求较高且能接受服务器资源消耗的场合，因为Session ID存储在服务器端，相对安全。

#### Cookie

**Cookie**是服务器发送到用户浏览器并保存在本地的一小段文本信息。用于识别用户或实现其他功能，通常用来存储Session ID或其他小量的用户信息。每次HTTP请求时，浏览器都会自动将Cookie信息发送给服务器，这样服务器就能识别用户状态。

优点：

- 简单易用，由浏览器自动管理。
- 可以实现自动登录等功能。

缺点：

- 大小受限，一般不超过4KB。
- 存在安全风险，如CSRF攻击。
- 用户可能禁用Cookie。

##### 应用场景：

- 常用于记住用户登录状态，如论坛网站可以记住用户的登录状态，用户下次访问时无需重新登录。
- 适用于跟踪用户行为，比如广告公司可能会用Cookie来收集用户在不同网站上的行为数据。

#### JWT (JSON Web Token)

**JWT**是一种开放标准，用于在网络上安全地传输信息。它由三部分组成：头部（Header）、有效载荷包含用户信息和其他数据。签名用于验证消息的完整性和来源。。

优点：

- 无状态（Stateless），不需要存储在服务器上。
- 可以轻松实现跨域认证。
- 可以生成自包含的访问令牌，减少网络请求。

缺点：

- 若泄露则安全性降低，因为不依赖于服务器的存储。
- 较难作废单个Token。

##### 应用场景：

- 适用于无状态认证场景，尤其是在RESTful API和微服务架构中，JWT可以被用来在客户端和服务器之间安全地传输信息。
- 适用于跨域认证，因为JWT可以编码所有 claims 信息，方便在分布式系统中传递用户身份信息。

#### Token

在身份验证和授权领域，**Token**是一个广泛的概念，可以是JWT，也可以是其他形式的访问令牌。Token通常由服务器生成，并包含了用户的权限或者身份信息。客户端在后续请求中使用该Token来证明自己的身份或获取授权。

优点：

- 提供了一种简单有效的方式来实现API的安全访问。
- 可以独立于用户会话进行身份验证。

缺点：

- 需要妥善保管，避免泄露导致安全风险。

#### SSO (Single Sign-On)

单点登录，是一种让用户使用一组凭据（如用户名和密码）登录一次，即可访问多个应用程序的技术。这样可以避免用户为每个应用程序重复输入凭据。

优点：

- 提升用户体验，避免了重复登录的不便。
- 减少了用户记住多个账号的需求。

缺点：

- 一旦SSO提供商被破解，所有关联服务都面临安全威胁。
- 实施和维护成本可能较高。

##### 应用场景：

- 适用于企业环境中，员工可以使用一组凭据访问多个内部系统，如微软的Active Directory就可以实现SSO功能。
- 适用于为用户提供便捷的登录体验，例如Google账户可以登录所有支持Google SSO的服务。

#### OAuth 2.0

是一个开放标准，用于授权第三方应用程序访问用户的资源 。它允许第三方应用程序可以使用这个令牌访问用户存储在某一服务提供商上的信息，而无需分享用户的凭据。OAuth 2.0有四种授权流程：授权码、隐式、密码和客户端凭证。

优点：

- 提供了明确的授权机制，用户可以精确控制数据访问权限。
- 支持多种应用场景和流程。

缺点：

- 协议相对复杂，实施难度较大。
- 需要良好的安全措施以保护授权过程中的敏感信息。

##### 应用场景：

- 适用于授权第三方应用访问用户数据的场景，如用户可以通过OAuth 2.0授权社交媒体账号登录其他应用。
- 适用于构建开放平台时，允许开发者通过OAuth 2.0获取用户数据来开发第三方应用。github，qq授权等。








