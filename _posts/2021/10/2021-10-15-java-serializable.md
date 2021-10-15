---
layout: post
title:   java 中的序列化
tagline: by 某某白米饭
categories: java
tags:
- 某某白米饭
---

大家好，我是指北君。

java 对象经常需要在网络中以 socket 传输或者需要保存到文件中。这时不管 java 对象是文件、数据、图像还是其他格式，都可以转换为一个 byte[] 数组保存到文件或者通过网络传输。这种转换方式就叫做序列化。将文件或者网络传输中得到的 byte[] 数组转换为 java 对象就叫做反序列化。

<!--more-->

### 怎么使用

如果一个 Java 对象要能被序列化，必须实现一个特殊的 java.io.Serializable 接口

```java
public interface Serializable {
}
```

Serializable 接口没有定义任何的方法，是一个空接口。为什么要有一个这样的接口？主要是因为安全。如果没有这个接口就代表着所有 java 对象都可以被序列化到磁盘上，然后通过反序列化看到所有属性的数据。有了这个 Serializable 就可以让开发人员选择 java 对象可以被序列化和反序列化，就增加了安全性。

### 序列化

将一个对象序列化后保存到文件

```java
import java.io.Serializable;

public class Order  implements Serializable {

    private Long orderId;
    private String orderNo;
    private String consignee;//收件人
    private String deliveryAddress;//收货地址

    //getter 和 setter

    @Override
    public String toString() {
        return "OrderDTO{" +
                "orderId=" + orderId +
                ", orderNo='" + orderNo + '\'' +
                ", consignee='" + consignee + '\'' +
                ", deliveryAddress='" + deliveryAddress + '\'' +
                '}';
    }
}
```

把一个 Java 对象变为 byte[] 数组，需要使用 ObjectOutputStream。它负责把一个Java 对象写入一个字节流：

```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.util.ArrayList;
import java.util.List;

public class Test {

    public static void main(String[] args) throws Exception {
        OrderDTO orderDTO = new OrderDTO();
        orderDTO.setOrderId(1L);
        orderDTO.setOrderNo("123456");
        orderDTO.setConsignee("李四");
        orderDTO.setDeliveryAddress("xxx路xxx弄xxxx号");

        OrderDTO orderDTO2 = new OrderDTO();
        orderDTO2.setOrderId(2L);
        orderDTO2.setOrderNo("78901");
        orderDTO2.setConsignee("王五");
        orderDTO2.setDeliveryAddress("yyy路yyy弄yyyy号");

        List<OrderDTO> list = new ArrayList<>();
        list.add(orderDTO);
        list.add(orderDTO2);

        FileOutputStream fos = new FileOutputStream("D:/order.txt");
        try ( ObjectOutputStream os = new ObjectOutputStream(fos)){
            os.writeObject(list);
        }
        System.out.println("序列化成功");
    }
}
```

这个时候就将两个 OrderDTO 对象序列化到了 D:/order.txt 中。

### 反序列化

序列化文件在本地打开都是乱码的，这应该用反序列化将文件解析成对象。

```java
import java.io.FileInputStream;
import java.io.ObjectInputStream;
import java.util.ArrayList;
import java.util.List;

public class Test {

    public static void main(String[] args) throws Exception {

        List<OrderDTO> list = new ArrayList<>();
        FileInputStream fis = new FileInputStream("D:/order.txt");
        try (ObjectInputStream is = new ObjectInputStream(fis)) {
            list = (List<OrderDTO>)is.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        for (OrderDTO orderDTO : list){
            System.out.println(orderDTO.toString());
        }

    }
}
```

输出结果:

```java
OrderDTO{orderId=1, orderNo='123456', consignee='李四', deliveryAddress='xxx路xxx弄xxxx号'}
OrderDTO{orderId=2, orderNo='78901', consignee='王五', deliveryAddress='yyy路yyy弄yyyy号'}
```

### 注意点

1. 静态变量和 transient 关键字修饰的变量不能被序列化
2. 反序列化的时候，字节流中的 serialVersionUID 和实体类中的 serialVersionUID 的不一致会抛出异常。serialVersionUID 没有写的话，会被默认一个。
3. 序列化实现了深克隆，对象引用的每一个对象数据也会被序列化。

### 总结

1. 序列化必须实现 Serializable。
2. serialVersionUID 不是必须的。

