

```
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common


    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


     sudo apt-key fingerprint 0EBFCD88

```


获取Ubuntu发行版的名称
```
lsb_release -cs
```
获得
```
cosmic
```

查看本机的系统架构
```
amd64
```
![](images/2018-11-23-00-12-10.png)

执行
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   cosmic\
   stable"
```

18.10的暂时 没有，故用的18.04的
```
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu  bionic  stable"
```

```
sudo apt-get update

sudo apt-get install docker-ce

```


https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-ce-1


https://hyperledger-fabric.readthedocs.io/en/latest/install.html