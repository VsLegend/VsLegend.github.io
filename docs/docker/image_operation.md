# 镜像管理

## 获取镜像
```dockerfile
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

例子
```dockerfile
docker pull mysql:8
```


**OPTIONS**：

- --all-tags , -a		Download all tagged images in the repository

- --disable-content-trust	默认：true	跳过镜像验证

- --platform		API 1.32+ Set platform if server is multi-platform capable

- --quiet , -q		Suppress verbose output


> 
> 具体的选项可以通过 docker pull --help 命令看到，这里说一下镜像名称NAME的格式。
> 
> NAME\[:TAG|@DIGEST]全部内容：**<域名/IP>\[:端口号] <仓库名>\[:标签|@摘要]**
> 
> 如果不填域名内容，那么默认从Docker官方镜像仓库获取镜像，其地址是 Docker Hub(docker.io)。
> 
> 此外，<仓库名>其实包含了两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
> 
> 名称后面的tag标签或DIGEST摘要是可选的，一般不选时，会默认拉取镜像的latest标签。每个摘要对应唯一产品。

<br>

## 列出本地镜像
```dockerfile
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

**OPTIONS**：

- --all , -a		展示全部镜像 (default hides intermediate images)

- --digests		展示摘要（hash）信息

- --filter , -f		Filter output based on conditions provided

- --format		Pretty-print images using a Go template

- --no-trunc		Don't truncate output

- --quiet , -q		只展示镜像IDs

<br>

## 查看本地镜像占用空间
```dockerfile
docker system df [OPTIONS]
```

**OPTIONS**：

- --format		Pretty-print images using a Go template

- --verbose , -v		Show detailed information on space usage

<br>

## 删除本地镜像
```dockerfile
docker image rm [OPTIONS] IMAGE [IMAGE...]
```

**OPTIONS**：

- --force , -f		强制删除

- --no-prune		Do not delete untagged parents

> \<IMAGE > 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要（DIGEST）
> 1. docker image ls 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了
> 1. 也可以用镜像名，也就是 <仓库名>:<标签>，来删除镜像
> 1. 更精确的是使用镜像摘要（docker image ls --digests）删除镜像

<br>

## 查询镜像与删除镜像配合
```dockerfile
docker image rm $(docker image ls -q redis)
```

像其它可以承接多个实体的命令一样，可以使用 docker image ls -q 来配合使用 docker image rm，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

<br>

## 删除虚悬镜像
```dockerfile
docker image prune [OPTIONS]
```

**OPTIONS**：

- --all , -a		Remove all unused images, not just dangling ones

- --filter		Provide filter values (e.g. 'until=<timestamp>')

- --force , -f		Do not prompt for confirmation


> docker pull、docker build 可能会生成虚悬镜像。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为<none>的镜像。
> 这类无标签镜像也被称为虚悬镜像(dangling image) ，一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的。

<br>

## 搜索镜像
```dockerfile
docker search [OPTIONS] TERM
```

**OPTIONS**：

- --filter , -f		Filter output based on conditions provided

- --format		Pretty-print search using a Go template

- --limit	25	Max number of search results

- --no-trunc		Don't truncate output



----

参考：[Docker — 从入门到实践](https://github.com/yeasy/docker_practice)
