# K8S


单台机器部署:

1. 编译安装软件(make install)  
2. 上线时，把代码同步(git/snv )上去


多台机器部署:
1. 收集线上所需要的软件版本
2. 在每台机器上 编译安装软件(make install)  
3. 把代码同步(rsync/scp) 上线

多台机器部署-docker版:
1. 安装docker
2. 收集线上所需要的软件版本
3. docker 下载 各种软件镜像
4. 每个项目编写自己的部署脚本
	- 依赖哪些软件及软件的版本
	- 项目的启动脚本
	- 项目的GIT仓库及使用GIT哪个分支
1. 编写 docker-composer ，创建每个项目需要的容器及依赖容器
2. 启动项目 docker 容器





>最原始

master/slave

pod：读取配置文件，docker ：创建、拉取镜像、设置共享硬盘和网络、日志等
node：一台实际的机器/服务器
kubectl：客户端工具，用于管理k8s

namespace
api-server：管理API的一个组件
scheduler：调度器。如：一个新的POD没有NODE创建了，进行调度。
kubelet
kube\-proxy

controller-manager
master\-\>node

master = api\-server scheduler controller\-manager

kubectl \-\>master

admin\-web\-uid

