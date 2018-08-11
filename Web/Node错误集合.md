---
title: Node错误集合
tags: NodeJs
grammar_cjkRuby: true
---

## connect ETIMEDOUT
npm WARN registry Unexpected warning for https://registry.npmjs.org/: Miscellaneous Warning ETIMEDOUT: request to https://registry.npmjs.org/nrm failed, reason: connect ETIMEDOUT 104.16.22.35:443
npm WARN registry Using stale package data from https://registry.npmjs.org/ due to a request error during revalidation.
npm WARN deprecated coffee-script@1.7.1: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)

在git控制台安装nrm时一直报这个，等好久一直卡着。然后去cmd(以管理员模式运行)中执行了同样的语句，40多秒就安装成功了。

## node-gyp错误之旅