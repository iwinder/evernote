---
title: Ldaps 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

```
[wind@localhost certs]$ sudo openssl req -new -x509 -days 365 -key cakey.pem -out ca.crt
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:beijing
Organizational Unit Name (eg, section) []:beijing
Common Name (eg, your name or your server's hostname) []:192.168.137.129
Email Address []:windcoderz@foxmail.com
```