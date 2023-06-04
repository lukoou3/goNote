
## 创建容器常用选项
命令格式： `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

| 选项                                      | 描述                                                      |
| ----------------------------------------- | --------------------------------------------------------- |
| -i, –interactive                          | 交互式                                                    |
| -t, –tty                                  | 分配一个伪终端                                            |
| -d, –detach                               | 运行容器到后台                                            |
| -e, –env                                  | 设置环境变量                                              |
| -p, –publish list                         | 发布容器端口到主机                                        |
| -P, –publish-all                          | 发布容器所有EXPOSE的端口到宿主机随机端口                  |
| --name string                             | 指定容器名称                                              |
| -h, –hostname                             | 设置容器主机名                                            |
| --ip string                               | 指定容器IP，只能用于自定义网络                            |
| --network                                 | 连接容器到一个网络                                        |
| -v, –volume list <br/> --mount mount（新方式） | 将文件系统附加到容器                                      |
| --restart string                          | 容器退出时重启策略，默认no，可选值： [always\|on-failure] |
| -m， –memory                              | 容器可以使用的最大内存量                                  |
| –memory-swap                              | 允许交换到磁盘的内存量                                    |
| –memory-swappiness=<0-100>                | 容器使用SWAP分区交换的百分比（0-100，默认为-1）           |
| –oom-kill-disable                         | 禁用OOM Killer                                            |
| --cpus                                    | 可以使用的CPU数量                                         |
| –cpuset-cpus                              | 限制容器使用特定的CPU核心，如(0-3, 0,1)                   |
| –cpu-shares                               | CPU共享（相对权重）                                       |

## 常用管理命令
命令格式： `docker container COMMAND`容器数据持久化
| 指令               | 命令                                        | 描述                                                      |
| ------------------ | ------------------------------------------- | --------------------------------------------------------- |
| ls                 | docker container ls 或者 docker ps          | 列出容器                                                  |
| inspect            | docker container inspect test               | 查看一个或多个容器详细信息                                |
| exec               | docker  [container] exec -it test /bin/bash | 在运行容器中执行命令                                      |
| commit             |                                             | 创建一个新镜像来自一个容器                                |
| cp                 | docker [container] cp data test:/tmp/       | 拷贝文件/文件夹到一个容器                                 |
| logs               | docker [container] logs test                | 获取一个容器日志                                          |
| port               | docker container port test                  | 列出或指定容器端口映射                                    |
| top                | docker [container] top test                 | 显示一个容器运行的进程                                    |
| stats              | docker [container] stats test               | 显示容器资源使用统计                                      |
| create             | docker [container] create                   | 新建一个容器                                              |
| run                | docker [container] run                      | 新建并启动容器，等价于先执行create命令，再执行start命令。 |
| stop/start/restart | docker [container] stop/start/restart       | 停止/启动一个或多个容器                                   |
| rm                 | docker [container] rm                       | 删除一个或多个容器                                        |
| prune              | docker container prune                      | 自动移除已停止的容器                                      |

## 容器数据持久化
Docker提供2种方式将数据从宿主机挂载到容器中：

* volumes： Docker管理宿主机文件系统的一部分（/var/lib/docker/volumes） 。
* bind mounts：将宿主机上的任意位置的文件或者目录挂载到容器中。

volumes方式可以忽略，基本使用bind mounts方式。

volumes示例：
1、创建数据卷
```
# docker volume create nginx-vol
# docker volume ls
# docker volume inspect nginx-vol
```
2、使用数据卷
```
# docker run -d --name=nginx-test --mount src=nginxvol,dst=/usr/share/nginx/html nginx
# docker run -d --name=nginx-test -v nginxvol:/usr/share/nginx/html nginx
```

**bind mounts示例**：

**挂载宿主机目录到容器**
```
# docker run -d --name=nginx-test --mount type=bind,src=/app/wwwroot,dst=/usr/share/nginx/html nginx
# docker run -d --name=nginx-test -v /app/wwwroot:/usr/share/nginx/html nginx
```

## 容器网络
![](assets/markdown-img-paste-20230604173141591.png)

* veth pair：成对出现的一种虚拟网络设备，数据从一端进,从另一端出。 用于解决网络命名空间之间隔离。  
* docker0：网桥是一个二层网络设备，通过网桥可以将Linux支持的不同的端口连接起来，并实现类似交换机那样的多对多的通信。  

Docker使用iptables实现网络通信。

![](assets/markdown-img-paste-20230604173336588.png)

```
外部访问容器：
# iptables -t nat -vnL DOCKER
容器访问外部：
# iptables -t nat -vnL POSTROUTING
```







