---
title: 获取txt文档编码格式
date: 2019-09-19 16:12:39
tags:
    - Java 
categories:
    - Java
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 
src="//music.163.com/outchain/player?type=2&id=28708061&auto=1&height=66"></iframe>

{% blockquote %}
网上好多不完全，总有个别txt的编码获取不到。
用这段代码，暂时没有发现问题。

它在我的机器上可以很好运行！  -------大部分程序员
{% endblockquote %}

##### 部分TXT识别不了的版本
```java
    InputStream inputStream = new FileInputStream("g62.txt");
    byte[] head = new byte[3];
    inputStream.read(head);
    String code = "GBK";
    if (head[0] == -1  &&  head[1] == -2){
        code = "UTF-16";
    }else if (head[0] == -2  &&  head[1] == -1){
        code = "Unicode";
    }else if (head[0] == -17  && head[1] == -69 && head[2] == -65){
        code = "UTF-8";
    }
    inputStream.close();

```
```java
    BufferedInputStream bin = new BufferedInputStream(new FileInputStream("62.txt"));
    int p = (bin.read() << 8) + bin.read();
    
    String code = null;
    
    switch (p) {
        case 0xefbb:
            code = "UTF-8";
            break;
        case 0xfffe:
            code = "Unicode";
            break;
        case 0xfeff:
            code = "UTF-16BE";
            break;
        default:
            code = "GBK";
    }
    bin.close();
```

##### 未发现问题版本
```Java
    private static  String getFileCharset(File sourceFile) {
        String charset = "GBK";
        byte[] first3Bytes = new byte[3];
        try {
            boolean checked = false;
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream(sourceFile));
            bis.mark(0);
            int read = bis.read(first3Bytes, 0, 3);
            if (read == -1) {
                return charset; //文件编码为 ANSI
            } else if (first3Bytes[0] == (byte) 0xFF
                    && first3Bytes[1] == (byte) 0xFE) {
                charset = "UTF-16LE"; //文件编码为 Unicode
                checked = true;
            } else if (first3Bytes[0] == (byte) 0xFE
                    && first3Bytes[1] == (byte) 0xFF) {
                charset = "UTF-16BE"; //文件编码为 Unicode big endian
                checked = true;
            } else if (first3Bytes[0] == (byte) 0xEF
                    && first3Bytes[1] == (byte) 0xBB
                    && first3Bytes[2] == (byte) 0xBF) {
                charset = "UTF-8"; //文件编码为 UTF-8
                checked = true;
            }
            bis.reset();
            if (!checked) {
                int loc = 0;
                while ((read = bis.read()) != -1) {
                    loc++;
                    if (read >= 0xF0)
                        break;
                    if (0x80 <= read && read <= 0xBF) // 单独出现BF以下的，也算是GBK
                        break;
                    if (0xC0 <= read && read <= 0xDF) {
                        read = bis.read();
                        if (0x80 <= read && read <= 0xBF) // 双字节 (0xC0 - 0xDF)
                            // (0x80
                            // - 0xBF),也可能在GB编码内
                            continue;
                        else
                            break;
                    } else if (0xE0 <= read && read <= 0xEF) {// 也有可能出错，但是几率较小
                        read = bis.read();
                        if (0x80 <= read && read <= 0xBF) {
                            read = bis.read();
                            if (0x80 <= read && read <= 0xBF) {
                                charset = "UTF-8";
                                break;
                            } else
                                break;
                        } else
                            break;
                    }
                }
            }
            bis.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return charset;
    }
```
