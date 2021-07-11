---
layout: post
title:  iCloud详解——键值存储
tagline: by simsky
categories: 架构设计 iCloud
tags: 
    - simsky

---

大家好，我是指北君，之前为大家介绍了iCloud的基础知识，相信小伙伴们对iCloud的一些机制原理也有了一定的了解。今天，指北君将继续为大家深入探索iCloud，一步步揭开iCloud的神秘面纱。

<!--more-->
### 前言
在iCloud的第一篇中，我们看到iCloud通过容器构建了一个安全、便捷的系统，为应用提供多端协同的强大能力。

### 概念
键值存储，简单的理解就是一个key-value存储，比较适合键值存储的比如首选项，应用的配置和状态等。在实际使用中，开发者将应用的配置，用户自定义选项等使用键值存储，然后通过iCloud同步到所有设备上，这样我们在其他设备上使用应用时就会得到一样的使用体验。并且只要我们任何设备更改相应的配置，其他设备上的该应用也会更新。

![iCloud容器](/assets/images/2021/simsky/architect-icloud-2-1.png)

在单个设备内，iCloud访问键值存储如下图所示

![iCloud容器](/assets/images/2021/simsky/architect-icloud-2-2.png)

在你使用NSUserDefaults对象时，你可以考虑使用iCloud的键值存储来保存对应的值或者属性列表，比如Bool，Double，Int64，String，Data这些基础类型，以及Array和Dictionary，数组和字典值可以包含所有基本类型。 NSUbiquitousKeyValueStore提供对应数据的读取和写入方法。

每次写入key-value数据，操作成功或失败都是原子的；要么写入所有数据，要么不写入。当应用需要确保一组值同时生效时，可以将相互依赖的值放在数组或字典中。

### 开启键值存储
如果要在应用中使用键值存储，则应用需要具有相应的权利。 在Xcode中开启iCloud访问权限。

### 准备使用
在你设备的上的应用，只要连接到你的iCloud账户，都可以修改键值并更新到iCloud云端。要获取这种变更，可以在应用启动时注册NSUbiquitousKeyValueStoreDidChangeExternallyNotification通知。为确保应用获取最新数据，通过调用synchronize方法从iCloud云端获取最新的键和值。
将synchronize代码放在您的application:didFinishLaunchingWithOptions:方法 (iOS) 或applicationDidFinishLaunching:方法 (OS X) 中。
在通知的处理流程中，检查信息并确定是否将更改写入应用。

### 键值冲突
当应用将本地的键值更新上传到iCloud时，iCloud会检查其他设备是否对键值存储进行了任何更改。如果没有何更改，则NSUbiquitousKeyValueStore将更新写入到服务器。如果进行了更改，则不会写入并产生NSUbiquitousKeyValueStoreDidChangeExternallyNotification通知，应用可以根据自身要求进行相应处理，比如应用强制更新本地数据，或者强制将本地数据同步到云端。

![键值冲突](/assets/images/2021/simsky/architect-icloud-2-3.png)

应用在接收到NSUbiquitousKeyValueStoreDidChangeExternallyNotification通知时，应用需要验证云端和本地数据，确保给出足够有意义的信息帮助用户选择以本地还是云端数据为准。如果云端数据与应用本地状态不匹配，还需要考虑云端上其他设备的数据是否有效。例如，如果用户在一台设备上的等级为13级，而在另一台设备上是5级，则选择13级的对应的数据显然更符合最新数据的可能。

### 容量计算和限制
单个应用使用iCloud的键值存储总空间1MB，最大键数量为1024，单个键的大小限制为1MB。比如，如果单个键存储一个为1MB的值，则达到了总空间限制，则无法再新增键。
键采用UTF8编码，长度最大为64字节，键的容量大小不计入1MB键值存储空间中；但是呢，键值的容量（最多消耗64KB）计入iCloud容量中。

![容量计算](/assets/images/2021/simsky/architect-icloud-2-4.png)

如果应用在键值存储中超出了容量限制，则iCloud键值存储会触发通知：NSUbiquitousKeyValueStoreDidChangeExternallyNotification:NSUbiquitousKeyValueStoreQuotaViolationChange。

### 使用对象的注意事项
尽管可以使用NSData对象作为键值存储的值，但需要谨慎操作。
首要问题是空间，如果您的数据对象很大，您的应用可能会很快超出存储限制（1MB）。对于这种场景，需要考虑使用文档存储来满足业务场景。

其次，每次对大数据对象进行小的更改时，都必须将整个数据对象发送到iCloud。与其将大量数据捆绑到单个数据对象中，不如将这些数据分成小块并单独存储这些数据，最好使用更透明的类型，例如数字和字符串。使用其他类型的属性列表对象更精细地存储数据可以最大限度地减少在进行小的更改时必须传输的数据量。

最后，由于数据对象是自定义对象，因此只有您的应用程序知道如何读取它们。当您更改数据格式时，旧版本的应用程序在尝试读取使用新格式的数据对象时可能会中断。为了防止旧版本的应用程序崩溃，您可能需要在数据对象中包含版本信息，如健壮性和跨平台兼容性设计中所要求的的。

### 不适合键值存储的场景
每个应用都可以使用键值存储来在多设备同步数据，但某些类型的数据不适合key-value存储。比如，对于使用文档的应用，不适合使用键值存储来存储每个文档的状态信息，如当前页面或当前选择，这些数据适合根据需要将特定于文档的状态存储在每个文档中。

另外，对于离线场景有较高要求的应用，也需要慎重考虑键值存储，要避免因为无法更新云端数据导致本地应用不可用的情况，需要应用可以依赖本地数据就能正常运行。

### 总结

以上就是指北君本期介绍的内容，比较粗略地介绍了iCloud中的键值存储设计。

我是指北君，操千曲而后晓声，观千剑而后识器。感谢各位人才的：点赞、收藏和评论，我们下期更精彩！

