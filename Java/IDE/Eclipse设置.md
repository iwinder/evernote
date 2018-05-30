---
title: Eclipse设置
tags: Java,IDE,Eclipse
grammar_cjkRuby: true
---

## 直接打开类文件/文件夹所在的本地目录

1、直接打开类文件/文件夹所在的本地目录
2、双击Program，新建一个Program。
3、设置新建的Program

location 里面填 ：C:\WINDOWS\explorer.exe 

Arguments 里面填: ${container_loc}

[Eclipse直接打开类文件/文件夹所在的本地目录](http://blog.csdn.net/rogers65/article/details/52982436)

[Eclipse中一键打开文件所在文件夹](http://blog.csdn.net/duanyipeng/article/details/7065085)

## 设置编码

### 整个工作空间

Window ->Preference->Genera-->Workspace
Text file encoding 设为 UTF-8，
new text file line delimiter 设为 Unix

### Properties 文件
Window ->Preference->Genera-->Content Types-->Text -->Java Properties File

设置Default encoding，把下面的ISO-8859-1改为UTF-8,然后update,确认保存退出。


[Eclipse如何设置编码格式？（3种方式）](http://blog.csdn.net/rainy_black_dog/article/details/52403735)

## 调整字体大小

### 控制台 

Window -> Preferences -> General -> Appearance -> Colors and Fonts -> Basic -> Text Font -> Edit 

### 调节主窗口字体大小。
 Window -> Preferences -> General -> Appearance -> Colors and Fonts -> Java -> Java Editor Text Font -> Edit
 
 [调整Eclipse字体大小](http://blog.csdn.net/magi1201/article/details/45921907)