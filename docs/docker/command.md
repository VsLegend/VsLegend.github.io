# docker命令行

## 基础语法
```dockerfile
docker [OPTIONS] COMMAND SUBCOMMAND
```

> 通过docker COMMAND --help可以查看docker的相关命令，每个单独的command有自己的options选项和subcommand参数

**OPTIONS**：

-   --config string      Location of client config files (default "/root/.docker")

-  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and defaultcontext set with "docker context use")

-  -D, --debug              Enable debug mode

-  -H, --host list          Daemon socket(s) to connect to

-  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")

-  --tls                Use TLS; implied by --tlsverify

-  --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")

-  --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")

-  --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")

-  --tlsverify          Use TLS and verify the remote

-  -v, --version            Print version information and quit


<br><br>


**一级命令行(management)**

命令：

-  app*        Docker App (Docker Inc., v0.9.1-beta3)

-  builder     Manage builds

-  buildx*     Docker Buildx (Docker Inc., v0.7.1-docker)

-  config      Manage Docker configs

-  container   容器管理

-  context     Manage contexts

-  image       Manage images

-  manifest    Manage Docker image manifests and manifest lists

-  network     Manage networks

-  node        Manage Swarm nodes

-  plugin      Manage plugins

-  scan*       Docker Scan (Docker Inc., v0.12.0)

-  secret      Manage Docker secrets

-  service     Manage services

-  stack       Manage Docker stacks

-  swarm       Manage Swarm

-  system      Manage Docker

-  trust       Manage trust on Docker images

-  volume      Manage volumes


**次级命令行**

>一级命令行的下级命令，但并不是所有一级命令都有以下命令，可以通过一级命令加help（docker command --help）查看具体用法。
>
>注意一些下级命令行在重构前，直接通过docker + 下级命令实现
>例如：docker container run = docker run

命令：

-  attach      Attach local standard input, output, and error streams to a running container

-  build       Build an image from a Dockerfile

-  commit      Create a new image from a container's changes

-  cp          Copy files/folders between a container and the local filesystem

-  create      仅创建一个新容器

-  diff        Inspect changes to files or directories on a container's filesystem

-  events      Get real time events from the server

-  exec        在一个正在运行中的容器中执行命令行

-  export      Export a container's filesystem as a tar archive

-  history     Show the history of an image

-  images      镜像列表管理

-  import      Import the contents from a tarball to create a filesystem image

-  info        Display system-wide information

-  inspect     Return low-level information on Docker objects

-  kill        Kill one or more running containers

-  load        Load an image from a tar archive or STDIN

-  login       Log in to a Docker registry

-  logout      Log out from a Docker registry

-  logs        获取容器日志

-  pause       Pause all processes within one or more containers

-  port        List port mappings or a specific mapping for the container

-  ps          List containers

-  pull        Pull an image or a repository from a registry

-  push        Push an image or a repository to a registry

-  rename      Rename a container

-  restart     Restart one or more containers

-  rm          删除一个或多个容器

-  rmi         Remove one or more images

-  run         创建一个新容器并执行命令

-  save        Save one or more images to a tar archive (streamed to STDOUT by default)

-  search      Search the Docker Hub for images

-  start       重启一个或多个已停止的容器

-  stats       Display a live stream of container(s) resource usage statistics

-  stop        停止一个或多个正在运行的容器

-  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

-  top         Display the running processes of a container

-  unpause     Unpause all processes within one or more containers

-  update      Update configuration of one or more containers

-  version     Show the Docker version information

-  wait        Block until one or more containers stop, then print their exit codes

