---
layout: post
title: 使用Java生成个性化的二维码
tagline: by feng
categories: Java基础
tags: 
    - feng
---


大家好，我是指北君。

指北君最近一直在思考一个问题，Java 能不能做一些比较有意思的事情，但是在网络上搜索的时候，有意思好玩的东西，都被 Python 给做了。Java 似乎就只剩下八股文，面试，框架，架构等等的内容。 

那为什么很少有人用 Java 做这些好玩的东西呢？在大家的固有观念里，Java 是比较笨重的，每次写代码必须要开启一个庞大的IDE来完成。但是也不妨碍我们拿 Java 出来玩一玩。

<!--more-->

今天指北君就先带大家做一个二维码玩玩。现在二维码在我们的生活中非常的常见。在我们的生活、交流、出行等等场景中，都用到了二维码。

### 准备工作

我们需要有一个生成二维码的依赖包，在Java中， zxing是目前使用最多的依赖包。

```xml
<!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.4.1</version>
</dependency>
```

在我们的项目中，我们通过maven的方式引入这个jar包。

### 生成二维码

简单版的二维码

```java 
@GetMapping("simple-qr")
public void createQrCode(String content, HttpServletResponse response) throws WriterException, IOException {
    Map<EncodeHintType, String> hints = new HashMap<>();
    hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
	BitMatrix bitMatrix = new QRCodeWriter().encode(content, BarcodeFormat.QR_CODE, 120, 120, hints);
    BufferedImage bufferedImage = new BufferedImage(bitMatrix.getWidth(), bitMatrix.getHeight(), BufferedImage.TYPE_INT_RGB);
    for (int i = 0; i < bitMatrix.getWidth(); i++) {
       for (int j = 0; j < bitMatrix.getHeight(); j++) {
           bufferedImage.setRGB(i, j, bitMatrix.get(i, j) ? FRONT_COLOR : BACKGROUND_COLOR);
        }
    }
    ImageIO.write(bufferedImage, "png", response.getOutputStream());
}
```

代码生成结果

![img](https://gitee.com/274904168/image-repo/raw/master/202111192350272.cn)

彩色的二维码

接下来，我们来生成彩色的二维码

```java
    @GetMapping("color-qr")
    public void createQrCode2(String content, HttpServletResponse response) throws WriterException, IOException {
        int width = 400, height = 400;
        Map<EncodeHintType, Object> hints = new HashMap<>();
        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.M);
        BitMatrix matrix = new QRCodeWriter().encode(content, BarcodeFormat.QR_CODE, width, height, hints);

        int[] pixels = new int[width * height];

        for (int y = 0; y < matrix.getHeight(); y++) {
            for (int x = 0; x < matrix.getWidth(); x++) {
                // 二维码颜色（RGB）
                int num1 = (int) (50 - (50.0 - 13.0) / matrix.getHeight()
                        * (y + 1));
                int num2 = (int) (165 - (165.0 - 72.0) / matrix.getHeight()
                        * (y + 1));
                int num3 = (int) (162 - (162.0 - 107.0)
                        / matrix.getHeight() * (y + 1));
                Color color = new Color(num1, num2, num3);
                int colorInt = color.getRGB();
                pixels[y * width + x] = matrix.get(x, y) ? colorInt : 16777215;
            }
        }

        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        image.getRaster().setDataElements(0, 0, width, height, pixels);
        ImageIO.write(image, "png", response.getOutputStream());
    }
```



![img](https://gitee.com/274904168/image-repo/raw/master/202111200001133.cn)

我们还可以自定义其他个性化的二维码，这里不再举例。

### 解析二维码

zxing 这个jar包，不仅仅可以用于生成二维码，还可以用于识别二维码。这个场景通常是在Android 开发中应用。指北君在这里也做一个简单的介绍。

```java
@GetMapping("decode-qr")
public String decodeQrcode(String fileName) throws IOException, NotFoundException {
    BufferedImage image = ImageIO.read(new File(fileName));
    LuminanceSource source = new BufferedImageLuminanceSource(image);
    Binarizer binarizer = new HybridBinarizer(source);
    BinaryBitmap binaryBitmap = new BinaryBitmap(binarizer);
    Map<DecodeHintType, Object> hints = new HashMap<DecodeHintType, Object>();
    hints.put(DecodeHintType.CHARACTER_SET, "UTF-8");
    Result result = new MultiFormatReader().decode(binaryBitmap, hints);
    return result.getText();
}
```
是不是感觉挺简单的呀。

### 总结

这次指北君主要演示了怎么使用zxing这个jar包，生成二维码和解析二维码的功能，更多个性化的功能，大家可以自己尝试哦。

