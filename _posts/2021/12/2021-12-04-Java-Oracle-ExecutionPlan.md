---
layout: post
title: Oracle执行计划
tagline: by 揽月中人
categories: Oracle
tags:
- 揽月中人
---
数据库执行SQL是都会先进行语义解析，然后将SQL分成一步一步可执行的计划，然后逐步执行。通过分析执行计划，我们可以清晰的看到数据库执行的操作，这对于数据库SQL的优化具有重大意义。
<!--more-->

### 1. 执行计划

用户成功连接数据库之后，用户和数据库成功建立起了会话。此后，用户每通过会话发出一条SQL语句，数据库系统都会对其进行一系列检查、分析、处理。

同时优化器会对SQL进行一些优化，并选择出一个它觉得最优的执行计划，然后再去执行这些操作。由于SQL不同的写法会影响优化器为之生成和选定的执行计划。所有我们就可以通过改写SQL语句来改变其执行计划，从而提升SQL语句性能。

### 2. 系统统计数据

系统统计数据反应了数据库系统的处理能力，会对执行计划中左右操作成本（其实就是性能消耗）计算产生重要影响。系统统计数据主要包括转速、单块读消耗时间、多块读消耗时间、多块读平均每次读取的数据块等。

系统统计数据会影响优化器计算分析SQL语句执行计划的成本所选择的算法，也会影响SQL语句生成和选择的执行计划。

### 3. 对象统计数据

优化器对SQL进行解析的时候，会根据系统统计数据和对象统计数据等星系，计算成本，最后选出最低成本的执行计划。由于系统统计数据认为很难干涉，所以对象统计数据对于SQL执行计划来说影响更大。

对象统计数据主要包括三个部分：表（分区及子分区）相关统计数据、索引相关统计数据和字段相关统计数据。所以收集这些信息则可以进行对象统计数据的分析，从而进行SQL优化。

### 4. 获取执行计划

获取执行计划有多种方法，下面分别介绍一下。

#### 4.1 通过各种GUI工具获得执行计划

通过各种GUI可以获取到执行计划，其优点是操作简单，灵活；获取的信息也比较多。

下面是通过Sql Developer中的工具直接获取到的执行计划示例

![image-20211219210349202](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-GUI.png)



#### 4.2 autotrace功能

autotrace功能是Oracle公司的产品，其功能强大、使用灵活，因而应用广泛。

4.2.1使用方法介绍

```sql
set autot off  关闭autotrace功能
set autot on 开启autotrace功能,输出SQL语句的查询结果，执行计划以及相关的性能统计数据
set autot on expl 开启autotrace功能，输出SQL语句的查询结果，执行计划，不输出性能统计数据
set autot on stat 开启autotrace功能，输出SQL语句的查询结果以及相关性能数据，不输出执行计划
set autot trace 开启autotrace功能，只输出SQL语句的执行计划以及性能数据，不输出查询结果
set autot trace expl 开启autotrace功能，只输出SQL的执行计划，不输出查询结果及性能数据
set autot trace stat 开启autotrace功能，只输出SQL的性能统计数据，不输出执行计划以及查询结果

```

如下示例：

```sql
set autotrace on
select * from emp join DEPT on emp.DEPTNO = DEPT.DEPTNO
where DNAME = 'SALES';
```

![](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-autotrace.png)

图中输出了执行计划以及性能数据.

#### 4.3 使用DBMS_XPLAN包

DBMS_XPLAN是Oracel数据库的内置包，该包提供了多个函数，通过这些函数，用户可以比较容易的获取执行计划等数据。

##### 4.3.1 DISPLAY方法

```SQL
DBMS_XPLAN.DISPLAY(
	table_name in varchar2 default 'PLAN_TABLE',
    statement_id in varchar2 default null,
    format in varchar2 default 'TYPICAL',
    filter_preds in varchar2 default null);
```

以上是DISPLAY的语法，默认执行计划存储表为PLAN_TABLE，如果要查询此表需要有SELECT的权限。

其中的参数含义如下：

- table_name ： 存储执行计划的表名。
- statement_id ： SQL语句的ID ，可以使用set statement_id 来指定其ID。如果为null，则表示获取最近被解释的SQL的执行计划。
- format ： 执行计划的具体输出级别 其值有 
  - 'BASIC' ： 基本输出，经输出执行计划中每个节点),
  - 'TYPICAL' ： 典型格式输出，默认格式。该格式输出每个节点的ID、操作名、节点的数据行、字节数、优化成本等。
  - 'SERIAL' ： 串行执行格式，输出与典型格式类似。
  - 'ALL' ： 完全格式， 最高用户级别的输出格式，除了输出典型格式的内容，还会输出投影以及别名的相关信息。

示例如下：

```sql
explain plan for 
select * from emp join DEPT on emp.DEPTNO = DEPT.DEPTNO
where DNAME = 'SALES';
select * from table(dbms_xplan.display())
```

![image-20211219222428539](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-xplan-display.png)



为了更好的控制执行计划的输出格式，如下的关键字可以添加到标准格式后面，用来自定义输出格式以及信息。

- ROWS 输出优化器估算出的数据行数
- BYTES 输出优化器估算出的字节数
- COST 输出优化器估算出的成本
- PARTITION 输出分区裁剪相关信息
- PREDICATE 输出谓词部分相关信息
- PARALLEL 输出并行操作(PX)相关信息
- PROJECTION 输出字段映射部分相关信息
- ALIAS 输出查询块/对象 别名相关信息
- REMOTE 输出分布式查询相关信息
- NOTE 输出执行计划的提醒部分相关信息



示例如下：

```sql
explain plan for 
select * from emp join DEPT on emp.DEPTNO = DEPT.DEPTNO
where DNAME = 'SALES';
select * from table(dbms_xplan.display());
select * from table(dbms_xplan.display(null,null,'BASIC ROWS BYTES'));
select * from table(dbms_xplan.display(null,null,'ALL -PROJECTION -NOTE'));
select * from table(dbms_xplan.display(null,null,'ALL PROJECTION NOTE'));
```



##### 4.3.2 DISPLAY_CURSOR方法

语法如下

```SQL
DBMS_XPLAN.DISPLAY_CURSOR(
	sql_id in varchar2 default null,--默认获取会话最后一个游标处的执行计划
    child_number in number default null,--游标的子号
    format in varchar2 default 'TYPICAL' --输出级别，与之前介绍相同
);
```

此函数可以获取内存游标缓存处的执行计划和统计信息。

示例如下：

```sql
alter session set statistics_level = all;
select * from emp join DEPT on emp.DEPTNO = DEPT.DEPTNO
where DNAME = 'SALES';
select * from table(dbms_xplan.display_cursor(null,null,'allstats last'));
```

执行结果：

![image-20211219192145828](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-alllstats.png)



以下函数使用较少，所以仅介绍其语法及功能。

##### 4.3.3 DISPLAY_AWR

语法如下

```sql
DBMS_XPLAN.DISPLAY_AWR(
    sql_id IN varchar2
	plan_hash_value in number default null,
    db_id in number default null,
    format in varchar2 default 'TYPICAL');
```

DISPLAY_AWR函数获取存储在AWR历史库中SQL语句的执行计划相关信息。

##### 4.3.4 DISPLAY_PLAN

语法如下

```SQL
DBMS_XPLAN.DISPLAY_PLAN(
    table_name in varchar2 default 'PLAN_TABLE',
    statement_id in varchar2 default null,
    format in varchar2 default 'TYPICAL',
    filter_preds in varchar2 default null,
    type in varchar2 default null --输出类型，其值为'TEXT','ACTIVE','HTML','XML'
);
```

该函数可获取执行计划存储表的内容。可显示CLOB类型信息，包括执行计划以及相关统计信息。

##### 4.3.5 DISPLAY_SQL_PLAN_BASELINE

语法如下

```SQL
DISPLAY_XPLAN.DISPLAU_SQL_PLAN_BASELINE(
    sql_handle in varchar2 := null,
    plan_name in varchar2 := null,
    format in varchar2 := 'TYPICAL')
return dbms_xpaln_type_table;
```

此函数和获取存储在系统视图中SQL语句计划基线的执行计划相关的信息。

##### 4.3.6 DISPLAY_SQLSET

```SQL
DBMS_XPLAN.DISPLAY_SQLSET(
	sqlset_name in varchar2,
    sql_id in varchar2,
    plan_hash_value in number := null,
    format in varchar2 := 'TYPICAL',
    sqlset_owner in varchar2 := null
)
return DBMS_XPLAN_TYPE_TABLE PIPELINED;
```

此函数获取存储在SQL调优集中SQL语句的执行计划以及相关信息。



#### 4.4 查询PLAN_TABLE获取执行计划

我们可以通过编写的SQL语句来查询执行计划。即直接查询执行计划存储表（默认为PLAN_TABLE）

```SQL
explain plan SET STATEMENT_ID = 'TEST1' for 
select * from emp join DEPT on emp.DEPTNO = DEPT.DEPTNO
where DNAME = 'SALES';

SELECT  ID, PARENT_ID ,OPERATION ,OBJECT_NAME NAME , BYTES ,IO_COST ,CPU_COST
FROM PLAN_TABLE WHERE STATEMENT_ID = 'TEST1' ORDER BY ID ;
```

![image-20211220164443704](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-FROM-PLAN-TABLE.png)

或者使用如下SQL查询

```sql
SELECT  ID, PARENT_ID ,
    LPAD(' ', LEVEL-1)||OPERATION||' '||OPTIONS||' '||OBJECT_NAME NAME
FROM PLAN_TABLE 
CONNECT BY prior id = parent_id 
    and prior statement_id = statement_id 
start with id = 0 
    and statement_id = 'TEST1'
 ORDER BY ID ;
```

结果如下

![image-20211220165521315](https://www.javanorth.cn/assets/images/2021/lyj/Oracle-explain-from-plan_table1.png)

#### 4.5 跟踪计划

通过对SQL语句进行跟踪，从而获取相关执行计划等。

主要方法有SQL_TRACE 和OPTIMIZER_TRACE ，前者会在跟踪文件里输出执行计划及性能统计等相关数据。OPTIMIZER_TRACE 在跟踪文件里记录优化器分析、选择执行计划的过程。





