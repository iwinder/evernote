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


```
[wind@localhost certs]$ sudo openssl req -new -key ldapkey.pem -out ldapserver.csr
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

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:0932
An optional company name []:

```

问题：
```
5be101b1 daemon: TLS not supported (ldaps:///)
5be101b1 slapd stopped.
5be101b1 connections_destroy: nothing to destroy.
```
解决
```
#理论上只需要openssl与openssl-devel
yum install *openssl* -y
```

问题
```
plugin.c:33:18: 致命错误：ltdl.h：没有那个文件或目录
```
依次检测是否安装了libtool-ltdl和libtool-ltdl-devel：
```
sudo yum install libtool-ltdl -y
sudo yum install libtool-ltdl-devel -y
```



```
keytool -import  -alias ldapserver -file G:\Work\certs\ldapserver.crt -keystore cacerts.jks -keypass 123456 -storepass 123456
```


[OpenLDAP配置TLS加密传输](https://www.cnblogs.com/netonline/p/7517685.html)

[基于OpenSSL自建CA和颁发SSL证书](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)



[](http://forum.spring.io/forum/spring-projects/security/86987-spring-security-ldap-truststore-keystore)
```
LDAPS connections are just SSL so any connection issues are handled at the JRE level. There's nothing LDAP-specific required to enable support for SSL.
```