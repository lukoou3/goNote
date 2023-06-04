
## 镜像是什么
镜像是Docker三大核心概念中最重要的，自Docker诞生之日起镜像就是相关社区最为热门的关键词。

Docker运行容器前需要本地存在对应的镜像，如果镜像不存在，Docker会尝试先从默认镜像仓库下载（默认使用Docker Hub公共注册服务器中的仓库），用户也可以通过配置，使用自定义的镜像仓库。

* 一个分层存储的文件，不是一个单一的文件  
* 一个软件的环境  
* 一个镜像可以创建N个容器  
* 一种标准化的交付  
* 一个不包含Linux内核而又精简的Linux操作系统配置加速器  

## 常用管理命令
| 指令    | 命令                                              | 描述                                       |
| ------- | ------------------------------------------------- | ------------------------------------------ |
| search  | docker search [option] keyword                    | 搜寻镜像                                   |
| ls      | docker images 或者docker image ls                  | 列出镜像                                   |
| build   | docker build -t tag .                             | 构建镜像来自Dockerfile                     |
| history | docker history ubuntu:18.04                       | 查看镜像历史                               |
| inspect | docker [image] inspect ubuntu:18.04               | 显示一个或多个镜像详细信息                 |
| pull    | docker pull ubuntu:18.04                          | 从镜像仓库拉取镜像                         |
| push    | docker push user/test:latest                      | 推送一个镜像到镜像仓库                     |
| rm      | docker rmi tag_or_id或者docker image rm tag_or_id | 移除一个或多个镜像                         |
| prune   | docker image prune                                | 移除没有被标记或者没有被任何容器引用的镜像 |
| tag     | docker tag ubuntu:latest myubuntu:latest          | 创建一个引用源镜像标记目标镜像             |
| save    | docker save -o ubuntu_18.04.tar ubuntu:18.04      | 保存一个或多个镜像到一个tar归档文件        |
| load    | docker load -i ubuntu_18.04.tar                   | 加载镜像来自tar归档或标准输入              |




