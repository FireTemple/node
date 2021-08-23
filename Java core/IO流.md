# IO流

## 1. 简介

从内存（程序）到文件（txt，json，数据库）的传输

### 1.1 主要流程

* 读取文件
* 创建流
* 读写文件
* 关闭流

## 2. 分类

### 2.1 数据单位

* 字节流 8 bits  图片 视频 非文本
* 字符流 文本文件

### 2.2 角色

* 节点流
* 处理流

### 2.3 抽象基类

![image-20200507181004867](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200507181004867.png)

**规律： inputOutput 类型的一般都是字节流，字符流一般含有reader or writer**

## 3. 简单案例（节点流）

### 3.1 输入

```java
package com.bohan;

import org.junit.Test;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;

public class FileReaderWriterTest {

    @Test
    public void testFileReader() throws IOException {
        // 相较于module 如果用main方法会相较于项目
        // 1.实例化file对象
        File file = new File("hello.txt");
        // 2.提供具体的流
        FileReader fileReader = new FileReader(file);

        // 3. 数据的读入 读一个字符并且指针后移
        int data = fileReader.read();
        while(data != -1){
            System.out.println((char) data);
            data = fileReader.read();
        }

        // 4. 关闭资源
        fileReader.close();

    }
}
```

#### 3.1.1 注意点

* 一般用try catch finnally来写，因为如果报异常 最后无法关闭，那么会导致资源浪费。

#### 3.1.2 标准写法

使用数组来存放 避免如3.1中的情况一个一个读

```java
@Test
public void testFileReader1() throws IOException {
    
   FileReader fileReader = null;
   try {
       File file = new File("hello.txt");

       fileReader = new FileReader(file);
        
       // 使用数组来保存
       char[] cbuf = new char[13];
       int len;
       
       while((len = fileReader.read(cbuf)) != -1){
           String str = new String(cbuf, 0 ,len);
           System.out.println(str);
       }
   } catch (IOException e){
       e.printStackTrace();
   } finally {
       if (fileReader != null){
           fileReader.close();
       }
   }

}
```

### 3.2 输出

```java
 @Test
    public void testFileWriter() throws IOException{
        FileWriter fileWriter = null;
        try {
            // 输出操作如果不存在会自动创建 如果存在会自动覆盖
            File file = new File("hello1.txt");

            fileWriter = new FileWriter(file);
            // 加上true则不覆盖
//        FileWriter fileWriter = new FileWriter(file, true);

            fileWriter.write("I have a dream !");
            fileWriter.write("you need to have a dream !");
          
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileWriter != null)
            fileWriter.close();
        }
        
    }
```

## 4. 文件复制

https://www.bilibili.com/video/BV1ab411p7VU?p=580

### 4.1 例子

这里使用的是图片复制 用的是 字节流

```java
@Test
public void testFileInputOutputStream() throws FileNotFoundException {

    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
        File file = new File("WX20200508-111724.png");
        File file1 = new File("copy.png");

        fis = new FileInputStream(file);
        fos = new FileOutputStream(file1);

        // 复制过程

        byte[] buffer = new byte[5];
        int len;
        while((len = fis.read(buffer)) != -1){
            fos.write(buffer, 0, len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fos != null) {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fis != null){
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


}
```

## 5. 缓冲流

* **处理流**之一，用来**加速读写效率**, 用法基本一样 **只是加了一层**，只做了字节流的例子，字符流一样。
* 作用在已有流之上

### 5.1 例子

```java
 @Test
    public void testBuffer() throws FileNotFoundException {

        FileInputStream fis = null;
        FileOutputStream fos = null;
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;
        try {
            File srcFile = new File("content.png");
            File desFile = new File("testBuffer.png");

            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(desFile);

            // 加一层
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);

            byte[] buffer = new byte[10];
            int len;

            while((len = bis.read(buffer)) != -1){
                bos.write(buffer, 0 , len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bis != null) {
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bos != null) {
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        // 从外到内关闭 关闭外层 会自动关闭内层 所以只关闭内层
//        fis.close();
//        fos.close();


    }
```

### 5.2 原理

调用了 fileinputstring里面的带缓存的构造方法，选择用了一个更大的缓冲区，并且拥有一个flush来清空缓存区从而提高效率

## 6. 转换流

**处理流**之一 **属于字符流**

#### 6.1 InputStreamReader/OutPutStringReader

* InputStreamReader 将一个字节的输入流转换为字符的输入流
* OutPutStringReader 讲一个字符的输出流转换为字节的输入流

#### 6.2 例子

```java
@Test
public void testTra() throws IOException {
  
  /*
  读取字节流的信息
  */
  
    FileInputStream fis = new FileInputStream("hello.txt");
    // 创建流 默认使用系统默认的字符集
    InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
    char[] cbuf = new char[20];
    int len;
    while ((len = isr.read(cbuf)) != -1){
        String str = new String(cbuf, 0,len);
        System.out.println(str);
    }

    isr.close();
}
```