---
title: 5-1MySQL高可用架构
tags: MySQL
grammar_cjkRuby: true
description: 网易DBA微专业笔记
---
## 高可用
应用提供持续不间断（可用）的服务的能力
系统高可用性的平价通常用可用率表示

|可用率目标|全年不可用时间
| --- |---|
|99%|87.6小时|
|99.9%|8.7小时|
|99。99%|52分钟|
|99.999%|5分钟|

### 造成不可用的原因
- 硬件故障（各种）
- 预期中的系统软硬件维护
- 软件缺陷（应用代码，服务器程序都可能存在bug..）
- 攻击，泄露，人为失误...等安全事件
- 对于系统来说，不可用时间是各关键组件不可用时间的总和。

## 提高可用性的主要手段
- 冗余，RedunDancy
- 关键软硬件通过备份冗余避免故障时长时间的不可用
- 数据软件，硬件，存储的数据，都需要通过冗余确保故障时可替换。

## 数据库冗余与可用性
数据库服务在冗余实现上有其特殊性
数据：服务“有状态”与数据冗余
实现放回多种多样，同一种数据也会有多种实现方案
可用性目标循序渐进：
任何故障都不会造成数据丢失→可以较快速恢复服务（高可用）

## MySQL-基于共享存储的单活方案（比较扯）

![enter description here][1]

非常可行，非常有效、非常稳定
共享存储一般由比较靠谱的几家供应商提供，如IBM等大厂提供
但费用比较昂贵，而且两台主机的成本只有一台在提供服务。
从技术目标上非常稳健可靠。
## 基于存储复制的数据冗余单活（不常用）

![enter description here][2]

两台机器仅一台提供服务，存在一定浪费，且DRBD也不能做到百分之百的保障。

## 基于MySQL主从复制（常用，普适）

![enter description here][3]

主库读写，从库可用于部分只读，另一从库只做备份或一些离线计算。

### 需要改进的问题
- 主从服务器各自有IP地址，发生主从切换后应用需要修改重启
- 人工判断主库是否故障在发起切换需要花较多时间
- 主从复制存在客观延迟，切换后可能造成事务数据丢失。

### 方案改进
- 为了避免应用人工修改切换IP，引入VIP漂移方案

 
 **方案一：使用虚拟IP** 。一旦主库有问题，将虚拟IP绑定从库。一旦漂移成功，程序基本不需要做什么修改。



![enter description here][4]

![enter description here][5]


**方案二：使用域名解析**。主库出问题，将域名解析到从库。

- 为了减少人工接入处理的时间开销引入自动探活处理机制。

![enter description here][6]

### 高可用中间层与RDS

- VIP解决应用切换问题
- 监控和管理服务器解决自动判断故障切换和VIP漂移
- VIP管理+探活+主从关系切换 = 高可用中间层
- 云环境 + 高可用中间层 + 底层数据库 = 一种PaaS = 基本RDS

### 高可用中间层

比较保守的两种。

#### MHA
- 自动选择复制延迟最小的从节点并试图补全日志（但大部分主机故障情况下行不通）
- 通常要求两从以上，会进行主从关系切换
- 不提供VIP管理方案。--需要自己通过

线上项目推荐方案。

#### MMM
- 提供了基本的VIP管理功能
- 适合双主配置的一对主机，不会主动主从关系
- 不支持主从数据延迟判断和补全
### MySQL主从复制延迟
日志传输延迟

数据库接收一个commit请求后，主库日志持久化到redo log中，redo log 一旦持久化成功返回commit，告知用户持久化成功，不需要写进数据库文件中，只需写进主库日志中即可。数据从写入主库日志然后到传输到从库，有时间间隔。从主库写到从库中再到从库持久化完成又一个时间间隔。

若主库日志持久化完成到传输到从库期间发生问题，可能由于主库出问题没来得及传到从库从而导致主从不一致。

![enter description here][7]

### MySQL半同步

主库接收到commit提交，主库完成持久化外需保证传到从库并在从库完成持久化才对主库返回commit成功。

当等待从库时，时间不可控。

![enter description here][8]

如，从库出问题

![enter description here][9]

有个关键参数，可以在等待n秒后，可将后面所有请求降级为异步，不保证从库的提交方式等。



## 基于集群提交通信协议的多主复制（一定场景使用）

![enter description here][10]

## 较完善的MySQL高可用方案

半同步复制+高可用中间层+VIP管理方案

如：
- 半同步复制+MHA+Keepalive
- 半同步复制+RDS

## MySQL高可用框架-MHA
- 用一个管理节点监控后端数据库可用性
- 提供VIP漂移接口，不提供具体办法
- 提供补全从库日志的脚步--不是特别实用。

## MHA的安装步骤
- 1、规划
- 2、配置服务器间域名和ssh互信访问
- 3、在manager节点暗中MHA node和manager组件及其依赖包。
- 4、在数据库服务器安装MHA node组件及其依赖包
- 5、配置VIP管理脚本master_ip_failover和master_iponlone_change
- 6、配置MHA配置文件mha_manager.cnf
- 7、数据库配置主从，添加mha连接用户...etc
- 8、启动MHA开始监控数据库主从集群

### 演示MHA部署--环境介绍

即步骤1规划

192.168.0.113      debtest1 -->MHA manager

192.168.0.114      debtest2 -->mysql1(master)

192.168,0.115       debtest3 -->mysql2(slave)

192.168.0.119                      --> vip

结构图：

	![enter description here][11]

### 配置
1、确保hosts和hostname配置正确

即确保三台机器上的ip、域名网络配置正确。
```
##
 
root@debtest1:~/.ssh# cat /etc/hosts
127.0.0.1    localhost
192.168.0.113    debtest1.sam.test    debtest1
192.168.0.114    debtest2.sam.test    debtest2
192.168.0.115    debtest3.sam.test    debtest3
 
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
root@debtest1:~/.ssh# cat /etc/hostname 
debtest1

```
2、使用ssh-keygen工具生成统一公私钥对,并同步到所有三台机器.测试公私钥验证访问

ssh命令：
```
ssh-keygen -t rsa
```
可生成公私钥对，生成时不要输出密码，这样拿公钥直接访问即可。id_rsa为私钥，id_rsa.pub为公钥。

命令：
```
cat id_rsa.pub > authorized_keys
```
将公钥写入authorized_keys文件（本地存储公钥，拿公钥对应的私钥访问时可直接通过）中

``` 
ssh-keygen -t rsa
 
cd ~/.ssh
cat id_rsa.pub authorized_keys
 
 ### 将文件同步到其他两台主机上
scp ./* root@debtest2:/root/.ssh/
scp ./* root@debtest3:/root/.ssh/
 
 ### 完成后，由于第一次访问需要输入yes，所以需要在每台服务器上都需要执行一遍如下操作：
 
ssh debtest1
ssh debtest2
ssh debtest3
```
3、规划节点用户和ip配置
```
##
 
 
192.168.0.113    debtest1 --> master
192.168.0.114    debtest2 --> mysql1(master)
192.168.0.115    debtest3 --> mysql2(slave)
192.168.0.119    vip
 
定好虚拟ip后别忘了在当前的主库上添加虚拟ip
##
ip addr add 192.168.0.119/32 dev eth1
 
#删除虚拟ip的命令
##ip addr del 192.168.0.119/32 dev eth1
```
4、在全部节点上安装mha node包和其依赖包
```
debtest1,debtest1,debtest1
 
apt-get install libdbd-mysql-perl
 
dpkg -i mha4mysql-node_0.53_all.deb
```

5、仅需要在manager节点上安装mha manager包及其依赖包
```
##
 
debtest1
 
apt-get install libdbd-mysql-perl
apt-get install libconfig-tiny-perl
apt-get install liblog-dispatch-perl
apt-get install libparallel-forkmanager-perl
 
dpkg -i mha4mysql-manager_0.53_all.deb
```

6、建立配置文件目录,编辑mha必要的三个文件,一个配置文件,2个虚拟ip管理脚本
```
##
 
master_ip_online_change
master_ip_failover
mha_manager.cnf
 
```
7、可以尝试验证一下配置是否成功
```
##
 
masterha_check_ssh --conf=./mha_manager.cnf
masterha_check_repl --conf=./mha_manager.cnf
```

8.在manager节点启动mha服务,然后观察日志,并尝试关闭当前主库,注意观察日志,主要看失效发现,日志检测,ip漂移和角色切换过程
```
nohup /usr/bin/masterha_manager --conf=/root/mha_base/mha_manager.cnf --ignore_last_failover  < /dev/null > /root/mha_base/manager.log 2>&1 &
```
### mha_manager.cnf配置文件
```
[server default]
manager_workdir=/root/mha_base
manager_log=/root/mha_base/manager.log
remote_workdir=/root/mha_base

ssh_user=root
ssh_port=22
user=mha
password=mha
repl_user=repl
repl_password=repl
multi_tier_slave=1
ping_interval=1
ping_type=CONNECT
master_ip_failover_script=/root/mha_base/master_ip_failover
master_ip_online_change_script=/root/mha_base/master_ip_online_change
secondary_check_script=/usr/bin/masterha_secondary_check -s 192.168.0.113 -s 192.168.0.115 --user=root --port=22 --master_host=debtest2 --master_ip=192.168.0.114 --master_port=3306


[server1]
candidate_master=0
ignore_fail=1
check_repl_delay = 1
hostname=debtest2
ip=192.168.0.114
port=3306
ssh_port=22
master_binlog_dir=/var/log/mysql/

[server2]
candidate_master=0
ignore_fail=1
check_repl_delay = 1
hostname=debtest3
ip=192.168.0.115
port=3306
ssh_port=22
master_binlog_dir=/var/log/mysql/
```
### master_ip_failover

管理IP的脚本,线上推荐带有自身检测或探活检测功能的程序管理vip

核心为：

```
my $vip = '192.168.0.119/24'; #virtual ip  定义虚拟ip--VIP
my $ssh_start_vip = "ip addr add $vip dev eth1";  ---添加VIP，到第一块网卡添加vip
my $ssh_stop_vip = "ip addr del $vip dev eth1";-----删除vip
```

完整为：

```
#!/usr/bin/env perl

## Note: This is a sample script and is not complete. Modify the script based on your environment.

use strict;
use warnings FATAL => 'all';

use Getopt::Long;
use MHA::DBHelper;


my (
  $command, $ssh_user, $orig_master_host,
  $orig_master_ip, $orig_master_port, $new_master_host,
  $new_master_ip, $new_master_port
);

my $vip = '192.168.0.119/24'; #virtual ip
my $ssh_start_vip = "ip addr add $vip dev eth1";
my $ssh_stop_vip = "ip addr del $vip dev eth1";


GetOptions(
  'command=s' => \$command,
  'ssh_user=s' => \$ssh_user,
  'orig_master_host=s' => \$orig_master_host,
  'orig_master_ip=s' => \$orig_master_ip,
  'orig_master_port=i' => \$orig_master_port,
  'new_master_host=s' => \$new_master_host,
  'new_master_ip=s' => \$new_master_ip,
  'new_master_port=i' => \$new_master_port,
  );
exit &main();

sub main {
  if ( $command eq "stop" || $command eq "stopssh" ) {

    # $orig_master_host, $orig_master_ip, $orig_master_port are passed.
    # If you manage master ip address at global catalog database,
    # invalidate orig_master_ip here.
    my $exit_code = 1;
    eval {
      print "Disabling the VIP on old master: $orig_master_host \n";
          &stop_vip();
      $exit_code = 0;
          # updating global catalog, etc

    };
    if ($@) {
      warn "Got Error: $@\n";
      exit $exit_code;
    }
    exit $exit_code;
  }
  elsif ( $command eq "start" ) {

    # all arguments are passed.
    # If you manage master ip address at global catalog database,
    # activate new_master_ip here.
    # You can also grant write access (create user, set read_only=0, etc) here.
    my $exit_code = 10;
    eval {
      print "Enabling the VIP - $vip on old master: $new_master_host \n";
          &start_vip();
      $exit_code = 0;
    };
    if ($@) {
      warn $@;

      # If you want to continue failover, exit 10.
      exit $exit_code;
    }
    exit $exit_code;
  }
  elsif ( $command eq "status" ) {

    # do nothing
    exit 0;
  }
  else {
    &usage();
    exit 1;
  }
}

# Enable the VIP on the new_master
sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}

# Disable the VIP on the old_master

sub stop_vip() {
my $ssh_user = "root";
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
  print
"Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

  [1]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung01.png "mysql_study_gaokeyung01"
  [2]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung02.png "mysql_study_gaokeyung02"
  [3]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung03.png "mysql_study_gaokeyung03"
  [4]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung05.png "mysql_study_gaokeyung05"
  [5]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung06.png "mysql_study_gaokeyung06"
  [6]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung08.png "mysql_study_gaokeyung08"
  [7]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung09.png "mysql_study_gaokeyung09"
  [8]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung010.png "mysql_study_gaokeyung010"
  [9]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung011.png "mysql_study_gaokeyung011"
  [10]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung04.png "mysql_study_gaokeyung04"
  [11]: https://assets.windcoder.com/xiaoshujiang/mysql_study_gaokeyung012.png "mysql_study_gaokeyung012"