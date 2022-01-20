---
layout: post
title: JavaCV 实现像素画马赛克功能
tagline: by feng
categories: Java基础
tags: 
    - feng
---

大家好，我是指北君。

指北君最近在逛一些社区的时候发现了有很多图片是像素图，感觉挺好玩的。正巧最近自己在学习OpenCV，所以在这里给大家演示一下如何使用OpenCV来处理像素图。

像素图其实有点类似于类似于打马赛克的功能。通过像素的变化，演示一个像素画的功能。像素画在 NFT 中特别的流行。

<!--more-->

### 准备工作

我们先引入 JavaCV 的依赖库 

```xml
  <dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.6</version>
  </dependency>
```

这种方式引入，会把javacv 所有包含的都引入进来。平时测试使用都时候，可以这样操作。但是到真实项目中，还是需要做一次精简才行。

另外我们准备了一个图片。

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b1a025e1d3f4853b5babda16ac19b6a.png)

### 代码实现

读取文件

```kotlin
    val path ="path/to/img/"
    val img = opencv_imgcodecs.imread(path + "meinv.jpeg")
```

获取原图的像素宽高，然后进行像素比缩放。

```kotlin
    val size = img.size()
    val height = size.height()
    val width = size.width()
    
    val pixelSize = 10
    
    val newWidth = width / pixelSize
    val newHeight = height / pixelSize
```

根据设定的像素比， 对原图进行缩小，再放大的两次 resize 操作。这样就完成了像素图的处理。

```kotlin
    val imgTmp: Mat? = null
    opencv_imgproc.resize(img, imgTmp, Size(newWidth, newHeight),0.0, 0.0, opencv_imgproc.INTER_NEAREST)
    opencv_imgproc.resize(img, imgTmp, Size(width, height),0.0, 0.0, opencv_imgproc.INTER_NEAREST)
```

那我们来看下处理过之后的图像效果吧

![在这里插入图片描述](https://img-blog.csdnimg.cn/109886008ada404e892141373ace2bf3.png)

效果看起来还可以，图片颜色单一图片尺寸稍微小一些的效果会好很多。

### 完整代码

```kotlin
import org.bytedeco.opencv.global.opencv_highgui
import org.bytedeco.opencv.global.opencv_imgcodecs
import org.bytedeco.opencv.global.opencv_imgproc
import org.bytedeco.opencv.opencv_core.Mat
import org.bytedeco.opencv.opencv_core.Size

fun main(args: Array<String>) {

    val path = "path/to/img/"
    val img = opencv_imgcodecs.imread(path + "meinv.jpeg")

    val size = img.size()
    val height = size.height()
    val width = size.width()

    val pixelSize = 10

    val newWidth = width / pixelSize
    val newHeight = height / pixelSize

    val imgTmp: Mat? = null
    opencv_imgproc.resize(img, imgTmp, Size(newWidth, newHeight), 0.0, 0.0, opencv_imgproc.INTER_NEAREST)
    opencv_imgproc.resize(img, imgTmp, Size(width, height), 0.0, 0.0, opencv_imgproc.INTER_NEAREST)


    opencv_highgui.imshow("meinv", img);
    opencv_highgui.waitKey(0)

}
```

### 总结

像素图处理的演示到这里就结束了，opencv还能做更多好玩的事情，后面陆陆续续更新一些好玩的图片处理功能。
