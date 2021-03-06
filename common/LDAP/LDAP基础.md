---
title: LDAP基础
tags: LDAP
grammar_cjkRuby: true
---
 
 LDAP信息模型是基于条目的. 一个条目是一个属性的集合，有一个全球唯一的识别名（ DN ）. DN用于明白无误地标识条目. 每个条目的属性有一个类型和一个或多个值. 该类型通常是可记忆的字符串，如“ cn ”就是标识通用名称，或“电子邮件”就是电子邮件地址。
 
 值的语法依赖于属性类型. 例如, 一个 cn 属性可以包含一个值 Babs Jensen. 一个 mail 属性可以包含值"babs@example.com". 一个 jpegPhoto 属性将包含一个JPEG (binary) 格式的照片.
 
##  LDAP简称对应
|简称|描述|
|---|---|
|o| organization（组织-公司）|
|ou|organization unit（组织单-部门）|
|c | countryName（国家）|
|dc | domainComponent（域名）|
|sn | suer name（真实名称）|
|cn | common name（常用名称）|
 
### 条目(Entry)
![enter description here](./images/1539764758166.png)
条目，也叫记录项，是LDAP中最基本的颗粒，每一个条目都有一个唯一的标识名（Distinguished Name, DN）。如上图中的 'cn=baby,ou=marketing,ou=people,dc=mydomain,dc=org'。


DN 中有三个组成：

- 相对标识（Relational DN, RDN）：一般是 DN 中最左边的部分，上例中的 cn=baby。
- 组织机构（Organization Unit, OU）：标记条目所属的组织，上例中的 ou=marketing,ou=people。
- 基准 DN（Base DN）：LDAP 目录树的根，上例中的 dc=mydomain,dc=org。

Base DN 有三种命名方式，可以按需选择：

- X.500 格式：形如 dc="Foobar, Inc",dc=US。
- 以 internet 地址表示：形如 dc=foobar.com。
- 以 DNS 域名的不同部分来表示（推荐）：形如 dc=foobar,dc=com。


### 属性（Attribute）

### 对象类（ObjectClass）
对象类是属性的集合，项目可以通过选择对象类来获得属性。如果两个对象类中有相同的属性，条目只会保存其中一个。

一共有三种对象类：

- 结构类型（STRUCTURAL）

	结构类是是最基本的类型

	每个条目属于且仅属于一个结构型对象类

- 抽象类型（ABSTRACT）
	抽象类可以被继承，是其他类型的模板
	条目不能直接继承抽象类
	top 对象类是所有抽象类的根类
	
- 辅助类型（AUXILIARY）
	辅助类型类规定了对象实体的扩展属性
	每个条目至少有一个结构性对象类
![enter description here](./images/1539767564354.png)

### 模式（Schema）
模式是对象类的集合。

对象类（ObjectClass）、属性类型（AttributeType）、语法（Syntax）分别约定了条目、属性、值，他们之间的关系如下图所示。所以这些构成了模式(Schema)——对象类的集合。条目数据在导入时通常需要接受模式检查，它确保了目录中所有的条目数据结构都是一致的。
![enter description here](./images/1539767544339.png)


[LDAP 笔记](https://blog.laisky.com/p/ldap/)

[LDAP服务器的概念和原理简单介绍](https://segmentfault.com/a/1190000002607140#articleHeader7)

[OpenLDAP2.4管理员指南](	http://wiki.jabbercn.org/index.php/OpenLDAP2.4%E7%AE%A1%E7%90%86%E5%91%98%E6%8C%87%E5%8D%97#.E4.BB.80.E4.B9.88.E6.98.AFLDAP.3F)




### 
```
/usr/local/etc/openldap/

sudo /usr/local/libexec/slapd


 ps -ef |grep slapd

ldapadd -x -D "cn=admin,dc=windcoderc,dc=com" -W -f test1.ldif

ldapsearch -x -b 'dc=windcoder,dc=com' '(objectClass=*)'



```



```
dn: dc=windcoder,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
dc: windcoder
o: windcoder com

dn: ou=Orgs,dc=windcoder,dc=com
objectClass: top
objectClass: organizationalUnit
ou: Orgs


dn: cn=Some Person,ou=Orgs,dc=windcoder,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
userPassword: password
cn: Some Person
sn: Person
description: Orgs  Some Person
telephoneNumber: 555-123456


```

最开始

```
dn: cn=Some Person,ou=Orgs,dc=windcoder,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
uid: some-person
userPassword: password
cn: Some Person
sn: Person
description: Orgs  Some Person
telephoneNumber: 555-123456
```

[LDAP安装配置(原创)](http://czmmiao.iteye.com/blog/1561703)


语法错误：
```
additional info: objectClass: value #1 invalid per syntax
```
未识别的objectClass。



```
#
# See slapd.conf(5) for details on configuration options.
# This file should NOT be world readable.
#
include         /usr/local/etc/openldap/schema/core.schema

# Define global ACLs to disable default read access.

# Do not enable referrals until AFTER you have a working directory
# service AND an understanding of referrals.
#referral       ldap://root.openldap.org

pidfile         /usr/local/var/run/slapd.pid
argsfile        /usr/local/var/run/slapd.args

# Load dynamic backend modules:
# modulepath    /usr/local/libexec/openldap
# moduleload    back_mdb.la
# moduleload    back_ldap.la

# Sample security restrictions
#       Require integrity protection (prevent hijacking)
#       Require 112-bit (3DES or better) encryption for updates
#       Require 63-bit encryption for simple bind
# security ssf=1 update_ssf=112 simple_bind=64

# Sample access control policy:
#       Root DSE: allow anyone to read it
#       Subschema (sub)entry DSE: allow anyone to read it
#       Other DSEs:
#               Allow self write access
#               Allow authenticated users read access
#               Allow anonymous users to authenticate
#       Directives needed to implement policy:
# access to dn.base="" by * read
# access to dn.base="cn=Subschema" by * read
# access to *
#       by self write
#       by users read
#       by anonymous auth
#
# if no access controls are present, the default policy
# allows anyone and everyone to read anything but restricts
# updates to rootdn.  (e.g., "access to * by * read")
#
# rootdn can always read and write EVERYTHING!

loglevel    256
logfile    /usr/local/etc/openldap/logs/slapd.log

#######################################################################
# MDB database definitions
#######################################################################

database        mdb
maxsize         1073741824
suffix          "dc=windcoder,dc=com"
# rootdn                "cn=Manager,dc=windcoder,dc=com"
rootdn          "cn=admin,dc=windcoder,dc=com"
# Cleartext passwords, especially for the rootdn, should
# be avoid.  See slappasswd(8) and slapd.conf(5) for details.
# Use of strong authentication encouraged.
#rootpw         secret
rootpw  {SSHA}LFvNxLuy20L00BudQ8MYgv8ZdxRSXNxd
#rootpw 123456
# The database directory MUST exist prior to running slapd AND
# should only be accessible by the slapd and slap tools.
# Mode 700 recommended.
directory       /usr/local/etc/openldap/datas/openldap-dataa
# Indices to maintain
index   objectClass     eq
```


[2. A Quick-Start Guide](http://www.openldap.org/doc/admin24/quickstart.html)


[Spring LDAP官方文档翻译(1-5章)](https://www.jianshu.com/p/77517e26a357)



### SSL
```
sudo openssl genrsa -out cakey.pem 2048
```

问题：
```
cakey.pem: Permission denied
140340072933264:error:0200100D:system library:fopen:Permission denied:bss_file.c:402:fopen('cakey.pem','w')
140340072933264:error:20074002:BIO routines:FILE_CTRL:system lib:bss_file.c:404:
```
权限导致。


[OpenLDAP配置TLS加密传输](https://www.cnblogs.com/netonline/p/7517685.html)