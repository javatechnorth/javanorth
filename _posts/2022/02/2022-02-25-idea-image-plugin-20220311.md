---
layout: post
title:  做了个图片压缩的插件
tagline: by feng
categories: idea
tags: 
    - feng
---

大家好，我是指北君。
​

上次指北君给大家介绍了如何从零开始开发一个 idea 插件。其他这个插件开发并不局限于idea。JetBrains 其他的IDE 也都是通用的。 但是用于开发 .net 的Rider 是另外的。
​
<!--more-->

今天指北君主要是想给大家介绍最近自己开发的图片压缩插件，该插件用到了 tinypng 的在线服务。 大家可能都知道，tinypng提供的图片压缩服务特别好用，压缩比例非常大，但是对图片质量影响比较小。
​

上次指北君已经教大家怎么使用开发了，这次创建项目之类的流程不再赘述。直接来看看代码就好了。
### 项目依赖
在这里我们需要添加 tinypng 的 API 依赖

```kotlin
implementation("com.tinify:tinify:latest.release")
```
### 核心实现
在这个 idea 插件项目中，指北君这边提供了一个 dialog 的方式来展示，这一部分指北君通过 Java Swing 的方式来展现。 以前听到swing 总感觉头大，感觉很复杂，但是在实际开发过程中，idea 在 Java Swing 的开发上，做了很大的改进。全程可以靠UI设计器搞定， 出来的UI也是挺好看的。
​

![image.png](https://img-blog.csdnimg.cn/img_convert/61da530cda15f0eb4034869cd229c64d.png)


Jetbrains 在Java UI 开发上真的下了不少的功夫。 通过 GridLayout 进行布局，全程拖控件， 有种回到当年做 .net winform 开发的感觉。
​

项目是通过 kotlin 进行开发的，涉及到了kotlin的一些函数扩展方法等特性。对于Java 的朋友可能有一些不太适应，但是看懂不难哦。
​

#### AnAction
在idea中每一个操作都是一个AnAction 对象，所以我们这里也需要创建我们自己的AnAction.
```kotlin
class ImageCompressionAction : AnAction() {

    override fun actionPerformed(e: AnActionEvent) {
        checkApiKeyFile(notExistAction = {
            popupInputKeyDialog(event = e)
        }, existAction = { apiKey ->
            setTinyPNGApiKey(apiKey)
            popupCompressDialog(event = e)
        })
    }

    private fun popupInputKeyDialog(event: AnActionEvent?) {
        InputKeyDialog(object : InputKeyDialog.DialogCallback {
            override fun onOkBtnClicked(tinyPngKey: String) = checkApiKeyValid(project = getEventProject(event), apiKey = tinyPngKey, validAction = {
                updateExpireApiKey(apiKey = tinyPngKey)
                popupCompressDialog(event)
            }, invalidAction = {
                popupInputKeyDialog(event)
            })
            override fun onCancelBtnClicked() {
                TODO("Not yet implemented")
            }

        }).showDialog(width = 530, height = 150, isInCenter = true, isResizable = false)
    }

    private fun popupCompressDialog(event: AnActionEvent?) {
        ImageCompressionDialog(readUsedDirs(), readUsedFilePrefix(), object : DialogCallback {
            override fun onOkClicked(model: ImageCompressionModel) {
                saveUsedDirs(model)
                saveUsedFilePrefix(model.filePrefix)
                val inputFiles = readInputDirFiles(model.inputDir)
                val startTime = System.currentTimeMillis()
                compressImage(
                    project = getEventProject(event),
                    inputFiles = inputFiles,
                    model = model,
                    successAction = {
                        Messages.showWarningDialog(
                            "压缩完成, 已压缩: ${inputFiles.size}张图片, 压缩总时长共计: ${(System.currentTimeMillis() - startTime) / 1000}s",
                            "来自ImageCompression提示"
                        )
                    },
                    failAction = {
                        popupInputKeyDialog(event = event)
                    })
            }

            override fun onCancelClicked() {
                TODO("Not yet implemented")
            }

        }).showDialog(width = 500, height = 200, isInCenter = true, isResizable = false)
    }
}
```
在这个AnAction 中我们主要做了一下几件事情：

- 检查 tinypng api key 是否有效
- 检查 弹出图片压缩界面
- 填充图片压缩界面的历史数据
  
#### 前端界面
主要又两个界面，一个是输入APIKey的界面，另外一个是选择图片路径进行压缩的界面。


1、API Key 输入界面，这个界面做的比较简单，只是简单的输入 TinyPng 网站申请过来的 key ，对key 进行保存。

![image.png](https://img-blog.csdnimg.cn/img_convert/f43aa55f48faf1dd384a94eff99b29f6.png)

2、 选择图片路径的界面， 选择图片的源目录，和输出目录， 并且可以设置输出文件的前缀名。

![image.png](https://img-blog.csdnimg.cn/img_convert/9eb8a78b6b9deda88da21be0ee260e44.png)

### 插件最终结果

ui 做的比较简单，选择源目录，输出目录，就可以了。 可以选择文件之前和后缀，

![image.png](https://img-blog.csdnimg.cn/img_convert/c004d79828a8f98bf55f796ce7150959.png)

### 总结

今天指北君给大家展示了新做的图片压缩插件， 插件还在继续完善中，感兴趣的朋友可以继续关注。
