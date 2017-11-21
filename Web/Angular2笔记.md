---
title: Angular2笔记
tags: Angular2
grammar_cjkRuby: true
---

alexa 后添加一个问号(?)表示可选字段。

```
export class Site {
  constructor(
    public id: number,
    public name: string,
    public url: string,
    public alexa?: number
  ) {  }
}
```
[Angular 2 表单](http://www.runoob.com/angularjs2/angularjs2-forms.html)