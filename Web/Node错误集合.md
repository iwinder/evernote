---
title: Node异常笔记
tags: NodeJs
grammar_cjkRuby: true
---


## 无法将“xxxx”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。
当修改npm全局路径时，windows下npm安装的模块执行可能会报如上错误。

解决方案： 将设置的prefix路径（如 F:\nodejs\node_global）添加到环境变量Path即可。

## connect ETIMEDOUT
npm WARN registry Unexpected warning for https://registry.npmjs.org/: Miscellaneous Warning ETIMEDOUT: request to https://registry.npmjs.org/nrm failed, reason: connect ETIMEDOUT 104.16.22.35:443
npm WARN registry Using stale package data from https://registry.npmjs.org/ due to a request error during revalidation.
npm WARN deprecated coffee-script@1.7.1: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)

在git控制台安装nrm时一直报这个，等好久一直卡着。然后去cmd(以管理员模式运行)中执行了同样的语句，40多秒就安装成功了。

## node-gyp错误之旅