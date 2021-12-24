---
layout: post
title:  java 对图片加文字和图片水印--20211214
tagline: by 某某白米饭
categories: java
tags: 
    - 某某白米饭
---

在项目中经常有需要在图片上添加水印的需求以及在某些场合下需要身份证图片，这时就可以对身份证上加水印防止被用于其他用途，java 处理图片水印，不需要额外的第三方包，BufferedImage 和 Graphics2D 就可以搞定。

<!--more-->

### 读取图片

读取图片非常简单，使用 ImageIO 读取 file 文件就行了。

```java
File imageFile = new File("img.png");
Image src = ImageIO.read(imageFile);
int width = src.getWidth(null);
int height = src.getHeight(null);
```

### 添加水印

Image 类是一个抽象类，无法被直接创建，我们可以使用 BufferedImage 读取缓存中的图像数据。Graphics2D 类继承于 Graphics 类，Graphics2D 类是 java 渲染文字及图片的基础类，提供了对绘制、填充、旋转和定义颜色的支持。

```java
//创建指定大小，指定图像类型的 BufferedImage 对象
BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
//创建 Graphics2D 对象
Graphics2D graphics2D = bufferedImage.createGraphics();
graphics2D.drawImage(src, 0, 0, width, height, null);
```

接下来就是创建水印的字体、颜色了，如果水印文字在右边的话，我们需要知道文字的长度。

```java
//设置字体和颜色
graphics2D.setColor(Color.lightGray);

Font font = new Font("宋体", Font.PLAIN, 20);
graphics2D.setFont(font);
//获取文字长度
FontMetrics fontMetrics = graphics2D.getFontMetrics(font);
int len = fontMetrics.stringWidth("这里是水印");
graphics2D.drawString("这里是水印", width - len - 10, height - 10);
graphics2D.dispose();
```

### 保存图片

最后使用 FileOutputStream 和 ImageIO.write() 保存图片。

```java
try(FileOutputStream outputStream = new FileOutputStream("0.png")) {
    ImageIO.write(bufferedImage, "png", outputStream);
}
```

效果：

![](https://files.mdnice.com/user/15960/172d7fe7-ec71-4438-bc63-7096334d611f.png)

### 添加图片水印

添加图片水印更是简单，不再需要设置字体和颜色，直接使用 graphics2D.drawImage() 方法。

```java
File waterMarkFile = new File("E:\\pdfProject\\src\\main\\java\\waterMark.png");
Image waterMarkImg = ImageIO.read(waterMarkFile);
int waterMarkWidth = waterMarkImg.getWidth(null);
int waterMarkHeight = waterMarkImg.getHeight(null);
graphics2D.drawImage(waterMarkImg,width - waterMarkWidth - 10, height - waterMarkHeight - 10,waterMarkWidth, waterMarkHeight, null);
try(FileOutputStream outputStream = new FileOutputStream("1.png")) {
    ImageIO.write(bufferedImage, "png", outputStream);
}
```

效果：

![](https://files.mdnice.com/user/15960/0ae2066e-1aeb-4604-9dea-bc7a51156115.png)

### 总结

BufferedImage 和 Graphics2D 还可以做其他事情，比如对表情包添加文字等。大家都可以去试试，做出自己的实用工具库。
