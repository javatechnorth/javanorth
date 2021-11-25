---
layout: post
title: JavaCV 实战：使用图片制作字符画
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

最近指北君在做一些图像处理的工作，主要是使用到了OpenCV。可能有的小伙伴听过OpenCV，OpenCV是通过C++开发的，官方只提供了C++、Python、JS 等版本的API。

Java 使用OpenCV 原生的库，比较麻烦，需要配置一些环境变量。指北君在GitHub上找了一圈，终于找到了一个Java版本的项目 -- JavaCV ，JavaCV 直接把OpenCV给嵌入到内部，不再需要其他的环境变量的支持。JavaCV另外包含了FFmpeg、Tesseract等一系列的音视频相关的库。

今天指北君就要带大家一起使用 JavaCV 将一张图片转换成一副字符画。

<!--more-->

### 准备工作

我们需要引入 JavaCV的依赖库

```xml
  <dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.6</version>
  </dependency>
```

另外，我们还需要准备一个图片

![](https://www.javanorth.cn/assets/images/2021/feng/shapes.png)

接下来我们就可以着手写代码了。

先使用opencv读取图片

```java
String path = "your path";
Mat img = opencv_imgcodecs.imread(path + "cards.jpg");
```

由于图片的宽高太大，做字符画不太好， 所以需要进行缩放。

```java
opencv_imgproc.resize(img, img,new Size(), 0.3, 0.3, opencv_imgproc.CV_INTER_LINEAR);
```

接下来，我们需要把图片转化成灰度图片

```java
Mat imgGray = new Mat(img.size());
opencv_imgproc.cvtColor(img, imgGray, opencv_imgproc.COLOR_BGR2GRAY);
```

我们来看下，灰度图片是怎么样的

![](https://www.javanorth.cn/assets/images/2021/feng/shapes-gray.png)

```java
UByteIndexer indexer = (UByteIndexer) imgGray.createIndexer();
int[]  bgr = new int[3];
for (long i = 0; i < indexer.rows() -1; i++) {
    for (long j = 0; j <indexer.cols()-1; j++) {
        indexer.get(i,j, bgr);
        int gray = (bgr[0] + bgr[1] + bgr[2]) %  charStr.length() ;
        strImage.append(String.valueOf(charStr.charAt(gray)));
    }
    strImage.append("\n");
}
```

通过获取灰度图片，每个点位的bgr颜色，然后根据颜色值转换成对应的字符，拼接形成一个完整的字符画。

![](https://www.javanorth.cn/assets/images/2021/feng/shapes-char.png)

好了，我们这次制作字符画就结束了。 完整的代码如下：

```java
public class App 
{
    private static final String path = "your path";

    private static final String charStr = " .,-'`:!1+*abcdefghijklmnopqrstuvwxyz<>()\\/{}[]?234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ%&@#$";

    public static void main( String[] args ) throws IOException {
        Mat img = opencv_imgcodecs.imread(path + "shapes.png");

        opencv_imgproc.resize(img, img,new Size(), 0.3, 0.3, opencv_imgproc.CV_INTER_LINEAR);


        Mat imgGray = new Mat(img.size());
        opencv_imgproc.cvtColor(img, imgGray, opencv_imgproc.COLOR_BGR2GRAY);
        //opencv_imgproc.threshold(imgGray, imgGray, 127, 255, opencv_imgproc.CV_THRESH_BINARY);
        opencv_highgui.imshow("gray", imgGray);
        StringBuilder strImage = new StringBuilder();
        opencv_highgui.waitKey(0);
        UByteIndexer indexer = (UByteIndexer) imgGray.createIndexer();
        int[]  bgr = new int[3];
        for (long i = 0; i < indexer.rows() -1; i++) {
            for (long j = 0; j <indexer.cols()-1; j++) {
                indexer.get(i,j, bgr);
                int gray = (bgr[0] + bgr[1] + bgr[2]) %  charStr.length() ;
                strImage.append(String.valueOf(charStr.charAt(gray)));
            }
            strImage.append("\n");
        }
        FileOutputStream fileOutputStream = new FileOutputStream(path + "file.txt");
        fileOutputStream.write(strImage.toString().getBytes());
        fileOutputStream.close();
    }
}
```

### 总结

今天指北君给大家展示了如何使用JavaCV制作一幅字符画。JavaCV可以做到事情还有很多很多，后面将持续更新。