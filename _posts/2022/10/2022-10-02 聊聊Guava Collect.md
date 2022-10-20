---
layout: post
title:  聊聊Guava Collect -20221012
tagline: by 沉浮
categories: Guava 集合
tags: 
- 沉浮
---

集合操作是编程中使用频率非常高的，所有有一款针对集合的操作工具是非常有必要的。
<!--more-->
## Guava Collect

集合操作是编程中使用频率非常高的，所有有一款针对集合的操作工具是非常有必要的。
通过框架提供的工具一方面可以减少开发相似功能的耗时；同时框架在安全与稳定性上更被推荐。

Guava Collect是Guava工具包中的一个子模块，主要对jdk中的集合操作添加了一些简易的API，同时也是对Collections工具类的扩展。
当然Guava还定义了一些特定场景的数据结构以及一些针对jdk集合的优化，最典型的就是Immutable Collections（不可变集合），你会发现调用Guava API很多都是不可变的

### 意义

我们常见的集合类有：
- List
- Set
- Vector
- Stack
- Map
- Queue

集合是一种非常常见的数据结构，JDK在处理各种数据集时，提供了以上集合类型的数据结构以及其对应API方便开发者高效简易地对数据对象操作

### 特色

guava主要提供了以下几个方面的支持：

 + 增加了不可变集合
   - 不受信任的库可以安全使用。
   - 线程安全：可以被许多线程使用，没有竞争条件的风险。
   - 不需要支持突变，并且可以通过该假设节省时间和空间。所有不可变集合实现都比它们的可变兄弟更节省内存。（分析）
   - 可以用作常数，期望它保持不变。

 + 增加了新的集合类型
   - Multiset
   与普通的Set相比，提供了元素出现频率的记录。可用于元素出现次数的记录
   
   - Multimap
   一个与Map相比，一个建可以对应对应多个值。与Spring中MultiValueMap一样
   
   - BiMap
   键值都是唯一的Map
   
   - Table
   具有行、列的表格，数据视图中可能更直观。
   
   - ClassToInstanceMap
   键为Class，值为Class实例的特殊Map
   
   - RangeSet
   代表一组数据区间，类似数学中的 [1,9)
   
   - RangeMap
   与RangeSet类似，不过将其区间作为建，可以有自己的值。 [1,9) -> 'VAL'
  
 + 优化了常用的操作
   - 集合的创建\
     ImmutableSet.of(elem ...)\
     Lists.newArrayList(elem ...)\
     Sets.newHashSet(elem ...)\
     Maps.newHashMap()\
     ...
   - 常用的操作
     判断两个集合是否相等： Iterables.elementsEqual()\
     集合分段处理：Lists.partition()\
     取集合的交集：Sets.intersection()\
     取集合的差集：Sets.difference()\
     ...

### 使用

Guava Collect作为集合操作工具，我们主要从实际业务中了解其能够帮助我们实现怎样的需求，下面看下其API的使用情况：

假设我们有10000名学生，通过Faker生成这些模拟的学生数据数据：
```
List<Student> students = new ArrayList<>();
Faker faker = new Faker(Locale.CHINA);
@Before
public void init(){
    Faker enFaker = new Faker();
    Name name = faker.name();
    IntStream.range(0,10000).forEach(index->{
        students.add(
        Student.of()
        .setId(String.valueOf(index+1))
        .setName(name.name())
        .setAge(faker.number().numberBetween(18,22))
        .setGender(new String[]{"男","女"}[faker.number().numberBetween(0,2)])
        .setAddress(faker.address().streetAddress())
        .setScore(faker.number().randomDouble(3,50,100))
        .setEmail( faker.internet().emailAddress(enFaker.name().username()))
        .setTelephone(faker.phoneNumber().cellPhone())
        );
    });
}
```

> **Multiset**  
获取元素出现频次。
比如获取男生与女生的学生数量分别为多少
```
    @Test
    public void multiset(){
        Multiset multiset = HashMultiset.create();
        students.forEach(student -> {
            if(Objects.equals(student.getGender(),"男")){
                multiset.add("男");
            }else{
                multiset.add("女");
            }
        });

        System.out.println("学生中男生数量:"+ multiset.count("男"));
        System.out.println("学生中女生数量:"+ multiset.count("女"));
    }
```
> **Multimap**\
一个键对应多个值时。
比如查看各个年龄的学生是哪些
```
    @Test
    public void multimap(){
        ListMultimap<Integer, Student> multimap =
                MultimapBuilder.hashKeys().arrayListValues().build();

        students.forEach(student -> {
            multimap.put(student.getAge(),student);
        });

        System.out.println( multimap.get(20) );
    }
```
> **BiMap**     
键和值都是唯一时。
比如处理学生的邮箱和手机号，客户互换键值位置
```
    @Test
    public void biMap(){
        BiMap biMap = HashBiMap.create();

        students.forEach(student -> {
            biMap.put(student.getEmail(),student.getTelephone());
        });

        BiMap inverse = biMap.inverse();// 键值更换

        System.out.println( biMap );
        System.out.println( inverse );
    }
```
> **Table**     
二维表，通过行（键）、列（键）取值
比如可以以学生为行数据，其中id为行键，列名分别为学生属性名称
 
| ID  | 姓名  | 年龄  | 性别  |
|-----|-----|-----|-----|
| 1   | TOM | 22  | 男   |

```
    @Test
    public void table(){
        Table<String, String, Object> weightedGraph = HashBasedTable.create();
        students.forEach(student -> {
            weightedGraph.put(student.getId(), "姓名", student.getName());
            weightedGraph.put(student.getId(), "年龄", student.getAge());
            weightedGraph.put(student.getId(), "性别", student.getGender());
            weightedGraph.put(student.getId(), "邮箱", student.getEmail());
            weightedGraph.put(student.getId(), "电话", student.getTelephone());
            weightedGraph.put(student.getId(), "地址", student.getAddress());
            weightedGraph.put(student.getId(), "分数", student.getScore());
        });

        Map<String, Object> row = weightedGraph.row("1");
        Map<String, Object> column = weightedGraph.column("姓名");
        Set<Table.Cell<String, String, Object>> cells = weightedGraph.cellSet();

        System.out.println( row );
        System.out.println( column );
        System.out.println( cells );
    }
```
> **ClassToInstanceMap**    
当值是键的类型实例时，通过该Map现在键值关系
```
    @Test
    public void classToInstanceMap(){
        ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
    
        numberDefaults.put(Number.class,1);
    
        Map<Class,Object> objectMap = new HashMap<>();
        objectMap.put(Number.class,2);
    }
```
> **RangeSet**      
区间Set。
比如通过学生分数确定学生等级
```
    @Test
    public void rangeSet(){
        RangeSet<Double> ArangeSet = TreeRangeSet.create();
        ArangeSet.add(Range.closed(90d,100d)); // [90,100]
        RangeSet<Double> BrangeSet = TreeRangeSet.create();
        BrangeSet.add(Range.closedOpen(80d,90d)); // [80,90)
        RangeSet<Double> CrangeSet = TreeRangeSet.create();
        CrangeSet.add(Range.closedOpen(70d,80d)); // [70,80)
        RangeSet<Double> DrangeSet = TreeRangeSet.create();
        DrangeSet.add(Range.closedOpen(60d,70d)); // [60,70)
        RangeSet<Double> ErangeSet = TreeRangeSet.create();
        ErangeSet.add(Range.lessThan(60d)); // [...,60)

        students.forEach(student -> {
            System.out.print( " 学生："+ student.getName() );
            System.out.print( ",分数为："+ student.getScore() );
            String rank = "";
            if(ArangeSet.contains(student.getScore())){
                rank = "A";
            }else if(BrangeSet.contains(student.getScore())){
                rank = "B";
            }else if(CrangeSet.contains(student.getScore())){
                rank = "C";
            }else if(DrangeSet.contains(student.getScore())){
                rank = "D";
            }else if(ErangeSet.contains(student.getScore())){
                rank = "E";
            }
            System.out.print( ",等级为："+ rank +"\n");
        });
    }
```
> **RangeMap**
和RangeSet类似，区别是添加了区间命名。
和上面一样
```
    @Test
    public void rangeMap(){
        RangeMap<Double, String> rangeMap = TreeRangeMap.create();
        rangeMap.put(Range.closed(90d,100d),"A"); // [90,100]
        rangeMap.put(Range.closedOpen(80d,90d),"B"); // [80,90)
        rangeMap.put(Range.closedOpen(70d,80d),"C"); // [70,80)
        rangeMap.put(Range.closedOpen(60d,70d),"D"); // [60,70)
        rangeMap.put(Range.lessThan(60d),"E"); // [...,60)

        students.forEach(student -> {
            System.out.print( " 学生："+ student.getName() );
            System.out.print( ",分数为："+ student.getScore() );
            System.out.print( ",等级为："+ rangeMap.get(student.getScore()) +"\n");
        });
    }
```

#### 下面看下对常用集合的一些操作
当然我们首先需要将数据使用Guava Collect对应的数据结构来存储数据，这样才能使用其对应的API：

+ 集合创建
  FluentIterable.of(elem ...)\
  Lists.newArrayList(elem ...)\
  Sets.newHashSet(elem ...)\
  Maps.newHashMap()\
  HashMultiset.create()\
  ArrayListMultimap.create()\
  Tables.newCustomTable(Maps.newLinkedHashMap(), () -> Maps.newLinkedHashMap())\

+ 条件过滤  
  FluentIterable.filter(predicate);
  FluentIterable.anyMatch(predicate);
  FluentIterable.allMatch(predicate);
  FluentIterable.firstMatch(predicate);

+ 拆分
  Iterables.partition(list, pageSize); // 拆解集合

+ 计算
  Iterables.frequency(list, elem); //元素出现的次数

+ 集合的并集、交集、差集
  // 并集
  Sets.union(set1, set2);
  // 交集
  Sets.intersection(set1, set2);
  // 差集  set1为参考
  Sets.difference(set1, set2);
  // 并集-交集
  Sets.symmetricDifference(set1, set2);
  // 同上
  Sets.difference(Sets.union(set1, set2),Sets.intersection(set1, set2) );
  // 笛卡尔积
  Sets.cartesianProduct(Arrays.asList(Sets.newHashSet(1, 2, 3), Sets.newHashSet(3, 4, 5, 6));
  // Map,KV相同的部分
  difference.entriesInCommon();
  // 同K不同V
  difference.entriesDiffering();
  // 左边存在的右边不存的K
  difference.entriesOnlyOnLeft();
  // 右边存在的左边不存的K
  difference.entriesOnlyOnRight();

+ 索引
  // 将元素中的子项作为索引，由于元素检索
  Maps.uniqueIndex()
  Multimaps.index()

### Jdk中的集合操作

 自从Jdk中引入了集合Stream的操作后，从很大程度上简化了对集合的操作，以前大量代码现在可能简单几行就能够达到相同的效果，同时支持并发处理，一并提升了效率。

下面看下常见的集合基于stream操作，同样以上面的学生为例:

> **遍历** forEach
```
    @Test
    public void forEach(){
        students.stream().forEach(System.out::println);
    }
```

> **转换** map    
将元素转换成其他类型。
比如根据学生名称、性别组成新的List；以id为键元素为值的Map或者学生姓名拼接的字符串等等
```
    @Test
    public void transform(){
        // 转换为数组
        List<String> listResult = students.stream()
        .map((val)-> val.getName() + ":" + val.getGender()).collect(Collectors.toList());
        System.out.println( listResult );
    
        // 转换成String
        String stringResult = students.stream().map(Student::getName).collect(Collectors.joining());
        System.out.println( stringResult );
    
        // 转换成Map
        Map<String, Student> mapResult = students.stream().collect(
        // key ,value ,mergerOperation, initialization
        Collectors.toMap(Student::getName,Student::self,(v1,v2)->{
            // 出现相同key时的合并规则
            return null;
            },HashMap::new)
        );
        System.out.println( mapResult );
    }
```

>**过滤** filter   
根据条件匹配满足要求的元素。如找出分数大于80分的学生
```   
     @Test
    public void filter(){
        List<Student> filterResult = students.stream().filter((val)->{
            return val.getScore()>80;
        }).collect(Collectors.toList());
        System.out.println(filterResult);
    }
```

> **拆解** flatMap     
将二层级集合进行拆解，并成一级集合。如[[1,2,3],[4,5,6]] -> [1,2,3,4,5,6]
```
    @Test
    public void flatMap(){
        //复合拆解
        List<Integer> result = Stream.of(Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6))
                .flatMap(subList -> subList.stream())
                .collect(Collectors.toList());
        System.out.println(result);// 1 2 3 4 5 6
    }
```

> **计算**
实现数据的汇总、求平均值、最大值...，当然主要针对数字（Number）类型
```
    @Test
    public void calculate(){
        // 求和
        double sum = students.stream().mapToDouble(Student::getScore).sum();
        // 最大值
        double max = students.stream().mapToDouble(Student::getScore).max().getAsDouble();
        // 最小值
        double min = students.stream().mapToDouble(Student::getScore).min().getAsDouble();
        // 平均值
        double avg = students.stream().mapToDouble(Student::getScore).average().getAsDouble();
        // 归约运算 fold . count、sum、min、max、average
        DoubleSummaryStatistics doubleSummaryStatistics = students.stream().mapToDouble(Student::getScore).summaryStatistics();
    }
```

> **归纳计算** reduce   
在很多语言中都存在的函数，如python、javascript。数据的累加、map的功能
```
    @Test
    public void reduce(){
        // 结果和identity（初始值）类型相同
        // identity accumulator combiner
        Map result = students.stream().reduce(
                new HashMap<String,Student>(), //初始值
                (map, student) -> {
                    map.put(student.getId(),student);
                    return map;
                },
                (map1, map2) -> {
                    // 并发执行时的map合并
                    return null;
                }
        );
    }
```

> **并发** parallel       
上面的操作我们还可以使用parallel对stream并发处理
```
    Arrays.asList().stream().parallel()...;
    Arrays.asList().parallelStream()...;
```

> **分段处理**
对集合按固定规格分段处理，处理大批量数据时，结合parallel实现分段并发处理来提示效率
```
    @Test
    public void partition(){
        List<String> list = new ArrayList<>();
        int partition = 100; //每段100个元素
    
        int part = list.size() / partition  + (list.size() % partition==0? 0:1);
        Stream.iterate(0, n -> n+1)
                .limit(part)
                .parallel() //并发
                .map(index -> list.stream().skip(index * partition).limit(partition).parallel().collect(Collectors.toList()))
                .forEach(System.out::println);
    }
```

### 总结

本章主要介绍了Guava Collect部分，以及对集合操作的常用API，通过示例可以看到有其对JDK集合的扩展有了更广泛与简易的操作。同时在JDK引入
了Stream操作后，Guava Collect中的很多功能通过Stream也可以比较容易的实现了，当然具体如何选择根据实际情况。需要注意的是Guava Collect中
返回的基本都是不可变的集合，这样在对数据的操作会更加的安全。
