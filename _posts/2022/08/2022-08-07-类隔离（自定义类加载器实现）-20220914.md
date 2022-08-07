---
layout: post
title: 类隔离（自定义类加载器实现）
tagline: by Greatom
categories: JVM
tags: Greatom
---

哈喽，大家好，我是指北君。又是提升自己的一天，先来段向上语录
> 自觉心是进步之母，自贱心是堕落之源，故自觉心不可无，自贱心不可有。——邹韬奋

给自己打完气，接下来就开始分享啦。请注意查收！！！

### 1、前言
由于微服务的快速迭代、持续集成等特性，越来越多的团队更倾向于它。但是也体现出了一些问题，比如在基础设施建设过程中，需要把通用功能下沉，把现有大而全的基础设施按领域拆分，考虑需要兼容现有生产服务，会产生不同的依赖版本，有时不注意就可以引发问题。比如本文遇到的依赖包版本冲突问题，以及利用如何类隔离技术解决的分析。



### 2、类隔离是什么？
类隔离是一种通过类加载器实现加载所需类的实现方式，使得不同版本类间隔离，避免了使用冲突问题，最终的效果就是不同模块的内容被不同的类加载器加载，满足同一环境下同时兼容不同接口实现类。

### 3、使用场景
- 比如业务服务A和业务服务B均需要消息通知等，均依赖消息中间件，但所引用版本不一致，导致最终只有一个版本加载到JVM，在某一个服务调用时会出现 NoSuchMethodError或NoSuchClassError问题，这就很难排查出来，没准会影响项目进度，最终月度的绩效（“鸡腿”）不保。
服务A pom.xml：
```
		<!-- common-message-->
        <dependency>
            <groupId>com.lgy</groupId>
            <artifactId>spring-common-message</artifactId>
            <version>1.0.0<version>
        </dependency>
```
服务B pom.xml：
```
		<!-- common-message-->
        <dependency>
            <groupId>com.lgy</groupId>
            <artifactId>spring-common-message</artifactId>
            <version>2.0.0<version>
        </dependency>
```
- 业务调用流程：
```c
	// 业务A调用微信服务通知
	MessageUtil.sendMessage(content,peopleId,templateId,"wechat");
	// 业务B调用微信服务通知
	MessageUtil.sendToWechat(content,peopleId,templateId);
```
- JVM最终加载的为 2.0.0 版本的依赖，导致业务A在调用时抛异常java.lang.NoSuchMethodError。

### 4、解决方案
- 大体的解决思路就是，在不改变业务代码的前提下， 业务A调用 1.0.0 版本的消息工具类， 业务B调用2.0.0版本的消息工具类，因此需要JVM能够利用自定义类加载器加载所需的类或关联的类。
- 实现思路
	- 重写类加载器，实现自定义类加载（java.lang.ClassLoader）
	- 重写类加载函数
		- 重写 findClass(String name)
		- 重写 loadClass(String name)
> 涉及的知识点
	- JVM加载过程：加载-》链接-》初始化（具体后续介绍）
	- 双亲委派机制：委托父加载器查询；如果父加载器查询不到，则调用自身的findClass加载

#### 4.1 重写findClass：
```c
	import java.io.*;
	import java.util.HashMap;
	import java.util.Map;

	public class CustomerFindClass extends ClassLoader {
		private Map<String, String> classPathMap = new HashMap<>();
		public CustomerFindClass() {
			// 业务A的自定义类加载器
			classPathMap.put("com.lgy.businessA.service.impl.MessageServiceImpl", "E:/dataway-demo/example/target/classes/com/lgy/businessA/service/impl/MessageServiceImpl.class");
			classPathMap.put("com.lgy.v1.message.util.MessageUtil", "E:/dataway-demo/example/target/classes/com/lgy/v1/message/util/MessageUtil.class");
		}
		
		/**
		* findClass方式加载类
		*/
		@Override
		protected Class<?> findClass(String name) throws ClassNotFoundException {
			String classPath = classPathMap.get(name);
			File file = new File(classPath);
			if (!file.exists()) {
				throw new ClassNotFoundException();
			}
			byte[] bytes = getClassData(file);
			if (null == bytes || 0 == bytes.length) {
				throw new ClassNotFoundException();
			}
			return defineClass(bytes, 0, bytes.length);
		}
		
		private byte[] getClassData(File file) {
			try (InputStream ins = new FileInputStream(file); 
					ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
				byte[] buffer = new byte[4096];
				int bytesNumRead = 0;
				while ((bytesNumRead = ins.read(buffer)) != -1) {
					baos.write(buffer, 0, bytesNumRead);
				}
				return baos.toByteArray();
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			} catch (IOException e) {
				e.printStackTrace();
			}
			return new byte[]{};
		}
```
- 最终结果与预期的结果不一致
	- 预期结果：业务A的MessageServiceImpl与MessageUtil由CustomerFindClass加载
	- 实际结果：业务A的MessageServiceImpl由CustomerFindClass加载，而MessageUtil由sun.misc.AppClassLoader加载。
	- 分析：由于JVM类加载的双亲委托机制，业务A调用消息工具类时，类加载器（CustomerFindClass）会委托父类加载器（AppClassLoader）加载类，如果存在，则不再执行自身的findClass方法加载，导致结果不理想。（main 方法类默认情况下都是由 JDK 自带的 AppClassLoader 加载的）。
#### 4.2 重写loadClass
```c
	private ClassLoader classLoader;
	
	/**
	* 重新loadClass方法
	*/
	@Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        Class result = null;
        try {
            //这里要使用 JDK 的类加载器加载 java.lang 包里面的类
            result = classLoader.loadClass(name);
        } catch (Exception e) {
            // ignore error
        }
        if (null != result) {
            return result;
        }
        String classPath = classPathMap.get(name);
        File file = new File(classPath);
        if (!file.exists()) {
            throw new ClassNotFoundException();
        }
        byte[] bytes = getClassData(file);
        if (null == bytes || 0 == bytes.length) {
            throw new ClassNotFoundException();
        }
        return defineClass(bytes, 0, bytes.length);
    }
```
- 满足业务A的MessageServiceImpl与MessageUtil由CustomerFindClass加载
	- 注意：这种方式破坏了双亲委托机制，但由于重写了loadClass方法，所有类均会有CustomerFindClass加载器加载，需要过滤出不需要隔离的类，如java.lang包下的类，需要由ExtClassLoader 来加载。

### 5、总结
- 本文分享的方式是从类加载器方向出发，实现最终的类隔离，避免了不同模块间不同类的冲突，其中顺便也简单带过了jvm类加载相关的知识点，也算是一劳多得，后续会结合实际使用场景进一步分析。
- 当然也有很多其它方式可以实现，有时间、有想法的朋友可以留言探讨交流。

