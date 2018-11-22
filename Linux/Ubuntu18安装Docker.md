

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