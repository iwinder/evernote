
## 安装Docker-Compose

Fabric需要Docker-Compose，所以顺便记录一下

最新版可去 [Docker-Compose](https://github.com/docker/compose/releases) 获取。

按其所言,原则上执行如下两行命令即可。

```shell
# 安装
curl -L https://github.com/docker/compose/releases/download/1.24.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 赋权
chmod +x /usr/local/bin/docker-compose
```

由于国内原因，可能会出现类似```github-production-release-asset-2e65be.s3.amazonaws.com```连接失败的问题。

- 可以通过将https协议改为http试试，多数情况下会成功。

- 也可以在host文件添一条类似映射，该方案未测试（ip地址百度可以得到好多，需要从中选一个能用的）。
```
52.216.20.171	github-production-release-asset-2e65be.s3.amazonaws.com
```

非Linux用户的安装方法在： [https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose) 