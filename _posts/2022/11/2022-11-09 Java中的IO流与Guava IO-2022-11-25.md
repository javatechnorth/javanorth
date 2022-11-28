---
layout: post
title:  2022-11-09 Java中的IO流与Guava IO
tagline: by 沉浮
categories: Guava IO流
tags: 沉浮
---

哈喽，大家好，我是指北君。  


<!--more-->

## Guava IO

系统间数据的交互、文件的上传与下载都是在开发中经常遇到的，此时我们会通过jdk提供的IO操作库帮助我们实现。IO指的是数据相对当前操作程序的入与出，将数据通过
输出流从程序输出，或者通过输入流将数据（从文件、网络、数据等）写入到程序，这里的IO指的是基于流作为载体进行数据传输。
如果把数据比作合理的水，河就是IO流，也是数据的载体。

Java为我们提供了非常多的操作IO的接口与类，帮助开发者实现不同源间的数据传输，比如硬盘文件、网络传输、应用调用间的数据交互与传递。今天我们来简单了解下Java中的流
以及在Guava工具包中，针对IO操作做了什么样的封装与设计。

### 分类
在java.io包中有非常多的IO相关接口，我们可以根据流的输出类型、处理对象以及功能将其分为以下几种类型：

+ 按数据流向
    - 输入流 (java.io.InputStream)  
    用于实现将数据读入到程序  

    - 输出流 (java.io.OutputStream)  
      用于实现将数据从程序写出

+ 按操作单位
    - 字节流  
      以字节（byte）为单位进行数据的读、写
      （其中针对文件也提供了按基础数据类型的读与写DataInpoutStream，也就是按照Java基础类型所占字节数来进行定量字节读取并合并）
  
    - 字符流  
      以字符（char）为单位进行数据的读、写，此时需要注意字符编码

> 区分：
>> 字节流一般以Stream结尾
>> 字符流一般以Reader或Writer结尾

+ 按操作方式
   - 读 (java.io.Reader)  
     主要针对字符流的读取操作
  
   - 写 (java.io.Writer)  
     主要针对字符流的写出操作

+ 按功能
    - 缓存流  
     按字节进行数据读写时，通过缓冲批量写入来提高传输效率
  
    - 转换流  
    实现输入/出与读/写方式间的转换

### 常用的流

+ 操作文件的   
java.io.FileinputStream/FileOutputStream
java.io.FileReader/FileWriter

+ 通用的字节流  
java.io.InputStreamReader/outputStreamWriter

+ 缓冲流  
java.io.BufferedReader/BufferedWriter
java.io.BufferedInputStream/BufferedOutputStream

+ 数据流  
java.io.DataInpoutStream/DataOutputStream

+ 功能型的  
java.io.PrintWriter/PrintStream

+ 对象序列化相关的  
java.io.ObjectInputStream/ObjectOutputStream

> 可见，提供的IO对象基本都是成对出现的，用以完成数据的输入输出，实现程序与外部载体间的数据交换

### 示例

下面我们通过一些常用示例来看看IO的使用的场景与使用方法：
 + 文件复制
 + 文件的合并
 + 读取文件内容为字符串
 + 字节数组转换成流
 + 对象序列化与反序列化
 + 流的转换
 + ......

---

 + 文件复制
```
    @Test
    public void copyByBytes() throws IOException {
        String root = FileTests.class.getResource("/").getPath();

        FileInputStream fis = new FileInputStream(new File(root,"/start.bat"));
        FileOutputStream fos = new FileOutputStream(root+"/out2.bat");

        byte[] buff = new byte[100];

        int b;
        while ( (b = fis.read(buff))!=-1 ){
            fos.write(buff, 0, b);
        }
        // close
    }
```

+ 文件合并
```
@Test
    public void mergeFiles() throws IOException {
        File file1 = new File("E:\\_projects\\sucls\\blog\\my_study\\guava\\guava-io\\src\\test\\java\\com\\sucls\\blog\\guava\\io\\category\\FileTests.java");
        File file2 = new File("E:\\_projects\\sucls\\blog\\my_study\\guava\\guava-io\\src\\test\\java\\com\\sucls\\blog\\guava\\io\\category\\StreamTests.java");

        Enumeration<InputStream> ins = Collections.enumeration(Arrays.asList(
                new FileInputStream(file1),
                new FileInputStream(file2)
        ));

        SequenceInputStream sequenceInputStream = new SequenceInputStream(ins);

        FileOutputStream fos = new FileOutputStream(root+"/out4");

        byte[] buff = new byte[100];

        int read; // 真实读取到的字节数
        while ( (read = sequenceInputStream.read(buff)) !=-1){
            fos.write(buff, 0, read);
        }

        fos.close();
    }
```

+ 读取文件内容为字符串
```
    @Test
    public void readStringFromFile() throws IOException {
        FileReader fileReader = new FileReader(new File(this.getClass().getResource("/").getPath(),"/start.bat"));

        StringBuilder stringBuilder = new StringBuilder();

        int i;
        while ( (i = fileReader.read())!=-1 ){
            stringBuilder.append( (char)i ); // 按字符读取
        }

        System.out.println( stringBuilder ); // 文件内容
    }
```

 + 字节数组转换成流
```
    @Test
    public void bytesToStream(){
        byte [] data = new byte[1024]; // 来源于其他数据源

        ByteArrayInputStream inputStream = new ByteArrayInputStream(data);

        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();

        int v;
        while ( (v=inputStream.read())!=-1 ){
            outputStream.write(v);
        }

        System.out.println( Arrays.toString( outputStream.toByteArray() ));
    }
```

+ 对象序列化与反序列化
```
@Test
    public void objectToFile() throws IOException {

        Person person = new Person();
        person.setName("张三").setAge(25);

        String root = FileTests.class.getResource("/").getPath();

        FileOutputStream fos = new FileOutputStream(new File(root,"/person"));
        ObjectOutputStream oos = new ObjectOutputStream(fos);

        oos.writeObject(person);
    }

    @Test
    public void fileToObject() throws IOException, ClassNotFoundException {
        String root = FileTests.class.getResource("/").getPath();

        FileInputStream fis = new FileInputStream(new File(root,"/person"));
        ObjectInputStream ois = new ObjectInputStream(fis);

        Person person = (Person) ois.readObject();
        System.out.println( person );
    }
```

+ 流的转换
将字节流转换成字符流来操作，同样以文件复制为例
```
    @Test
    public void copyByBuffer() throws IOException {
        String root = FileTests.class.getResource("/").getPath();

        FileInputStream fis = new FileInputStream(new File(root,"/start.bat"));
        InputStreamReader isr = new InputStreamReader(fis);
        BufferedReader br = new BufferedReader(isr);

        FileOutputStream fos = new FileOutputStream(root+"/out3.bat");
        OutputStreamWriter osw = new OutputStreamWriter(fos);
        BufferedWriter bw = new BufferedWriter(osw);

        String line;
        while ( (line = br.readLine())!=null ){
            bw.append(line);
            bw.newLine();
            bw.flush();
        }

        // close
    }
```

> 关于流的操作非常多，像包括网络通信中、音视频文件处理、流合并等等

### Guava中的IO

关于IO的内容并不复杂，上面的那些例子在很多工具库中基本都会提供对应的API方便开发者调用，今天主要看下Guava IO模块针对流的操作提供了什么样的
封装

> Files
>> 提供对文件快捷读写方法
>> 其中主要提供了ByteSource、ByteSink、CharSource、CharSink 4个类，分别对应按字节的读写与按字符的读写，

```
 /**
     * 文件复制
     */
    @Test
    public void copy() throws IOException {
        File from = new File(root,"from");
        File to = new File(root,"to");
        Files.copy(from,to);
    }

    /**
     * 文件移动
     */
    @Test
    public void move() throws IOException {
        File from = new File(root,"from");
        File to = new File(root,"to");
        Files.move(from,to);
    }

    /**
     * 按行读取文件
     * @throws IOException
     */
    @Test
    public void readLines() throws IOException {
        File dest = new File(root,"start.bat");
        List<String> lines = Files.readLines(dest, Charset.defaultCharset());
        lines.forEach(System.out::println);
    }

    /**
     * 写入文件
     * @throws IOException
     */
    @Test
    public void writeToFile() throws IOException {
        File dest = new File(root,"demo.txt");
        Files.write("hello world!".getBytes(Charset.defaultCharset()), dest);
    }

    /**
     * 修改文件更新时间
     * @throws IOException
     */
    @Test
    public void touch() throws IOException {
        File dest = new File(root,"demo.txt");
        Files.touch(dest);
    }

    /**
     * 文件的零拷贝
     * @throws IOException
     */
    @Test
    public void map() throws IOException, URISyntaxException {
        File from = new File(root,"from");
        File to = new File(root,"to");
        Files.touch(to);
        
        MappedByteBuffer fromBuff = Files.map(from, MapMode.READ_ONLY, 1024);
        // =>
        FileChannel channel = FileChannel.open(Paths.get(to.toURI()), StandardOpenOption.WRITE);

        channel.write(fromBuff);

        channel.close();
    }

    /**
     * 读文件为字节数组
     * @throws IOException
     */
    @Test
    public void fileAndBytes() throws IOException {
        File dest = new File(root,"start.bat");
        ByteSource byteSource = Files.asByteSource(dest);
        byte[] bytes = byteSource.read();
        System.out.println( bytes );

        // 字节写入文件，实现复制
        File target = new File(root, "start2.bat");
        ByteSink byteSink = Files.asByteSink(target);
        byteSink.write(bytes);
    }

    @Test
    public void wrapper(){
        File dest = new File(root,"start.bat");
        // 作为字节读
        Files.asByteSource(dest);
        // 作为字节写
        Files.asByteSink(dest);

        // 作为字符读
        Files.asCharSource(dest, Charset.defaultCharset());
        // 作为字符写
        Files.asCharSink(dest, Charset.defaultCharset());
    }
```

### 其他

  > 管道流
  >> PipedOutputStream  PipedInputStream
  >> 实现多线程间的数据通信；
  >> 类似生产消费者模式
```
@Test
    public void pipe() throws IOException {
        PipedOutputStream pipedOutputStream = new PipedOutputStream();
        PipedInputStream pipedInputStream = new PipedInputStream();
        pipedOutputStream.connect(pipedInputStream);

        new Thread(()->{
            while (true){
                String date = new Date().toString();
                try {
                    pipedOutputStream.write( date.getBytes(StandardCharsets.UTF_8) );
                    pipedOutputStream.flush();
                    TimeUnit.SECONDS.sleep(2);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
        
        new Thread(()->{
            while (true){
                byte [] buff = new byte[1024];
                try {
                    int read = pipedInputStream.read(buff);
                    TimeUnit.SECONDS.sleep(4);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println( new String(buff) );
            }
        }).start();
    }
```


### 结束语

 在任何编程语言中，数据的IO都是比较常见并相当重要的。Guava作为工具型类库，主要是帮助开发者封装常用、重复的操作，开放出简介的API，不仅能让让代码更加整洁，
 同时对开发出稳健程序也是比不可少的。 




 