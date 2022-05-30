---
layout: post
title:  如何用Java播放音乐
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在本篇文章中，我们将学习如何用Java播放音乐。Java 声音 API 的设计是为了流畅和连续地播放声音，甚至是很长的声音。我们将使用 Java 提供的 `Clip` 和 `SourceDataLine` 声音API播放一个音频文件。

<!--more-->

### 播放声音的Java APIs

一般来说，`javax.sound` 包中的Java Sound APIs提供了两种播放音频的方法。在这两种方法之间，在如何指定声音文件数据方面有区别。Java Sound APIs可以以流式、缓冲方式和内存、非缓冲方式处理音频传输。Java的两个最著名的声音API是 `Clip` 和 `SourceDataLine`。

### Clip API

`Clip` API是Java的一个非缓冲或内存声音API。`Clip`类是`javax.sound.sampled`包的一部分，它在读取和播放短的声音文件时有用。在播放之前，整个音频文件被加载到内存中，用户可以完全控制播放。除了循环播放声音外，它还允许用户在一个随机的位置开始播放。

让我们首先创建一个示例类，`SoundPlayerWithClip`，它实现了`LineListener`接口，以便接收播放的线事件（`OPEN`、`CLOSE`、`START`和`STOP`）。我们将从`LineListener`实现`update()`方法来检查播放状态。

```java
public class SoundPlayerUsingClip implements LineListener {

    boolean isPlaybackCompleted;
    
    @Override
    public void update(LineEvent event) {
        if (LineEvent.Type.START == event.getType()) {
            System.out.println("Playback started.");
        } else if (LineEvent.Type.STOP == event.getType()) {
            isPlaybackCompleted = true;
            System.out.println("Playback completed.");
        }
    }
}
```

其次，让我们从我们项目的资源文件夹中读取音频文件。我们的资源文件夹包含三个不同格式的音频文件--即WAV、MP3和MPEG。

```java
InputStream inputStream = getClass().getClassLoader().getResourceAsStream(audioFilePath);
```

第三，从文件流中，我们将创建一个`AudioInputStream`。

```java
AudioInputStream audioStream = AudioSystem.getAudioInputStream(inputStream);
```

现在，我们将创建一个`DataLine.Info`对象。

```java
AudioFormat audioFormat = audioStream.getFormat();
DataLine.Info info = new DataLine.Info(Clip.class, audioFormat);
```

让我们从这个`DataLine.Info`创建一个`Clip`对象，并打开流，然后调用`start`来开始播放音频。

```java
Clip audioClip = (Clip) AudioSystem.getLine(info);
audioClip.addLineListener(this);
audioClip.open(audioStream);
audioClip.start();
```

最后，我们需要关闭任何开放的资源。

```java
audioClip.close();
audioStream.close();
```

一旦代码运行，音频文件就会播放。

由于音频被预装在内存中，我们有许多其他有用的API，我们可以从中受益。

我们可以使用`Clip.loop`方法来连续循环播放音频片段。

例如，我们可以把它设置为播放五次音频。

```java
audioClip.loop(4);    
```

或者，我们可以设置它无限期地播放音频（或直到中断）。

```java
audioClip.loop(Clip.LOOP_CONTINUUSLY);
```

`Clip.setMicrosecondPosition`设置媒体位置。当剪辑下次开始播放时，它将从这个位置开始。例如，要从第30秒开始，我们可以这样设置。

```java
audioClip.setMicrosecondPosition(30_000_000);
```

### SourceDataLine API

`SourceDataLine` API是java的一个缓冲或流式声音API。`SourceDataLine`类是`javax.sound.sampled`包的一部分，它可以播放无法预装到内存中的长声音文件。

当我们希望优化大的音频文件的内存时，或者在流传实时音频数据时，使用`SourceDataLine`更有效。如果我们事先不知道声音有多长，何时结束，它也很有用。

让我们首先创建一个示例类，从我们项目的资源文件夹中读取音频文件。我们的资源文件夹包含三个不同格式的音频文件--即WAV、MP3和MPEG。

```java
InputStream inputStream = getClass（）.getClassLoader（）.getResourceAsStream(audioFilePath);
```

第二，从文件输入流中，我们将创建一个`AudioInputStream`。

```java
AudioInputStream audioStream = AudioSystem.getAudioInputStream(inputStream);
```
现在，我们将创建一个`DataLine.Info`对象。

```java
AudioFormat audioFormat = audioStream.getFormat();
DataLine.Info info = new DataLine.Info(Clip.class, audioFormat);
```

让我们从这个`DataLine.Info`创建一个`SourceDataLine`对象，打开流，并调用`start`来开始播放音频。

```java
SourceDataLine sourceDataLine = (SourceDataLine) AudioSystem.getLine(info);
sourceDataLine.open(audioFormat);
sourceDataLine.start();
```

现在，在`SourceDataLine`的情况下，的音频数据是分块加载的，我们需要提供缓冲区的大小。

```java
private static final int BUFFER_SIZE = 4096;
```

现在，让我们从`AudioInputStream`读取音频数据，并将其发送到`SourceDataLine的`播放缓冲区，直到它到达流的末端。

```java
byte[] bufferBytes = new byte[BUFFER_SIZE];
int readBytes = -1;
while ((readBytes = audioStream.read(bufferBytes)) != -1) {
    sourceDataLine.write(bufferBytes, 0, readBytes);
}
```

最后，让我们关闭任何开放的资源。

```java
sourceDataLine.drain();
sourceDataLine.close();
audioStream.close();
```

一旦代码运行，音频文件就会播放。 在这里，我们不需要实现任何`LineListener`接口。

### `Clip`和`SourceDataLine`之间的比较

让我们来讨论一下两者的优点和缺点。

|Clip|SourceDataLine|
|---|---|
|支持从音频的任何位置播放。参见 `setMicrosecondPosition(long)` 或 `setFramePosition(int).`| 不能从声音中的任意位置开始播放。|
|支持在循环中播放（全部或部分的声音）。  参见 `setLoopPoints(int, int)` 和 `loop(int).`|不能播放（循环）全部或部分声音。|
|可以在播放前知道声音的持续时间。参见 `getFrameLength()` 或 `getMicrosecondLength().`|在播放前不能知道声音的持续时间。|
|可以在当前位置停止播放，稍后继续播放。请看 `stop()` 和 `start()` |不能在中间停止和恢复播放。|
|不适合播放大的音频文件，也没有效率，因为它是在内存中的。|适合播放长的声音文件或实时的声音流。|
|Clip的start()方法确实在播放声音，但它不会阻塞当前线程（它立即返回），所以它需要实现LineListener接口来了解播放状态。|与Clip不同，我们不需要实现LineListener接口来知道什么时候播放完成。|
|不可能控制什么声音数据被写入音频线的播放缓冲区。|可以控制哪些声音数据要被写入音频线的播放缓冲区。|

### Java API对MP3格式的支持

目前，`Clip`和`SourceDataLine`都可以播放AIFC、AIFF、AU、SND和WAV格式的音频文件。

我们可以使用`AudioSystem`检查支持的音频格式。

```java
    Type[] list = AudioSystem.getAudioFileTypes();
    StringBuilder supportedFormat = new StringBuilder("Supported formats:");
    for (Type type : list) {
        supportedFormat.append(", " + type.toString());
    }
    System.out.println(supportedFormat.toString());
```

然而，我们不能用Java Sound APIs `Clip` 和 `SourceDataLine` 播放流行的音频格式MP3/MPEG。`我们需要寻找一些能播放MP3格式的第三方库。

如果我们向 `Clip` 或 `SourceDataLine` API提供MP3格式的文件，我们会得到`UnsupportedAudioFileException` 。

```java
    javax.sound.sampled.UnsupportedAudioFileException: could not get audio input stream from input file
        at javax.sound.sampled.AudioSystem.getAudioInputStream(AudioSystem.java:1189)
```

### 总结

在这篇文章中，我们学习了如何用Java播放声音。我们还了解了两个不同的Java声音API - `Clip` 和 `SourceDataLine`。我们了解了 `Clip` 和 `SourceDataLine` API之间的区别。
