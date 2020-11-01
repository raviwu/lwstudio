+++
title = "InputStream 、 DataInputStream 與 BufferedInputStream"
date = 2020-09-11T13:46:57+08:00
publishdate = 2020-09-11T13:46:57+08:00
tags = ["coding", "java", "specification"]
draft = false

description = ""
summary = ""
keywords = []

[amp]
    elements = []

[author]
    name = "Ravi Wu"
    homepage = "https://raviwu.github.io"

[image]
    src = ""

[sitemap]
    changefreq = "monthly"
    priority = 0.5
    filename = "sitemap.xml"
+++

Java 有很多不同的 InputStream 類別，老是搞不清楚，`InputStream`、`DataInputStream`跟`BufferedInputStream`最近出現在我看的書的練習題裡，一些從 Java Doc (1.8) 文件中拉出來的相關連結：

*   [Closeable](https://docs.oracle.com/javase/8/docs/api/java/io/Closeable.html)
*   [FilterInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/FilterInputStream.html)
*   [InputStream](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html)
*   [DataInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/DataInputStream.html)
*   [BufferedInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/BufferedInputStream.html)

`InputStream`是抽象類別，實踐了`Closeable`，所以可以當成`try with resource`裡的資源。類別方法裡定義了`InputStream.nullInputStream()`可以產生空的`InputStream`。另外也規範所有實作子類別必須提供`public int read() throws IOException`方法來回傳下一個 byte 的內容。

`FilterInputStream`在生成時吃進`InputStream`後把`InputStream`存起來，並且覆寫了所有`InputStream`裡的方法，大部分覆寫的方法都是把原來的操作轉到物件生成時丟進去的`InputStream`上，以供更細分的子類別使用，例如：

```java
public int read(byte b[], int off, int len) throws IOException {
    return in.read(b, off, len);
}
```

`DataInputStream`跟`BufferedInputStream`都進一步繼承`FilterInputStream`。`DataInputStream`主要是從`InputStream`直接讀入 Java 的 primitive data type。

`BufferedInputStream`在內部會產生一個緩衝陣列（buffer array）來支援`mark`跟`reset`方法，透過額外使用的緩衝空間來先讀入資料，以優化資料讀入的效能。