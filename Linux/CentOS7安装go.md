```shell
# 官方 https://golang.org/dl/ 获取所需版本并下载
wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz

# 解压
tar -zxvf go1.11.4.linux-amd64.tar.gz

# 配置环境变量

## 打开文件

sudo vi /etc/profile

## 在最后一行添加如下，并保存退出

export GOROOT=/home/wind/apps/go
export PATH=$PATH:$GOROOT/bin

## 使用source使命令生效

source /etc/profile

```