---
title: node-gyp错误之旅 
tags: Nodejs, node-gyp
grammar_cjkRuby: true
---

Node.js 在安装模块的时候一直报错，提示安装node-gyp时报python不可用的情况
```
ERR! configure error gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
```
查询得知需要安装python2.7，原有的python3不行（参考地址：http://blog.csdn.net/notejs/article/details/49681517 ，官方地址：https://github.com/nodejs/node-gyp  ）。更改后 该问题解决。
然后开始报：
```
在此解决方案中一次生成一个项目。若要启用并行生成，请添加“/m”开关。
MSBUILD : error MSB3428: 未能加载 Visual C++ 组件“VCBuild.exe”。要解决此问题，
1) 安装 .NET Fram
ework 2.0 SDK；2) 安装 Microsoft Visual Studio 2005；或 3) 如果将该组件安装到了
其他位置，请将其位置添加到系统
路径中。
```
但系统中vs2005是存在的，最后参考 https://github.com/nodejs/node-gyp#installation 文档使用 
```
npm install --global --production windows-build-tools
```
后解决。
