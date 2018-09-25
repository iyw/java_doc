centos7 安装docker 
===

## 介绍
Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。 
社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。 
Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。 
总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

#### 下面一些概念有助于理解docker： 
1. Docker 镜像(Images)：Docker镜像是用于创建 Docker 容器的模板。 
2. Docker容器(Container)：容器是独立运行的一个或一组应用。 
3. Docker客户端(Client)：Docker 客户端通过命令行或者其他工具使用 Docker API (https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信。 
4. Docker主机(Host)：一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。 
5. Docker仓库(Registry)：Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。 
6. Docker Hub(https://hub.docker.com)： 提供了庞大的镜像集合供使用。 
7. Docker Machine：Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。

#### 前提条件
1.64位版本的CentOS 7。命令lsb_release -a 
2.CentOS系统的内核版本高于 3.10。命令uname -r



#### centos下安装

##### 1.卸载（可选）

Docker的旧版本被称为docker或docker-engine，若以前安装过，卸载命令如下：
```vim
yum remove docker docker-common container-selinux docker-selinux docker-engine
```
有些文章在这里使用了yum upgrade,命令的意思是升级所有包同时也升级软件和系统内核。我建议最好不执行命令，可能千万系统崩溃，因为系统版本从低级升级到高级，有些软件可能会出现问题。

##### 2.安装yum-utils device-mapper-persistent-data lvm2软件包

```vim
yum install -y yum-utils device-mapper-persistent-data lvm2
```

##### 3.配置稳定版本库

大多数用户设置了Docker的存储库并从中安装，以方便安装和升级任务。 这是推荐的方法

```vim
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

##### 4.查看所有仓库中所有docker版本
查看所有社区版的docker-ce版本。
```vim
yum list docker-ce --showduplicates | sort -r
```

1. repo中默认只开启stable仓库，所以这里展示的都是稳定版本的docker。 
2. 使用sort -r命令对结果进行排序，版本号由最高到最低，并被截断。 
3. 第一列是软件名，第二列是版本字符串， 第三列是存储库名称，安装指定版本使用包名-第二列是版本字符串



##### 5.安装docker

指定版本安装，如下：
```vim
yum install docker-ce-18.03.1.ce-1.el7.centos
```
最新版本
```vim
yum install docker-ce1

```
由于repo中默认只开启stable仓库，故这里安装的是最新稳定版18.03

##### 6.启动docker

启动命令
systemctl start docker1

查看版本
docker -version1



#### 7.卸载

查看docker已经安装版本

yum list installed | grep docker1

卸载
```vim
 yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine12345678910

```

##### 8.添加DOCKER_HOST

打开

```vim
vi /etc/profile
在文件末尾添加（如果默认端口2375修改成了9000）

export DOCKER_HOST=tcp://0.0.0.0:23751
让配置生效：

source /etc/profile1
```



##### 9.开放远程访问端口

```vim
打开
vi /usr/lib/systemd/system/docker.service

在ExecStart后面添加

-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock 

结果如下
ExecStart=/usr/bin/dockerd-current -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock 
```

#### 安装docker-machine

命令

```vim
curl -L https://github.com/docker/machine/releases/download/v0.10.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine

```
测试
docker-machine -v


#### docker启动/重启/停止等命令
添加docker开机启动：systemctl enable docker 
启动命令：systemctl start docker 
重启命令：systemctl restart docker 
停止命令：systemctl stop docker 
查看版本：docker version 
卸载：yum remove docker-ce 
查看窗口端口映射：docker ps 
查看所以容器（包括被关闭的）:docker ps --all 
其它docker命令参数这里


#### 配置加速器

由于访问官方镜像地址很慢，不得不配置国内的镜像，我这里使用阿里云的加速器，毕竟，我ECS，mysql，redis都使用阿里云的，那我加速器也使用它的，个人选择。 
登陆阿里云 - 开发者平台,和阿里云控制台帐号是通用帐号，登陆成功，左侧中间选择镜像加速器标签，结果如下： 
 
按图式方法配置CentOS系统上Docker加速器:

vi /etc/docker/daemon.json
修改文件如下
```json
{
  "registry-mirrors": ["https://l8ue6x6v.mirror.aliyuncs.com"]
}
```
  这里有几个其它的加速器，只要替换上面的地址即可 
  网易加速器：http://hub-mirror.c.163.com 
  官方中国加速器：https://registry.docker-cn.com 
  ustc的镜像：https://docker.mirrors.ustc.edu.cn 
  daocloud：https://www.daocloud.io/mirror#accelerator-doc（注册后使用）
重新加载配置，重启docker
```vim
systemctl daemon-reload
systemctl restart docker
```


测试加速器效果 
摘取镜像

docker pull busybox

成功打印出“hello world”说明阿里云加速器配置成功。
```vim
docker run busybox echo “hello world” 
```