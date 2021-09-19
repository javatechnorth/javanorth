---
layout: post
title:  Mybatis 面试题
tagline: by IT可乐
categories: Mybatis 面试题
tags: 
    - IT可乐
---

Mybatis 面试题
<!--more-->

### 1、什么是Mybatis？
MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。  
iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

（1）Mybatis是一个半ORM（对象关系映射）框架，它内部封装了JDBC，加载驱动、创建连接、创建statement等繁杂的过程，开发者开发时只需要关注如何编写SQL语句，可以严格控制sql执行性能，灵活度高。

（2）作为一个半ORM框架，MyBatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO映射成数据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。

称Mybatis是半自动ORM映射工具，是因为在查询关联对象或关联集合对象时，需要手动编写sql来完成。不像Hibernate这种全自动ORM映射工具，Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。

（3）通过xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过java对象和 statement中sql的动态参数进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射为java对象并返回。（从执行sql到返回result的过程）。

（4）由于MyBatis专注于SQL本身，灵活度高，所以比较适合对性能的要求很高，或者需求变化较多的项目，如互联网项目。

### 2、为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？
Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。

而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。

Mybatis直接编写原生态sql，可以严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是灵活的前提是mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套sql映射文件，工作量大。 

Hibernate对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件，如果用hibernate开发可以节省很多代码，提高效率。 

### 3、Mybaits的优缺点
（1）优点：

① 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除sql与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用。

② 与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接；

③ 很好的与各种数据库兼容（因为MyBatis使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）。

④ 能够与Spring很好的集成；

⑤ 提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护。

（2）缺点：

① SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求。

② SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

### 4、不使用 Mybatis，使用JDBC如何操作数据库？
①、加载数据库驱动  

②、获取数据库连接  

③、定义 sql 语句  

④、获取预编译处理的statement  

⑤、向数据库发出 sql 语句，并返回结果  

⑥、解析结果  

⑦、关闭数据库连接
```java
package com.itcoke.jdbc;

import com.itcoke.bean.Person;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class JDBCUtils {
    //MySQL数据库驱动,注意多了一个cj，com.mysql.jdbc.Driver和mysql-connector-java 5一起用
    public static String driverClass = "com.mysql.cj.jdbc.Driver";
    //MySQL用户名
    public static String userName = "root";
    //MySQL密码
    public static String passWord = "root1234";
    //MySQL URL
    public static String url = "jdbc:mysql://localhost:3306/mybatis-study";
    //定义数据库连接
    public static Connection conn = null;
    //定义声明数据库语句,使用 预编译声明 PreparedStatement提高数据库执行性能
    public static PreparedStatement ps = null;
    //定义返回结果集
    public static ResultSet rs = null;

    public static List<Person> selectPersonByName(String pname){
        List<Person> personList = new ArrayList<>();

        try {
            // 1、加载数据库驱动
            Class.forName(driverClass);
            // 2、获取数据库连接
            conn = DriverManager.getConnection(url,userName,passWord);
            // 3、定义 sql 语句,?表示占位符
            String sql = "select * from person where pname=?";
            // 4、获取预编译处理的statement
            ps = conn.prepareStatement(sql);
            //设置sql语句中的参数，第一个为sql语句中的参数的?(从1开始)，第二个为设置的参数值
            ps.setString(1, "itcoke");
            // 5、向数据库发出 sql 语句查询，并返回结果集
            rs = ps.executeQuery();
            while(rs.next()){
                // 6、读取ResultSet，转换成我们要的对象
                Person person = new Person();
                person.setPid(rs.getLong("pid"));
                person.setPname(rs.getString("pname"));
                personList.add(person);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            // 7、关闭数据库连接
            if(rs!=null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(ps!=null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        return personList;
    }

    public static void main(String[] args){
        List<Person> personList = JDBCUtils.selectPersonByName("itcoke");
        System.out.println(personList);
    }
}

```

### 5、使用 JDBC 的缺点
①、问题一：每执行一次增删改查，就要建立连接，关闭连接，频繁的获取连接和关闭连接，会造成数据库资源浪费，影响数据库性能。

②、问题二：将 sql 语句硬编码到程序中，如果sql语句修改了，那么需要重新编译 Java 代码，不利于系统维护

③、问题三：在 PreparedStatement 中设置参数，对占位符设置值都是硬编码在Java代码中，不利于系统维护

④、问题四：从 resultset 中遍历结果集时，对表的字段存在硬编码，不利于系统维护

⑤、问题五：重复性代码特别多，包括建立建立，加载驱动等

⑥、问题六：没有缓存，如果存在数据量大，且频繁查询的情况，这种方式性能特别低

⑦、问题七：sql 的移植性不好，如果换个数据库，那么sql 语句可能要重写

上面这些问题，Mybatis 都能解决。

### 6、MyBatis 与 Hibernate 有哪些不同？
①、Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己编写 Sql 语句，不过 mybatis 可以通过 XML 或注解方式灵活配置要运行的 sql 语句，并将java 对象和 sql 语句映射生成最终执行的 sql，最后将 sql 执行的结果再映射生成 java 对
象。  

②、Mybatis 学习门槛低，简单易学，程序员直接编写原生态 sql，可严格控制 sql 执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的前提是 mybatis 无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定义多套 sql 映射文件，工作量大。  

③、Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如需求固定的定制化软件）如果用 hibernate 开发可以节省很多代码，提高效率。但是Hibernate 的缺点是学习门槛高，要精通门槛更高，而且怎么设计 O/R 映射，在性能和对象模型之间如何权衡，以及怎样用好 Hibernate 需要具有很强的经验和能力才行。  

总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都是好架构，所以框架只有适合才是最好

### 7、Mybatis 编程步骤
①、根据配置文件创建 SqlSessionFactory 工厂  

②、通过 SqlSessionFactory 工厂生产 SqlSession 对象  

③、通过 SqlSession 对象进行 CRUD 操作  

④、调用 SqlSession.commit() 提交事务  

⑤、调用 SqlSession.close() 关闭会话  


### 8、#{}和${}的区别是什么？
> "${}"是Properties文件中的变量占位符，它可以用于标签属性值和sql内部，属于静态文本替换，比如${driver}会被静态替换为com.mysql.jdbc.Driver。

> "#{}"是sql的参数占位符，Mybatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0, parameterValue)，#{item.name}的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。  

使用#{}可以有效的防止SQL注入，提高系统安全性。  
一般能用#的就别用$。  
$方式一般用于传入数据库对象，例如传入表名.

### 9、Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？
不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；

原因就是namespace+id是作为Map的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

备注：在旧版本的Mybatis中，namespace是可选的，不过新版本的namespace已经是必须的了。

### 10、简述 Mybatis 的 Xml 映射文件和 Mybatis 内部数据结构之间的映射关系？  
Mybatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在Xml 映射文件中， <parameterMap>标签会被解析为 ParameterMap 对象，  
其每个子元素会被解析为 ParameterMapping 对象。 <resultMap>标签会被解析为 ResultMap 对象，其每个子元素会被解析为 ResultMapping 对象。  
每一个<select>、 <insert>、 <update>、 <delete>标签均会被解析为 MappedStatement 对象，标签内的 sql 会被解析为 BoundSql 对象。

### 11、什么是 MyBatis 的接口绑定,有什么好处？
接口映射就是在 MyBatis 中任意定义接口,然后把接口里面的方法和 SQL 语句绑定,我们直接调用接口方法就可以,这样比起原来了 SqlSession 提供的方法我们可以有更加灵活的选择和设置

### 12、接口绑定有几种实现方式,分别是怎么实现的?
接口绑定有两种实现方式,一种是通过注解绑定,就是在接口的方法上面加上@Select@Update 等注解里面包含 Sql 语句来绑定,另外一种就是通过 xml 里面写 SQL 来绑定,在这种情况下,要指定 xml 映射文件里面的 namespace 必须为接口的全路径名


### 13、Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？
Mybatis动态sql可以让我们在Xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能，Mybatis提供了9种动态sql标签trim|where|set|foreach|if|choose|when|otherwise|bind。

其执行原理为，使用OGNL从sql参数对象中计算表达式的值，根据表达式的值动态拼接sql，以此来完成动态sql的功能。

### 14、Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？
第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。  

第二种是使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。

有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。


### 15、当实体类中的属性名和表中的字段名不一样 ，怎么办？
第1种： 通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。  
```xml
<select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”> 
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id}; 
</select> 
```

第2种： 通过<resultMap>来映射字段名和实体类属性名的一一对应的关系
```xml
<select id="getOrder" parameterType="int" resultMap="orderresultmap">
        select * from orders where order_id=#{id}
    </select>
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”> 
        <!–用id属性来映射主键字段–> 
        <id property=”id” column=”order_id”> 
        <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–> 
        <result property = “orderno” column =”order_no”/> 
        <result property=”price” column=”order_price” /> 
    </reslutMap>
```

### 16、Mybatis是如何进行分页的？分页插件的原理是什么？
①、Mybatis 使用 RowBounds 对象进行分页，也可以直接编写 sql 实现分页，也可以使用Mybatis 的分页插件。  

②、分页插件的原理：实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql。  

举例： select * from student，拦截 sql 后重写为： select t.* from （ select * from student） t limit 0， 10


### 17、简述 Mybatis 的插件运行原理，以及如何编写一个插件？
①、Mybatis 仅可以编写针对 ParameterHandler、 ResultSetHandler、 StatementHandler、Executor 这 4 种接口的插件， Mybatis 通过动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是
InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。  

②、实现 Mybatis 的 Interceptor 接口并复写 intercept()方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。


### 18、使用 MyBatis 的 mapper 接口调用时有哪些要求？
①、Mapper 接口方法名和 mapper.xml 中定义的每个 sql 的 id 相同  

②、Mapper 接口方法的输入参数类型和 mapper.xml 中定义的每个 sql 的 parameterType 的类型相同  

③、Mapper 接口方法的输出参数类型和 mapper.xml 中定义的每个 sql 的 resultType 的类型相同  

④、Mapper.xml 文件中的 namespace 即是 mapper 接口的类路径。



















