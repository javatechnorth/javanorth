---
title: 新瓶装旧酒：在SpringBoot中使用Axis1.4 20230516已发
date: 2023-05-16 08:15:00
author: gotanks广楠
categories: gotanks广楠
tagline: by 广楠
tags: ["SpringBoot", "Axis1.4", "gotanks广楠"]
---

> 只看封面图，一股浓浓的怀旧风迎面扑来......

哈喽，大家好，我是了不起。  

最近在做一个老旧工程的迁移，其中一个服务端接口是使用 WebService 做的，而且用的是 Axis1.4 这种老的不能再老的框架。经过一番探索和研究，终于成功迁移到Springboot工程里，下面跟着我一步步操作。


<!--more-->

## Axis配置文件

Axis1.4 的入口配置文件是 `server-config.wsdd`，在 resources 中创建一个 WEB-INF 文件夹，将配置文件拷贝到其中。

![](https://files.mdnice.com/user/34902/34970951-a68e-4258-8dc8-f7d3c5ebe51d.png)

我们看一下配置文件的内容：
```
<?xml version="1.0" encoding="UTF-8"?>
<deployment xmlns="http://xml.apache.org/axis/wsdd/"
            xmlns:java="http://xml.apache.org/axis/wsdd/providers/java">

    <handler name="URLMapper" type="java:org.apache.axis.handlers.http.URLMapper"/>
    <handler name="LocalResponder" type="java:org.apache.axis.transport.local.LocalResponder"/>

    <transport name="http">
        <requestFlow>
            <handler type="URLMapper"/>
        </requestFlow>
    </transport>

    <transport name="local">
        <responseFlow>
            <handler type="LocalResponder"/>
        </responseFlow>
    </transport>

    <!-- 示例接口 -->
    <service name="DemoService" provider="java:RPC">
        <parameter name="className" value="com.test.webservice.DemoService"/>
        <parameter name="allowedMethods" value="*"/>
    </service>

</deployment>
```

主要看最后一段：声明了一个示例接口：服务名为`DemoService`，映射的类为`com.test.webservice.DemoService`，方法名为`*`(通配符代表与类中的方法名一样)

## 接口类

简简单单的一个接口类：

```java
package com.test.webservice;

import org.springframework.stereotype.Component;

@Component
public class DemoService {

    public String test() {
        return "这是一个Axis1.4接口。";
    }

}

```

我们如何把这个接口发布出去呢，接着往下看。

## 引入依赖

我这里用的是 gradle 构建的：
```
implementation('org.apache.axis:axis:1.4')
implementation('org.apache.axis:axis-jaxrpc:1.4')
implementation('axis:axis-wsdl4j:1.5.1')
implementation('commons-discovery:commons-discovery:0.5')
```


## 继承AxisServlet

创建一个Servlet类，继承AxisServlet：

```java
package com.test.webservice.servlet;

import org.apache.axis.transport.http.AxisServlet;
import javax.servlet.annotation.WebServlet;

@WebServlet(
        urlPatterns = "/axis-services/*",
        loadOnStartup = 1,
        name = "AxisServlet"
)
public class MyAxisServlet extends AxisServlet {
}
```

注意，Axis默认的过滤 url 是`/services/`，而我这里将 **urlPatterns** 配置为`/axis-services/`，是因为工程中同时使用了 **Cxf框架** ，如果使用默认配置，会与 Cxf 冲突。

### 重写Axis配置工厂

网上拷贝的配置工厂类：

```java
/*
 * Copyright 2002-2004 The Apache Software Foundation.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.axis.configuration;

import org.apache.axis.AxisProperties;
import org.apache.axis.ConfigurationException;
import org.apache.axis.EngineConfiguration;
import org.apache.axis.EngineConfigurationFactory;
import org.apache.axis.components.logger.LogFactory;
import org.apache.axis.server.AxisServer;
import org.apache.axis.utils.ClassUtils;
import org.apache.axis.utils.Messages;
import org.apache.commons.logging.Log;

import javax.servlet.ServletConfig;
import java.io.InputStream;

/**
 * This is a default implementation of ServletEngineConfigurationFactory.
 * It is user-overrideable by a system property without affecting
 * the caller. If you decide to override it, use delegation if
 * you want to inherit the behaviour of this class as using
 * class extension will result in tight loops. That is, your
 * class should implement EngineConfigurationFactory and keep
 * an instance of this class in a member field and delegate
 * methods to that instance when the default behaviour is
 * required.
 *
 * @author Richard A. Sitze
 * @author Davanum Srinivas (dims@apache.org)
 */
public class EngineConfigurationFactoryServlet
        extends EngineConfigurationFactoryDefault {
    protected static Log log =
            LogFactory.getLog(EngineConfigurationFactoryServlet.class.getName());

    private ServletConfig cfg;

    /**
     * Creates and returns a new EngineConfigurationFactory.
     * If a factory cannot be created, return 'null'.
     * <p>
     * The factory may return non-NULL only if:
     * - it knows what to do with the param (param instanceof ServletContext)
     * - it can find it's configuration information
     *
     * @see EngineConfigurationFactoryFinder
     */
    public static EngineConfigurationFactory newFactory(Object param) {
        /**
         * Default, let this one go through if we find a ServletContext.
         *
         * The REAL reason we are not trying to make any
         * decision here is because it's impossible
         * (without refactoring FileProvider) to determine
         * if a *.wsdd file is present or not until the configuration
         * is bound to an engine.
         *
         * FileProvider/EngineConfiguration pretend to be independent,
         * but they are tightly bound to an engine instance...
         */
        return (param instanceof ServletConfig)
                ? new EngineConfigurationFactoryServlet((ServletConfig) param)
                : null;
    }

    /**
     * Create the default engine configuration and detect whether the user
     * has overridden this with their own.
     */
    protected EngineConfigurationFactoryServlet(ServletConfig conf) {
        super();
        this.cfg = conf;
    }

    /**
     * Get a default server engine configuration.
     *
     * @return a server EngineConfiguration
     */
    @Override
    public EngineConfiguration getServerEngineConfig() {
        return getServerEngineConfig(cfg);
    }

    /**
     * Get a default server engine configuration in a servlet environment.
     *
     * @param cfg a ServletContext
     * @return a server EngineConfiguration
     */
    private static EngineConfiguration getServerEngineConfig(ServletConfig cfg) {

        // Respect the system property setting for a different config file
        String configFile = cfg.getInitParameter(OPTION_SERVER_CONFIG_FILE);
        if (configFile == null) {
            configFile =
                    AxisProperties.getProperty(OPTION_SERVER_CONFIG_FILE);
        }
        if (configFile == null) {
            configFile = SERVER_CONFIG_FILE;
        }

        /**
         * Flow can be confusing.  Here is the logic:
         * 1) Make all attempts to open resource IF it exists
         *    - If it exists as a file, open as file (r/w)
         *    - If not a file, it may still be accessable as a stream (r)
         *    (env will handle security checks).
         * 2) If it doesn't exist, allow it to be opened/created
         *
         * Now, the way this is done below is:
         * a) If the file does NOT exist, attempt to open as a stream (r)
         * b) Open named file (opens existing file, creates if not avail).
         */

        /*
         * Use the WEB-INF directory
         * (so the config files can't get snooped by a browser)
         */
        String appWebInfPath = "/WEB-INF";
        //由于部署方式变更为jar部署，此处不可以使用改方式获取路径
//        ServletContext ctx = cfg.getServletContext();
//        String realWebInfPath = ctx.getRealPath(appWebInfPath);

        FileProvider config = null;
        String realWebInfPath = EngineConfigurationFactoryServlet.class.getResource(appWebInfPath).getPath();

        /**
         * If path/file doesn't exist, it may still be accessible
         * as a resource-stream (i.e. it may be packaged in a JAR
         * or WAR file).
         */
        InputStream iss = ClassUtils.getResourceAsStream(EngineConfigurationFactoryServlet.class, appWebInfPath + "/" + SERVER_CONFIG_FILE);
        if (iss != null) {
            // FileProvider assumes responsibility for 'is':
            // do NOT call is.close().
            config = new FileProvider(iss);
        }

        if (config == null) {
            log.error(Messages.getMessage("servletEngineWebInfError03", ""));
        }

        /**
         * Couldn't get data  OR  file does exist.
         * If we have a path, then attempt to either open
         * the existing file, or create an (empty) file.
         */
        if (config == null && realWebInfPath != null) {
            try {
                config = new FileProvider(realWebInfPath, configFile);
            } catch (ConfigurationException e) {
                log.error(Messages.getMessage("servletEngineWebInfError00"), e);
            }
        }

        /**
         * Fall back to config file packaged with AxisEngine
         */
        if (config == null) {
            log.warn(Messages.getMessage("servletEngineWebInfWarn00"));
            try {
                InputStream is =
                        ClassUtils.getResourceAsStream(AxisServer.class,
                                SERVER_CONFIG_FILE);
                config = new FileProvider(is);

            } catch (Exception e) {
                log.error(Messages.getMessage("servletEngineWebInfError02"), e);
            }
        }

        return config;
    }
}
```

有两点需要注意：

1. 这个配置类必须放在`org.apache.axis.configuration`包下，否则不生效，别问为什么，我也不知道...

2. 代码中135行左右，`String appWebInfPath = "/WEB-INF";`指向的文件夹就是一开始创建的 `WEB-INF`



## 最后一步

在启动类加上注解 `@ServletComponentScan`

```java
@SpringBootApplication
@ServletComponentScan
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

启动程序，访问 `http://localhost:8080/axis-services`，会显示已发布的 WebService 接口信息，示例接口地址为：`http://localhost:8080/axis-services/DemoService?wsdl`。

大功告成！
