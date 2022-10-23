---
layout: post
title:  Maven删除重复的依赖关系
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在我们平时的开发过程中，常常会遇到引入各种不同的 jar 包，然后引发的 `Maven` 依赖冲突，今天我们来学习下如何使用 Maven 命令检测 `pom.xml` 中的重复依赖项。

<!--more-->

### 为什么要检测重复的依赖关系

在`pom.xml`中, 经常引入各种不同的jar 包， 又会依赖其他的jar。特别是一些常用的工具库，比较容易出现版本冲突，例如，让我们考虑下面的`pom.xml`。

```xml
<project>
  [...]
  <dependencies>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.12.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.11</version>
    </dependency>
  </dependencies>
   [...]
</project>
```

从上面的代码中，`commons-lang3` 被引用了两次，而且版本号也不一样。 现在我们就来看看如何使用Maven命令来检测这些重复的依赖关系。

### 依赖树命令

让我们在终端运行 `mvn dependency:tree`的命令，看看输出结果

```shell
$ mvn dependency:tree
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.javanorth:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 15
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -------------< com.javanorth:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ maven-duplicate-dependencies ---
[WARNING] The artifact xml-apis:xml-apis:jar:2.0.2 has been relocated to xml-apis:xml-apis:jar:1.0.b2
[INFO] com.javanorth:maven-duplicate-dependencies:jar:0.0.1-SNAPSHOT
[INFO] \- org.apache.commons:commons-lang3:jar:3.11:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.136 s
...
```

我们可以看到，`commons-lang3` jar的3.11版和 3.12 版同时被引入进来了，出现这种情况是因为Maven选择了`pom.xml`中后来出现的依赖。

### 依赖关系`analyze-duplicate`命令

现在让我们运行 `mvn dependency:analyze-duplicate`，看看输出输出结果。

```shell
$ mvn dependency:analyze-duplicate
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.javanorth:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 15
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -------------< com.javanorth:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:analyze-duplicate (default-cli) @ maven-duplicate-dependencies ---
[WARNING] The artifact xml-apis:xml-apis:jar:2.0.2 has been relocated to xml-apis:xml-apis:jar:1.0.b2
[INFO] List of duplicate dependencies defined in <dependencies/> in your pom.xml:
        o org.apache.commons:commons-lang3:jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.835 s
。。。
```

我们注意到，`WARNING`和`INFO`日志语句都提到了重复依赖的存在。

### 如果存在重复的依赖，则构建失败

在上面的例子中，我们看到了如何检测重复的依赖关系，但`BUILD`仍然是成功的，单这可能导致使用了不正确的 jar 版本。

使用[Maven Enforcer Plugin]（https://maven.apache.org/enforcer/maven-enforcer-plugin/index.html），我们可以确保在存在重复依赖的情况下构建不成功。

我们需要在`pom.xml`中加入这个Maven插件，并加入`banDuplicatePomDependencyVersions`规则。

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>3.0.0</version>
        <executions>
          <execution>
            <id>no-duplicate-declared-dependencies</id>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <banDuplicatePomDependencyVersions/>
              </rules>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```

现在，该规则约束了我们的Maven构建。

```shell
$ mvn verify
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.javanorth:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 14
[WARNING]
[INFO] -------------< com.javanorth:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-enforcer-plugin:3.0.0:enforce (no-duplicate-declared-dependencies) @ maven-duplicate-dependencies ---
[WARNING] Rule 0: org.apache.maven.plugins.enforcer.BanDuplicatePomDependencyVersions failed with message:
Found 1 duplicate dependency declaration in this project:
 - dependencies.dependency[org.apache.commons:commons-lang3:jar] ( 2 times )

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-enforcer-plugin:3.0.0:enforce (no-duplicate-declared-dependencies) on project maven-duplicate-dependencie
s: Some Enforcer rules have failed. Look above for specific messages explaining why the rule failed.
```

### 删除重复的依赖关系

只要确定了重复的依赖关系，我们就需要在 `pom.xml`中删除它们，只保留那些我们项目使用的唯一依赖关系。

### 总结

本文中，我们学习了如何使用`mvn dependency:tree`和`mvn dependency:analyze-duplicate`命令检测Maven中的重复依赖，还学习了如何使用Maven Enforcer插件，通过应用内置规则使包含重复依赖的构建失败。
