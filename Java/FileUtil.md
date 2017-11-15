---
title: FileUtil
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

## 获取文件扩展名
```
	private String getExt(String filePath) {
		return filePath.substring(filePath.lastIndexOf("."));
	}
```

## 获取文件名称-不带扩展名
```
private String getFileNameNoExt(String filePath) {
		return filePath.substring(0,filePath.lastIndexOf("."));
	}
```