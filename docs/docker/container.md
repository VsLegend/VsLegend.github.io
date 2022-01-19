# 容器管理

## container语法
```dockerfile
docker container COMMAND
```

**次级命令**：

-  attach      将本地标准输入、输出和错误流附加到正在运行的容器

-  commit      Create a new image from a container's changes

-  cp          Copy files/folders between a container and the local filesystem

-  create      Create a new container

-  diff        Inspect changes to files or directories on a container's filesystem

-  exec        Run a command in a running container

-  export      Export a container's filesystem as a tar archive

-  inspect     Display detailed information on one or more containers

-  kill        Kill one or more running containers

-  logs        Fetch the logs of a container

-  ls          List containers

-  pause       Pause all processes within one or more containers

-  port        List port mappings or a specific mapping for the container

-  prune       Remove all stopped containers

-  rename      Rename a container

-  restart     Restart one or more containers

-  rm          Remove one or more containers

-  run         Run a command in a new container

-  start       Start one or more stopped containers

-  stats       Display a live stream of container(s) resource usage statistics

-  stop        Stop one or more running containers

-  top         Display the running processes of a container

-  unpause     Unpause all processes within one or more containers

-  update      Update configuration of one or more containers

-  wait        Block until one or more containers stop, then print their exit codes

<br>

## 创建容器
```dockerfile
docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**OPTIONS**：

- --name	为容器命名

- --rm		容器关闭时，自动删除该容器

- --workdir , -w		容器内部的工作目录

- --mount		将数据卷挂载到容器

- ……

- 其他OPTIONS见：[docker container create](https://docs.docker.com/engine/reference/commandline/container_create/)

<br>

## 启动容器
> 后台启动

```dockerfile
docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

**OPTIONS**：

- --attach , -a		Attach STDOUT/STDERR and forward signals

- --checkpoint		experimental (daemon)Restore from this checkpoint

- --checkpoint-dir		experimental (daemon)Use a custom checkpoint storage directory

- --detach-keys		Override the key sequence for detaching a container

- --interactive , -i		Attach container's STDIN

将一个已经终止（exited）的容器启动运行。容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 ps 或 top 来查看进程信息。

<br>

## 创建新容器并执行命令
```dockerfile
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```


**OPTIONS**：

- --name	为容器命名

- --rm		容器关闭时，自动删除该容器

- --workdir , -w		容器内部的工作目录

- --detach , -d		后台运行容器并返回容器id

- --mount		将数据卷挂载到容器

- ……

- 其他OPTIONS见：[docker container run](https://docs.docker.com/engine/reference/commandline/container_run/)


**COMMAND**：

- -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上

- -i 则让容器的标准输入保持打开


>
>  当利用 docker container run （docker run）来创建容器时，Docker 在后台运行的标准操作包括：
>
>  - 检查本地是否存在指定的镜像，不存在就从 registry 下载
>
>  - 利用镜像创建并启动一个容器
>
>  - 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
>
>  - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
>
>  - 从地址池配置一个 ip 地址给容器
>
>  - 执行用户指定的应用程序
>
>  - 执行完毕后容器被终止

<br>

## 后台容器执行命令
```dockerfile
docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**OPTIONS**：

- --detach , -d		Detached mode: run command in the background

- --detach-keys		Override the key sequence for detaching a container

- --env , -e		API 1.25+ Set environment variables

- --env-file		API 1.25+ Read in a file of environment variables

- --interactive , -i		Keep STDIN open even if not attached

- --privileged		Give extended privileges to the command

- --tty , -t		Allocate a pseudo-TTY

- --user , -u		Username or UID (format: <name|uid>[:<group|gid>])

- --workdir , -w		API 1.35+ Working directory inside the container

<br>

## 后台容器执行命令，退出时终止容器
```dockerfile
docker container attach [OPTIONS] CONTAINER
```

**OPTIONS**：

- --detach-keys		Override the key sequence for detaching a container

- --no-stdin		Do not attach STDIN

- --sig-proxy	true	Proxy all received signals to the process

<br>

## 终止容器
```dockerfile
docker container stop [OPTIONS] CONTAINER [CONTAINER...]
```

**OPTIONS**：
- --time , -t	10	Seconds to wait for stop before killing it

当 Docker 容器中指定的应用终结时，容器也自动终止。
例如对于上一章节中只启动了一个终端的容器，用户通过 exit 命令或 Ctrl+d 来退出终端时，所创建的容器立刻终止。终止状态的容器可以用 docker container ls -a 命令看到。

<br>

## 删除容器
删除一个停止运行的容器
```dockerfile
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

**OPTIONS**：

- --force , -f		强制删除运行中的容器 (uses SIGKILL)

- --link , -l		Remove the specified link

- --volumes , -v		Remove anonymous volumes associated with the container

<br>

## 清理终止的容器
清理所有处于终止状态的容器
```dockerfile
docker container prune [OPTIONS]
```

**OPTIONS**：

- --filter		Provide filter values (e.g. 'until=<timestamp>')

- --force , -f		不提示确认信息

<br>

## 查看后台运行容器的日志
```dockerfile
docker container logs [OPTIONS] CONTAINER
```

**OPTIONS**：

- --details		展示额外的详细日志信息

- --follow , -f		同tail -f，即时更新日志

- --since		Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)

- --tail , -n	all	展示日志行数

- --timestamps , -t		展示时间戳

- --until		API 1.35+ Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)

<br>

## 查看容器列表
```dockerfile
docker container ls [OPTIONS]
```

**OPTIONS**：

- --all , -a		展示所有容器（默认展示运行中容器）

- --filter , -f		Filter output based on conditions provided

- --format		Pretty-print containers using a Go template

- --last , -n	默认-1，展示最早创建的n条容器 (includes all states)

- --latest , -l		展示最近创建的n条容器 (includes all states)

- --no-trunc		Don't truncate output

- --quiet , -q		Only display container IDs

- --size , -s		展示文件大小
