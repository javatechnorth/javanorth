---
layout: post
title:  函数引擎
tagline: by simsky
categories: 架构设计 FaaS
tags: 
    - simsky

---

大家好，我是指北君，最今年IaaS，SaaS，PaaS都非常火爆，也是主流商业发展的方向，今天指北君给大家介绍与此类似的FaaS平台。

<!--more-->
### 概述
通常，我们服务的承载通过应用服务在服务器、虚拟机或容器中按照业务功能开发并提供相应的接口，这种形式开发团队针对业务需求分析、设计并完成开发、测试和部署上线，以往大多数的业务的交付模式都是按照这种方式进行的，无论是独立定制应用服务还是SaaS化应用。但是在面临大量客户定制程度高的场景，往往研发的定制成本非常高，而且交付的周期也无法满足客户的需要。
基于无服务架构（serverless），提供业务功能服务动态构建的能力，是解决服务定制和扩展的有效方法。

### FaaS
无服务架构核心功能包含：基于API网关提供API服务，API开发平台提供自定义的API开发和管理、以及核心FaaS引擎。FaaS平台核心是函数引擎和调度管理，本次介绍一个可以基于开源项目快速搭建的FaaS引擎。
最简单的函数引擎我们可以使用Groovy进行验证和构造，但是Groovy无法满足平台及的诉求，平台级项目除了基本功能外，还需要考虑并发、高可用、可扩展等质量属性。因此Groovy可以作为执行引擎，但其他能力则需要配套相关的部件来完成。本次，指北君将为大家介绍OpenWhisk项目，基于该项目可以构建商业化平台及FaaS项目。

![FaaS引擎](/assets/images/2021/simsky/architect-faas-1-1.png)

集成平台的函数式引擎，除了基本脚本执行引擎外，还需要支持大批量请求并发，动态扩展等非功能性需求，因此函数式引擎包含如下模块：执行调度，集群管理（管理，监控，动态伸缩），执行引擎。
完整方式的工作原理图如下：
 
![函数式引擎工作原理图](/assets/images/2021/simsky/architect-faas-1-2.png)

### OpenWhisk
OpenWhisk是属于 Apache 基金会的开源 FaaS 计算平台, 由 IBM 在2016年公布并贡献给开源社区。从业务逻辑上看，OpenWhisk 同 AWS Lambda 一样，为用户提供基于事件驱动的无状态的计算模型，并直接支持多种编程语言。

特点
1. 高性能、高扩展性的分布式Faas计算平台
2. 函数的代码及运行是全部在Docker容器中进行，利用Docker引擎实现Faas函数运行的管理、负载均衡和扩展
3. 同时，Openwhisk架构中的所有其他组件（API网关、控制器、触发器）也全部运行在Docker容器中，这使得其全栈可以容易的部署在IAAS/PAAS平台上 。
4. 更重要的是，相比其他Faas实现，Openwhisk更像是一套完整Serverless解决方案，除了容易调用和函数管理，Openwhisk还包括了身份验证/鉴权、函数异步触发等功能。

### 工作原理
在openwhisk中集成了一些非常成熟的组件：Nginx，Kafka，Docker，CouchDB等，下面将为大家描述系统的整个执行过程：
以下面脚本块为例：demo.js：
```js
function main() {
    console.log('Demo');
    return { demo: 'testing' };
}
```

使用下面命令创建动作
wsk action create myDemo demo.js
现在，我们可以使用以下命令调用该操作：
wsk action create myDemo demo.js
那么我现在来看看在Openwhisk中具体发生了什么，整个过程如下图所示：

![处理流程(来至官网)](/assets/images/2021/simsky/architect-faas-1-3.png)

#### API网关
Openwhisk通过API对外提供服务，通过wsk-cli转换为基于HTTP的API请求调用：
POST /api/v1/namespaces/$userNamespace/actions/myDemo
Host: $openwhiskEndpoint

#### 控制器
控制器用于请求和任务的控制管理，接受网关转发的外部请求，并根据请求内容控制后续所有的操作过程。

#### 认证和鉴权
控制器将请求转发给认证和鉴权模块验证用户的身份和权限。用户的身份信息（credentials）保存在CouchDB的用户身份数据库中，验证无误后，控制器进行下一步处理。

#### 函数执行信息
验证通过后，控制器从CouchDB中加载API请求对应的操作对象（myDemo）。操作记录主要要执行的代码和要传递给操作的默认参数，并与实际调用请求中包含的参数合并。它还包含执行时对其施加的资源限制，例如允许使用的内存。

#### 执行器服务治理
通过上面两步，控制器已经获取了函数执行所需要的全部信息，控制器从Consul获取处于空闲状态的执行器，然后将执行信息发送给执行器Invoker。  
请求通过Kafka发送，当控制器得到 Kafka 收到请求消息的确认后，会直接向发出请求的用户返回一个 ActivationId，当用户收到确认的 ActivationId，即可认为请求已经成功存入到 Kafka 队列中。用户可以稍后通过 ActivationId 查询函数运行结果。

#### 执行器运行
Invoker从对应的Kafka topic中接受控制器传来的请求，会生成一个Docker容器，注入动作代码，试用传递给他的参数执行它，获取结果，消灭容器。这也是进行了大量性能优化以减少开销并缩短响应时间的地方。

#### CouchDB存储请求结果
Invoker的执行结果最终会被保存在CouchDB的whisk数据库中，格式如下所示：
```js
{
   "activationId": "f120dfb9dc93a6f6fc9de27ebd44c4b9",
   "response": {
       "statusCode": 0,
       "result": {
           "demo": "testing"
       }
   },
   "end": 1474689415621,
   "logs": [
       "2021-07-21T12:06:21.922343386Z stdout: Demo"
   ],
   "start": 1474689415595,
}
```

保存的结果中包括用户函数的返回值，及日志记录。控制器通过activationID取回函数运行结果，然后返回给用户。

### 扩展点
基于原生能力，脚本功能有较大的受限，在共用能力抽象等模块化能力方面存在一定的不足，因此需要基于运行环境提供增强的公共模块化能力，公共模块化能力支持组件化技术，可衍生为组件能力市场。

### 总结

基于Openwhisk的FaaS技术，本次就介绍到这里。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！