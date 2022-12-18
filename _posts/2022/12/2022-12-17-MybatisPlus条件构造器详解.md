---
layout: post
title:  一文彻底弄懂MybatisPlus复杂的条件构造器
tagline: by IT可乐
categories: mybatisplus
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
<!--more-->
上篇文章我们介绍过通过 Mybatis Plus 进行增删改查，如下这段代码：

```java
/**
 * 根据id修改
 * UPDATE user SET user_name=?, user_age=? WHERE (id = ?)
 */
@Test
public void testudpateById(){
    User user = new User();
    user.setUserAge("25");
    user.setUserName("test update");
    UpdateWrapper updateWrapper = new UpdateWrapper();
    updateWrapper.eq("id","3");
    int num = userMapper.update(user, updateWrapper);
    System.out.println("修改的记录数为："+num);
}
     
/**
 * 查询指定记录
 * SELECT id,user_name,user_age FROM user WHERE (user_name = ?)
 */
@Test
public void testSelectWrapper(){
    QueryWrapper wrapper = new QueryWrapper();
    wrapper.eq("user_name","IT可乐");
    List<User> users = userMapper.selectList(wrapper);
    users.forEach(x-> System.out.println(x.getId()+"-"+x.getUserName()+"-"+x.getUserAge()));
}
```

上面两个方法分别是根据id修改表记录，和根据user_name查询记录。构造的条件使用了 UpdateWrapper 和 QueryWrapper ，那么这是什么呢？其实 mybatis plus 通过条件构造器可以组成复杂的SQL语句。本篇博客我们将详细介绍。

### 1、Wrapper

Mybatis Plus 提供的几种条件构造器，关系如下：

![](http://www.javanorth.cn/assets/images/2022/itcoke/mybatisplus/mybatisplus-02-00.png)

我们主要通过 QueryWrapper 和 UpdateWrapper 进行条件构造，这两个和 LambdaQueryWrapper、LambdaUpdateWrapper 差不多是等价的，只不过后者采用了 JDK1.8 提供的lambda 语法，使用起来更简洁。



### 2、语法详情总结

关于条件构造器的各个用法介绍，可以参考官网：https://mp.baomidou.com/guide/wrapper.html#abstractwrapper

这里我们做一下总结：

| 方法名                                                  | 说明                                                       | 用法实例                                                   | 等价SQL                                             |
| ------------------------------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------- |
| 官网地址                                                | https://mp.baomidou.com/guide/wrapper.html#abstractwrapper | ----:                                                      | :----:                                              |
| allEq(Map<R, V> params)                                 | 全部等于                                                   | map.put("id","3");map.put("user_name","IT可乐");allEq(map) | user_name = "IT可乐" AND id = 3                     |
| eq(R column, Object val)                                | 等于 =                                                     | eq("id","3")                                               | id = 3                                              |
| ne(R column, Object val)                                | 不等于 <>                                                  | ne("id", "3")                                              | id <> 3                                             |
| gt(R column, Object val)                                | 大于 >                                                     | gt("user_age","18")                                        | user_age > 18                                       |
| ge(R column, Object val)                                | 大于等于 >=                                                | ge("user_age","18")                                        | user_age >= 18                                      |
| lt(R column, Object val)                                | 小于 <                                                     | lt("user_age","18")                                        | user_age < 18                                       |
| le(R column, Object val)                                | 小于等于 <=                                                | le("user_age","18")                                        | user_age <= 18                                      |
| between(R column, Object val1, Object val2)             | BETWEEN 值1 AND 值2                                        | between("user_age","18","25")                              | user_age BETWEEN 18 AND 25                          |
| notBetween(R column, Object val1, Object val2)          | NOT BETWEEN 值1 AND 值2                                    | notBetween("user_age","18","25")                           | user_age NOT BETWEEN 18 AND 25                      |
| like(R column, Object val)                              | LIKE '%值%'                                                | like("user_name","可乐")                                   | like ‘%可乐%’                                       |
| notLike(R column, Object val)                           | NOT LIKE '%值%'                                            | notLike("user_name","可乐")                                | not like ‘%可乐%’                                   |
| likeLeft(R column, Object val)                          | LIKE '%值'                                                 | likeLeft("user_name","可乐")                               | like ‘%可乐’                                        |
| likeRight(R column, Object val)                         | LIKE '值%'                                                 | likeRight("user_name","可乐")                              | like ‘可乐%’                                        |
| isNull(R column)                                        | 字段 IS NULL                                               | isNull("user_name")                                        | user_name IS NULL                                   |
| isNotNull(R column)                                     | 字段 IS NOT NULL                                           | isNotNull("user_name")                                     | user_name IS NOT NULL                               |
| in(R column, Collection<?> value)                       | 字段 IN (value.get(0), value.get(1), ...)                  | in("user_age",{1,2,3})                                     | user_age IN (?,?,?)                                 |
| notIn(R column, Collection<?> value)                    | 字段 NOT IN (value.get(0), value.get(1), ...)              | notIn("user_age",{1,2,3})                                  | user_age NOT IN (?,?,?)                             |
| inSql(R column, String inValue)                         | 字段 IN ( sql语句 )                                        | inSql("id","select id from user")                          | id IN (select id from user)                         |
| notInSql(R column, String inValue)                      | 字段 NOT IN ( sql语句 )                                    | notInSql("id","select id from user where id > 2")          | id NOT IN (select id from user where id > 2         |
| groupBy(R... columns)                                   | 分组：GROUP BY 字段, ...                                   | groupBy("id","user_age")                                   | GROUP BY id,user_age                                |
| orderByAsc(R... columns)                                | 排序：ORDER BY 字段, ... ASC                               | orderByAsc("id","user_age")                                | ORDER BY id ASC,user_age ASC                        |
| orderByDesc(R... columns)                               | 排序：ORDER BY 字段, ... DESC                              | orderByDesc("id","user_age")                               | ORDER BY id DESC,user_age DESC                      |
| orderBy(boolean condition, boolean isAsc, R... columns) | ORDER BY 字段, ...                                         | orderBy(true,true,"id","user_age")                         | ORDER BY id ASC,user_age ASC                        |
| having(String sqlHaving, Object... params)              | HAVING ( sql语句 )                                         | having("sum(user_age)>{0}","25")                           | HAVING sum(user_age)>25                             |
| or()                                                    | 拼接 OR                                                    | eq("id",1).or().eq("user_age",25)                          | id = 1 OR user_age = 25                             |
| and(Consumerconsumer)                                   | AND 嵌套                                                   | and(i->i.eq("id",1).ne("user_age",18))                     | id = 1 AND user_age <> 25                           |
| nested(Consumerconsumer)                                | 正常嵌套 不带 AND 或者 OR                                  | nested(i->i.eq("id",1).ne("user_age",18))                  | id = 1 AND user_age <> 25                           |
| apply(String applySql, Object... params)                | 拼接 sql(不会有SQL注入风险)                                | apply("user_age>{0}","25 or 1=1")                          | user_age >'25 or 1=1'                               |
| last(String lastSql)                                    | 拼接到 sql 的最后,多次调用以最后一次为准(有sql注入的风险)  | last("limit 1")                                            | limit 1                                             |
| exists(String existsSql)                                | 拼接 EXISTS ( sql语句 )                                    | exists("select id from user where user_age = 1")           | EXISTS (select id from user where user_age = 1)     |
| notExists(String notExistsSql)                          | 拼接 NOT EXISTS ( sql语句 )                                | notExists("select id from user where user_age = 1")        | NOT EXISTS (select id from user where user_age = 1) |

### 3、语法详情演示

对于上表出现的每个语法，这里通过代码展示出来。

更多可以参考地址：https://github.com/YSOcean/mybatisplusstudy.git

```java
package com.ys.mybatisplusstudy;
 
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.ys.mybatisplusstudy.entry.User;
import com.ys.mybatisplusstudy.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
 
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
 
@SpringBootTest
public class WrapperTest {
    @Autowired
    private UserMapper userMapper;
 
 
    /**
     * 新增一条记录
     */
    @Test
    public void testInsert(){
        User user = new User();
        user.setId(4L);
        user.setUserName("test insert");
        user.setUserAge("1");
        int insert = userMapper.insert(user);
        System.out.println("影响记录数："+insert);
    }
 
    /**
     * allEq 全部等于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name = ? AND id = ?)
     */
    @Test
    public void testAllEq(){
        QueryWrapper queryWrapper = new QueryWrapper();
        Map map = new HashMap<>();
        map.put("id","3");
        map.put("user_name","IT可乐");
        queryWrapper.allEq(map);
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * eq 等于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (id = ?)
     */
    @Test
    public void testEq(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.eq("id","3");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * ne 不等于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (id <> ?)
     */
    @Test
    public void testNe(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.ne("id","3");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * gt 大于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age > ?)
     */
    @Test
    public void testGt(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.gt("user_age","18");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * ge 大于等于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age >= ?)
     */
    @Test
    public void testGe(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.ge("user_age","18");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * lt 小于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age < ?)
     */
    @Test
    public void testLt(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.lt("user_age","18");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * le 小于等于
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age <= ?)
     */
    @Test
    public void testLe(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.le("user_age","18");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * between 值1和值2之间,两边临界值都包含
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age BETWEEN ? AND ?)
     */
    @Test
    public void testBetween(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.between("user_age","18","25");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * notBetween 不在值1和值2之间，两边临界值都包含
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age NOT BETWEEN ? AND ?)
     */
    @Test
    public void testNoBetween(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.notBetween("user_age","18","25");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * like 模糊查询，会在参数左右两边加上 %
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name LIKE ?)
     */
    @Test
    public void testLike(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.like("user_name","可乐");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * notLike NOT LIKE ‘%parameter%’
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name NOT LIKE ?)
     */
    @Test
    public void testNotLike(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.notLike("user_name","可乐");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * likeLeft LIKE ‘%parameter’
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name LIKE '%parameter')
     */
    @Test
    public void testLikeLeft(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.likeLeft("user_name","可乐");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * likeRight LIKE ‘parameter%’
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name LIKE 'parameter%')
     */
    @Test
    public void testLikeRight(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.likeRight("user_name","可乐");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
    /**
     * isNull 判断字段为null
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name IS NULL)
     */
    @Test
    public void testIsNull(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.isNull("user_name");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * isNotNull 判断字段不为null
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_name IS NOT NULL)
     */
    @Test
    public void testIsNotNull(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.isNotNull("user_name");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * in 范围定值查询
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age IN (?,?,?))
     */
    @Test
    public void testIn(){
        QueryWrapper queryWrapper = new QueryWrapper();
        List<Integer> queryList = new ArrayList<>();
        queryList.add(18);
        queryList.add(1);
        queryList.add(25);
        queryWrapper.in("user_age",queryList);
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * notIn
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age IN (?,?,?))
     */
    @Test
    public void testNotIn(){
        QueryWrapper queryWrapper = new QueryWrapper();
        List<Integer> queryList = new ArrayList<>();
        queryList.add(18);
        queryList.add(1);
        queryList.add(25);
        queryWrapper.notIn("user_age",queryList);
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * inSql
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (id IN (select id from user))
     */
    @Test
    public void testInSql(){
        QueryWrapper queryWrapper = new QueryWrapper();
        //查询所有数据
        queryWrapper.inSql("id","select id from user");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * notInSql
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (id NOT IN (select id from user where id > 2))
     */
    @Test
    public void testNotInSql(){
        QueryWrapper queryWrapper = new QueryWrapper();
        //查询所有数据
        queryWrapper.notInSql("id","select id from user where id > 2");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * groupBy 分组
     * 下面SQL有个问题，在MySQL8.0版本中，是可以执行下面SQL语句的，select user_name并没有出现在group by 语句中
     * 实例SQL：SELECT id,user_name,user_age FROM user GROUP BY id,user_age
     */
    @Test
    public void testGroupBy(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.groupBy("id","user_age");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * orderByAsc 升序
     * 实例SQL：SELECT id,user_name,user_age FROM user ORDER BY id ASC,user_age ASC
     */
    @Test
    public void testOrderByAsc(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.orderByAsc("id","user_age");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * orderByDesc 降序
     * 实例SQL：SELECT id,user_name,user_age FROM user ORDER BY id DESC,user_age DESC
     */
    @Test
    public void testOrderByDesc(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.orderByDesc("id","user_age");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * orderBy 指定顺序排序
     * 实例SQL：SELECT id,user_name,user_age FROM user ORDER BY id ASC,user_age ASC
     */
    @Test
    public void testOrderBy(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.orderBy(true,true,"id","user_age");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * having
     * 实例SQL：SELECT id,user_name,user_age FROM user GROUP BY id,user_age HAVING sum(user_age)>?
     */
    @Test
    public void testHaving(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.groupBy("id","user_age");
        queryWrapper.having("sum(user_age)>{0}","25");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * having
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (id = ? OR user_age = ?)
     */
    @Test
    public void testOr(){
        QueryWrapper queryWrapper = new QueryWrapper();
        queryWrapper.eq("id",1);
        queryWrapper.or();
        queryWrapper.eq("user_age",25);
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * and
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE ((id = ? AND user_age <> ?))
     */
    @Test
    public void testAnd(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.and(i->i.eq("id",1).ne("user_age",18));
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * nested
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE ((id = ? AND user_age <> ?))
     */
    @Test
    public void testNested(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.nested(i->i.eq("id",1).ne("user_age",18));
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * apply
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (user_age>?)
     */
    @Test
    public void testApplyd(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.apply("user_age>{0}","25 or 1=1");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * last
     * 实例SQL：SELECT id,user_name,user_age FROM user limit 1
     */
    @Test
    public void testLast(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.last("limit 1 ");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
 
    /**
     * exists
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (EXISTS (select id from user where user_age = 1))
     */
    @Test
    public void testExists(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.exists("select id from user where user_age = 1");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
 
 
    /**
     * notExists
     * 实例SQL：SELECT id,user_name,user_age FROM user WHERE (EXISTS (select id from user where user_age = 1))
     */
    @Test
    public void testNotExists(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.notExists("select id from user where user_age = 1");
        List<User> list = userMapper.selectList(queryWrapper);
        System.out.println(list);
    }
}
```

### 4、LambdaQueryWrapper和LambdaUpdateWrapper（推荐）

LambdaQueryWrapper 和 LambdaUpdateWrapper 这是相对于 QueryWrapper 及 UpdateWrapper 的 Lmbda 语法实现方式。

分别通过如下两种方式获取：

```java
//两种方式        
LambdaQueryWrapper queryLambda = new QueryWrapper().lambda();
LambdaQueryWrapper lambdaQueryWrapper = new LambdaQueryWrapper<>();

//两种方式
LambdaUpdateWrapper updateLambda = new UpdateWrapper().lambda();
LambdaUpdateWrapper lambdaUpdateWrapper = new LambdaUpdateWrapper();
```

注意：获取LambdaQueryWrapper 和 LambdaUpdateWrapper 对象时，为了使用lambda语法，要使用泛型。

下面我演示几个实例：

```java
/**
 * LambdaQueryWrapper
 * SQL实例：SELECT id,user_name,user_age FROM user WHERE (id = ? AND user_age <> ?)
 */
@Test
public void testLambdaQueryWrapper(){
    LambdaQueryWrapper<User> queryLambda = new LambdaQueryWrapper<>();
    queryLambda.eq(User::getId,"1").ne(User::getUserAge,25);
    List<User> users = userMapper.selectList(queryLambda);
    System.out.println(users);
 
}
 
/**
 * LambdaQueryWrapper
 * SQL实例：UPDATE user SET user_name=? WHERE (user_name = ?)
 */
@Test
public void testLambdaUpdateWrapper(){
    User user = new User();
    user.setUserName("LambdaUpdateWrapper");
    LambdaUpdateWrapper<User> userLambdaUpdateWrapper = new LambdaUpdateWrapper<>();
    userLambdaUpdateWrapper.eq(User::getUserName,"IT可乐");
    userMapper.update(user,userLambdaUpdateWrapper);
 
}
```

### 5、总结

对于mybatis plus 中的四种条件构造器，我们就到此结束了，大家可以按照我的实例敲一遍代码，基本上就没啥问题了。 

有没有发现使用 Lamba 语法很爽，语法简洁，另外有个优点是，使用QueryWrapper或者UpdateWrapper时，对于条件的某个列，我们是写的字符串配置，比如 QueryWrapper.eq("id",1);这里的id是数据库表的列名，很有可能我们会写错，但是通过lambda 的方式，LambdaQueryWrapper.eq(User::getId,1)，这样就不会有写错的可能了。所以推荐大家使用Lambda 的方式。

至此，mybatis plus 的常规用法就全部介绍结束了，当然，事情还远没有结束，为了让大家用的更爽，后续将给大家介绍一些高阶玩法。　　　　
　　
