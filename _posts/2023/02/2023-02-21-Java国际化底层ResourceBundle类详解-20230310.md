---
layout: post
title:  Java国际化底层ResourceBundle类详解-已发
tagline: by IT可乐
categories: Java
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
做项目应该都会实现国际化，那么大家知道Java底层是如何实现国际化的吗？
<!--more-->

在Java开发中，ResourceBundle是一种方便地管理本地化资源的机制。它可以使得程序能够根据当前系统环境的语言和国家/地区来自动加载相应的本地化资源文件，从而避免了硬编码和减少了重复的代码。以下是使用ResourceBundle的基本步骤：

### 1. 准备资源文件

ResourceBundle通过加载资源文件来实现本地化，因此需要为每种语言和国家/地区准备一个对应的资源文件。资源文件可以是.properties格式的文本文件，也可以是.class文件或.jar文件。

在资源文件中，需要为每个需要本地化的字符串指定一个属性名，然后为每个属性名分别提供该语言下的翻译。例如，以下是一个名为messages.properties的资源文件的示例：

```properties
greeting=Hello
farewell=Goodbye
```

在不同的语言和国家/地区下，可以为同一属性名提供不同的翻译。例如，以下是名为messages_fr.properties的法语资源文件的示例：

```properties
greeting=Bonjour
farewell=Au revoir
```



### 2. 加载资源文件

在Java中，可以使用ResourceBundle类来加载资源文件。ResourceBundle类提供了几种不同的构造函数来加载资源文件，例如：

```java
ResourceBundle rb = ResourceBundle.getBundle("messages", Locale.getDefault());
```

这个语句会根据当前系统环境的默认语言和国家/地区来加载名为messages的资源文件。如果系统环境是英语和美国，那么这个语句会加载messages.properties资源文件。如果系统环境是法语和法国，那么这个语句会加载messages_fr.properties资源文件。

如果需要加载指定语言和国家/地区下的资源文件，可以使用带有Locale参数的getBundle()方法。例如：

```java
Locale locale = new Locale("fr", "FR");
ResourceBundle rb = ResourceBundle.getBundle("messages", locale);
```

这个语句会加载名为messages_fr_FR.properties的法语/法国资源文件。



### 3. 获取本地化字符串

一旦成功加载了资源文件，就可以使用ResourceBundle的getString()方法来获取本地化字符串。例如：

```java
String greeting = rb.getString("greeting");
String farewell = rb.getString("farewell");
```

这些语句会从资源文件中获取属性名为greeting和farewell的本地化字符串，并将它们分别赋值给greeting和farewell变量。如果无法找到指定的属性名，getString()方法会抛出MissingResourceException异常。



### 4. ResourceBundle 使用技巧

除了上述基本步骤，使用ResourceBundle还有以下一些值得注意的特点和技巧：

#### 4.1 选择合适的资源文件格式

ResourceBundle支持多种资源文件格式，包括.properties、.xml和.class文件等。对于简单的本地化字符串，.properties格式通常是最常用的选择，因为它简单易用、易于编辑和本地化。

对于较复杂的本地化资源，如图像、声音、视频等，可能需要使用其他格式的资源文件。例如，可以使用.class文件或.jar文件来包含图像或声音文件，并使用ResourceBundle的ClassLoader.getSystemClassLoader()方法来加载这些文件。



#### 4.2 处理本地化字符串中的特殊字符

在本地化字符串中可能包含各种特殊字符，如换行符、制表符、Unicode字符等。如果直接将这些字符嵌入到资源文件中，可能会导致不必要的麻烦和错误。

为了避免这些问题，可以使用Java的转义字符来表示这些特殊字符。例如，可以使用"\n"表示换行符，"\t"表示制表符，"\uXXXX"表示Unicode字符等。



#### 4.3 处理缺失的本地化字符串

在某些情况下，可能存在某些语言下的本地化字符串没有提供翻译的情况。为了避免程序出现MissingResourceException异常，可以在资源文件中为这些缺失的字符串提供一个默认的翻译，如英语翻译。例如，以下是一个带有默认翻译的messages_fr.properties文件的示例：

```java
greeting=Bonjour
farewell=Au revoir
warning=Attention: This message has no translation in French. Please refer to the English version.
```

这样，在法语环境下，如果无法找到某个属性名的本地化字符串，ResourceBundle就会自动返回该属性名的默认翻译，从而避免了程序出现异常。



#### 4.4 处理动态本地化字符串

有些本地化字符串可能包含动态内容，如时间、日期、数字、货币等。为了正确地本地化这些字符串，需要使用Java的格式化机制，如MessageFormat和NumberFormat等。例如，以下是一个使用MessageFormat来本地化动态字符串的示例：

```java
String pattern = rb.getString("greeting");
Object[] arguments = {"John"};
String greeting = MessageFormat.format(pattern, arguments);
```

这个示例中，pattern是一个包含占位符"{0}"的本地化字符串，"{0}"表示需要替换为动态内容的位置。arguments是一个包含实际动态内容的数组，它会按照顺序依次替换"{0}"的位置。最后，MessageFormat.format()方法会返回一个本地化后的字符串。



#### 4.5 处理多个资源文件

在一些情况下，可能需要使用多个资源文件来管理不同类型或不同用途的本地化资源。在这种情况下，可以使用ResourceBundle.Control类的方法来指定资源文件的搜索路径和加载顺序。

例如，可以使用ResourceBundle.Control.getControl()方法来获取默认的ResourceBundle.Control实例，然后使用ResourceBundle.getBundle()方法来指定基础名称和Locale信息，以便查找合适的资源文件。例如，以下是一个使用多个资源文件来管理本地化字符串的示例：

```java
ResourceBundle.Control control = ResourceBundle.Control.getControl(ResourceBundle.Control.FORMAT_PROPERTIES);
ResourceBundle messages = ResourceBundle.getBundle("Messages", new Locale("fr"), control);
ResourceBundle errors = ResourceBundle.getBundle("Errors", new Locale("fr"), control);

String greeting = messages.getString("greeting");
String error = errors.getString("invalid_input");

System.out.println(greeting); // Bonjour
System.out.println(error); // Entrée invalide
```

在这个示例中，我们使用ResourceBundle.Control.FORMAT_PROPERTIES指定了资源文件的格式为.properties文件，然后分别使用Messages和Errors作为基础名称来获取不同类型的资源文件。这样，我们就可以轻松地管理不同类型的本地化资源，从而使程序更加可读和易于维护。

#### 4.6 自定义资源加载器

如果默认的资源加载机制无法满足需求，我们还可以自定义资源加载器来实现更高级的功能。自定义资源加载器需要继承java.util.ResourceBundle.Control类，并重写其中的方法来实现自定义逻辑。

例如，以下是一个使用自定义资源加载器来加载本地化字符串的示例：

```java
public class MyResourceLoader extends ResourceBundle.Control {
    @Override
    public ResourceBundle newBundle(String baseName, Locale locale, String format, ClassLoader loader, boolean reload)
            throws IllegalAccessException, InstantiationException, IOException {
        String bundleName = toBundleName(baseName, locale);
        String resourceName = toResourceName(bundleName, "myproperties");
        InputStream stream = loader.getResourceAsStream(resourceName);
        if (stream != null) {
            try {
                return new PropertyResourceBundle(stream);
            } finally {
                stream.close();
            }
        } else {
            return super.newBundle(baseName, locale, format, loader, reload);
        }
    }
}

ResourceBundle.Control control = new MyResourceLoader();
ResourceBundle messages = ResourceBundle.getBundle("Messages", new Locale("fr"), control);

String greeting = messages.getString("greeting");

System.out.println(greeting); // Bonjour
```

在这个示例中，我们定义了一个名为MyResourceLoader的自定义资源加载器，并重写了其中的newBundle()方法来实现自定义资源加载逻辑。然后，我们使用这个自定义资源加载器来获取Messages资源文件中的本地化字符串。这样，我们就可以实现更高级的资源加载功能，从而满足更复杂的需求。



#### 4.7 动态更新资源文件

有时候，在应用程序运行期间，可能需要动态地更新资源文件中的某些值。在Java中，我们可以使用PropertyResourceBundle类来实现这个功能。

PropertyResourceBundle是ResourceBundle的一个子类，它可以读取.properties格式的资源文件，并将其转换为一个键值对的形式。然后，我们可以通过这个键值对来动态地更新资源文件中的值。

例如，以下是一个使用PropertyResourceBundle来动态更新本地化字符串的示例：

```java
// 加载资源文件
InputStream stream = new FileInputStream("Messages.properties");
PropertyResourceBundle bundle = new PropertyResourceBundle(stream);

// 动态更新本地化字符串
bundle.handleKey("greeting", (key, value) -> "Hello");

// 输出本地化字符串
String greeting = bundle.getString("greeting");
System.out.println(greeting); // Hello
```

在这个示例中，我们首先使用FileInputStream来加载Messages.properties资源文件，然后将其转换为一个PropertyResourceBundle对象。然后，我们使用handleKey()方法来动态地更新greeting这个键对应的值。最后，我们使用getString()方法来获取更新后的本地化字符串。

这种动态更新资源文件的方式可以使应用程序更加灵活，能够快速响应变化。但是需要注意的是，这种方式需要保证资源文件的正确性和一致性，否则可能会导致应用程序运行出错。

### 5. 总结

Java中的ResourceBundle提供了一种便捷的方式来管理本地化资源，使得应用程序能够轻松地适应不同的语言和文化环境。通过熟练掌握ResourceBundle的使用方法，我们可以在开发Java应用程序时更加灵活和高效。



