```shell
sudo yum install screen -y
screen -S fabric
git clone https://github.com/hyperledger/fabric.git
git clone https://github.com/hyperledger/fabric-samples.git
```
![](images/2019-01-23-10-12-40.png)

### 生成配置
```
./byfn.sh -m generate -c jschannel
```
![](images/2019-01-23-10-18-07.png)

### 启动网络

```
./byfn.sh -m up -c jschannel
```
![](images/2019-01-23-10-46-07.png)
```
screen -S fabric
screen -ls
screen -r fabric
```
[inux screen 命令详解](https://www.cnblogs.com/cute/p/5015852.html)

[centos7安装配置Hyperledger fabric1.4.0](https://blog.csdn.net/asn_forever/article/details/86505376)

[安装Fabric示例](http://chixiang.me/2018/07/13/%E5%AE%89%E8%A3%85Fabric%E7%A4%BA%E4%BE%8B/#more)

```
# 测试
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 1.4.0
```

### 错误

```
[wind@localhost first-network]$ ./byfn.sh -m up -c jschannel
Starting for channel 'jschannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
LOCAL_VERSION=1.4.0
DOCKER_IMAGE_VERSION=1.4.0
./byfn.sh:行169: docker-compose: 未找到命令
ERROR !!!! Unable to start network
```


https://github.com/docker/compose/releases

[CentOS7 - Docker&Docker-Compose安装](https://blog.csdn.net/qq_38591756/article/details/82828130#%E4%B8%89%E3%80%81Docker%20Compose%E5%AE%89%E8%A3%85)

github-production-release-asset-2e65be.s3.amazonaws.com