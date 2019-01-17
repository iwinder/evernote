## 卸载旧版本
```shell
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
## 安装需要的包
```shell
 sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

## 添加安装仓库地址
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
### 查看可用版本
```
yum list docker-ce --showduplicates | sort -r
```

### 开启与废弃非稳定版本

#### 1.开启：
可选是否需要开启相应的非稳定版本，默认为disable的。
```
## 开启edge版本
sudo yum-config-manager --enable docker-ce-edge
## 开启test版本
sudo yum-config-manager --enable docker-ce-test
```

#### 2.关闭：
参照开启部分

```
sudo yum-config-manager --disable docker-ce-edge
sudo yum-config-manager --disable docker-ce-test
```
### 'Operation timed out after 30004 milliseconds with 0 out of 0 bytes received'
国内安装时可能无法连接到官方库，从而出现超时错误。可通过添加国内库地址。

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装
```
sudo yum install docker-ce
```

## 启动
```
sudo systemctl start docker
```
## 运行hello-world镜像验证安装
```
 sudo docker run hello-world
```

## 非root用户使用docker
如果您想以非root用户使用Docker，需要将您的用户(your-user)添加到“docker”组，例如：

```
sudo usermod -aG docker your-user
```

之后登出，重新登录shell即可。此时执行上面的hello-world镜像无需sudo：
```
docker run hello-world
```

## 参考资料
[Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce)

### 扩展
[centos7 安装 docker-ce](https://www.jianshu.com/p/36d43ff8aa9a)