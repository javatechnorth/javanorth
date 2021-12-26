---
layout: post
title: HTML转换成PDF，这样就搞定了！
tagline: by 揽月中人
categories: PDF
tags:
- 揽月中人
---
哈喽，今天是一篇HTML to PDF速食指南。

Java 转换 HTML 到PDF有许多类库，今天我们介绍一下第三方免费的类库OpenPDF。

<!--more-->

### 1. OpenPDF

OpenPDF是免费的Java类库 ，遵从LGPL 和 MPL协议，所以基本上能够可以随意使用。OpenPDF是基于iTEXT的，目前来说也是维护的比较好的Java操作PDF的开源软件。

话不多说，且看所需要的依赖，

```xml
<dependency>    
    <groupId>org.jsoup</groupId>    
    <artifactId>jsoup</artifactId>   
    <version>1.13.1</version> 
</dependency>
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-core</artifactId>
    <version>1.0.6</version>
</dependency>
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-pdfbox</artifactId>
    <version>1.0.6</version>
</dependency>
```

jsoup可以将html文件转换成输入流等，也可以遍历html的DOM节点，提取元素及样式等。

### 2. 示例

本篇示例将以下html文件转换成pdf

```html
<html>
<head>
    <style>
        .center_div {
            border: 1px solid #404e94;
            margin-left: auto;
            margin-right: auto;
            background-color: #f6d0ed;
            text-align: left;
            padding: 8px;
        }
        table {
            width: 100%;
            border: 1px solid black;
        }
        th, td {
            border: 1px solid black;
        }
        body,html,input{font-family:"msyh";}
    </style>
</head>
<body>
<div class="center_div">
    <h1>Hello java North!</h1>
    <div>
        <p>convert html to pdf.</p>
    </div>
    <div>
        <table>
            <thead>
                <th>ROLE</th>
                <th>NAME</th>
                <th>TITLE</th>
            </thead>
            <tbody>
                <tr>
                    <td>MARKSMAN</td>
                    <td>ASHE</td>
                    <td>THE FROST ARCHER</td>
                </tr>
                <tr>
                    <td>MAGES</td>
                    <td>ANNIE</td>
                    <td>THE DARK CHILD</td>
                </tr>
                <tr>
                    <td>射手</td>
                    <td>凯塔琳</td>
                    <td>皮城女警</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
</body>
</html>
```



以上html用浏览器打开如下，乱码是因为中文字体不识别，下面转换的时候会加载对应的字体来进行转换。

![image-20211226231754213](https://www.javanorth.cn/assets/images/2021/lyj/html-to-pdf-1.png)



使用Java转换HTML到PDF代码如下：

```java
public class HtmlToPDFOpenSource {
    public static void main(String[] args) throws IOException {
        HtmlToPDFOpenSource htmlToPDFOpenSource = new HtmlToPDFOpenSource();
        htmlToPDFOpenSource.generatePdfByOpenhtmltopdf();
    }

    private  void generatePdfByOpenhtmltopdf() throws IOException {
        File inputHtml = new File("E:\\javaNorth\\java-study-note\\javaOpenSource\\src\\main\\resources\\test.html");

        //加载html文件
        Document document = Jsoup.parse(inputHtml, "UTF-8");
        document.outputSettings().syntax(Document.OutputSettings.Syntax.html);
        
        //引入资源目录，可以单独引入css，图片文件等
        String baseUri = FileSystems.getDefault()
            .getPath("javaOpenSource\\src\\main\\resources")
            .toUri().toString();
       
        try (OutputStream os = new FileOutputStream("javaOpenSource\\src\\main\\resources\\testOpenLeagueoflegends1.pdf")) {
            PdfRendererBuilder builder = new PdfRendererBuilder();
            builder.withUri("javaOpenSource\\src\\main\\resources\\testOpenLeagueoflegends1.pdf");
            builder.toStream(os);
            builder.withW3cDocument(new W3CDom().fromJsoup(document), baseUri);
            
            //引入指定字体，注意字体名需要和css样式中指定的字体名相同
            builder.useFont(new File("javaOpenSource\\src\\main\\resources\\fonts\\msyh.ttf"),"msyh",1,BaseRendererBuilder.FontStyle.NORMAL, true);
            builder.run();
        }
    }
}
```



使用Java代码转换成PDF如下（示例中使用了微软雅黑中文字体）：

![image-20211226232021451](https://www.javanorth.cn/assets/images/2021/lyj/html-to-pdf-2.png)



上述html文件中增加如下**外部样式**：

```html
<link href="style.css" rel="stylesheet">
```

并在resources目录下添加style.css文件，重新生成PDF文件如下。

![image-20211227005033972](https://www.javanorth.cn/assets/images/2021/lyj/html-to-pdf-3.png)



### 3. 总结

本片介绍了使用OpenPDF将html文件转换成PDF文件。同时也使用了自定义字体，外部样式。但是以下几点需要格外注意。

- Java代码中加载的字体名称要和HTML引用的CSS样式中的字体名相同 （{font-family:"msyh";}）。

- HTML文件标签节点必须闭合（<xxx></xxx> ）.否则解析会失败。

  

全部示例在此：https://github.com/javatechnorth/java-study-note/tree/master/javaOpenSource/src/main/java/pdf
