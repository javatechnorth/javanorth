## 一文搞懂BeanFactory 和 FactoryBean



在Spring框架中，BeanFactory和FactoryBean是两个关键的概念，它们都与创建和管理Bean有关，但它们在功能和作用上有很大的区别。

以下是关于它们的详细解释，以及它们之间的区别。

### BeanFactory

BeanFactory是Spring框架的核心接口之一，它定义了Spring容器的基本行为，负责管理Bean的生命周期、配置元数据和依赖注入。BeanFactory的主要功能包括：

1. **Bean的实例化和管理**：BeanFactory负责创建、初始化和管理Bean的生命周期。它会根据配置文件中定义的Bean定义来创建Bean的实例。
2. **依赖注入**：BeanFactory负责解决Bean之间的依赖关系，确保每个Bean都能获取它所依赖的其他Bean。
3. **配置元数据的管理**：BeanFactory会读取和管理应用程序的配置元数据，通常以XML、注解或Java配置的方式定义Bean及其属性。
4. **延迟初始化**：BeanFactory支持延迟初始化，即只有在需要时才创建Bean实例。
5. **AOP支持**：BeanFactory支持面向切面编程（AOP），允许在Bean的生命周期中应用切面。

BeanFactory是Spring IOC容器的基础，但它通常不会直接使用，而是通过其更高级的实现来使用，如ApplicationContext。

### BeanFactory的子类

1. **XmlBeanFactory**：XmlBeanFactory是Spring 2.5之前的BeanFactory实现，它通过解析XML配置文件来创建和管理Bean。它的作用是从XML文件中加载Bean定义并提供Bean实例化、依赖注入等基本功能。
2. **DefaultListableBeanFactory**：DefaultListableBeanFactory是BeanFactory接口的主要实现，它是Spring IoC容器的核心，负责管理Bean的生命周期、依赖注入、AOP支持等。它支持各种不同的Bean定义来源，包括XML、注解和Java配置。
3. **ApplicationContext**：ApplicationContext是BeanFactory的子类，它是更高级的Spring容器。它扩展了BeanFactory的功能，提供了更多的应用级功能，如国际化、事件传播、资源加载、应用上下文层次结构等。这个类是我们最熟悉的类，也是spring的核心。

### FactoryBean

FactoryBean是一个特殊的Bean，它是一个工厂类的接口，负责创建其他Bean的实例。FactoryBean的主要功能包括：

1. **自定义Bean的创建过程**：FactoryBean允许您自定义Bean的创建逻辑。您可以编写一个实现FactoryBean接口的类，重写`getObject`方法，以自定义Bean的创建逻辑。
2. **懒加载**：FactoryBean可以控制Bean的懒加载。如果您的FactoryBean返回一个代理对象，它可以推迟实际Bean的创建，直到被请求时。
3. **Bean的包装**：FactoryBean可以用于包装其他Bean。您可以在FactoryBean中创建一个Bean的代理，以便在Bean的生命周期中添加额外的行为。
4. **处理复杂逻辑**：FactoryBean常用于创建复杂的Bean实例，例如连接池、远程服务代理等。它们允许您在Bean的创建过程中执行复杂的逻辑。

### FactoryBean的子类

1. **ProxyFactoryBean**：ProxyFactoryBean是一个FactoryBean的实现，它用于创建代理对象。您可以配置ProxyFactoryBean来创建JDK动态代理或CGLIB代理，用于AOP切面。它的作用是在Bean的创建过程中创建代理，以实现切面逻辑。
2. **ListFactoryBean**：ListFactoryBean是FactoryBean的实现，它用于创建List类型的Bean。您可以配置ListFactoryBean来包含其他Bean的引用，然后以List的形式注入到其他Bean中。
3. **MapFactoryBean**：MapFactoryBean是FactoryBean的实现，它用于创建Map类型的Bean。您可以配置MapFactoryBean来包含键值对，然后以Map的形式注入到其他Bean中。
4. **ServiceLocatorFactoryBean**：ServiceLocatorFactoryBean是FactoryBean的实现，它用于实现服务定位模式。它的作用是在Spring中创建服务接口的代理，以便进行动态查找和调用服务。

### BeanFactory 和 FactoryBean区别

1. **用途**：
   - BeanFactory是Spring IoC容器的核心接口，负责管理Bean的生命周期和依赖注入。
   - FactoryBean是一个特殊的Bean，充当其他Bean的工厂，用于自定义Bean的创建过程。
2. **创建对象**：
   - BeanFactory负责创建Bean对象。
   - FactoryBean是一个Bean，它的实例本身是一个工厂，负责创建其他Bean的实例。
3. **自定义性**：
   - BeanFactory通常不需要自定义实现，而是由Spring框架提供的。
   - FactoryBean需要自定义实现，您需要编写一个类，实现FactoryBean接口，并重写`getObject`方法来定义Bean的创建逻辑。
4. **懒加载**：
   - BeanFactory默认支持懒加载，可以配置Bean的延迟初始化。
   - FactoryBean可以通过返回代理对象来实现懒加载，它控制何时创建实际的Bean实例。

### 小结

BeanFactory是Spring IoC容器的核心接口，负责管理Bean的生命周期和依赖注入，大多数的Bean对象，包括Spring中内置的Bean对象和应用程序自定义的Bean对象，都是由BeanFactory创建。

而FactoryBean是一个特殊的Bean，它充当其他Bean的工厂，用于自定义Bean的创建过程，支持懒加载、包装和代理，以及处理复杂的逻辑。

Bean可以由两种不同的方式创建：

1. **由BeanFactory创建**：大多数Bean是由Spring的BeanFactory或ApplicationContext容器直接创建的，这些Bean是普通的Java对象，不需要实现FactoryBean接口。当您在Spring配置中定义一个Bean时，通常是直接指定该Bean的类，并且Spring容器会根据类的信息来实例化和管理Bean的生命周期。这些Bean不需要实现FactoryBean接口。
2. **由FactoryBean创建**：有些特殊类型的Bean是由实现了FactoryBean接口的类创建的。FactoryBean是一种用于创建其他Bean的工厂，它允许您自定义Bean的创建过程。这些FactoryBean实现类实现了FactoryBean接口，重写了`getObject`方法，用于定义Bean的创建逻辑。通常，当您配置FactoryBean作为Bean时，您实际上配置的是FactoryBean的实例，而不是FactoryBean创建的Bean实例。

总结：不是所有的Bean都是由FactoryBean创建的。大多数普通的Bean由BeanFactory（或ApplicationContext）创建，而FactoryBean通常用于创建特殊类型的Bean，或者对Bean的创建过程进行自定义控制。如果您只需要普通Bean，不需要实现FactoryBean接口。