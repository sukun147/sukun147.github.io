# Docker客户端与服务器机制


<!--more-->

# Docker客户端与服务器机制

## 简单介绍

![docker on ubuntu](https://s2.loli.net/2022/04/05/TvN5P4uqQFh9yeg.png)

当我们使用`docker version`命令查询docker 版本信息时，我们发现有两块内容，事实上这是由 docker 的客户端与服务器机制决定的。

Docker 使用了常见的**CS**架构，也就是 client-server 模式，Docker Client 负责处理用户输入的各种命令，比如`docker build`、`docker run`，真正工作的其实是 Docker 服务端，也就是 Docker Engine 或者说是 Docker Daemon。

值得注意的是，docker 客户端和 Docker Daemon 可以运行在同一台机器上。

当我们写完 dockerfile 交给 docker “编译”，使用`docker build`，那么 client 在接收到请求后转发给 docker daemon，接着 docker daemon根据 dockerfile 创建出“可执行程序” image。

接下来使用命令`docker run`，docker daemon 接收到该命令后找到具体的 image，然后加载到内存开始执行，image 执行起来就是所谓的 container。

用户通过 docker client 发送命令`docker pull`，docker daemon 接收到命令后向 docker registry 发送 image 下载请求，下载后存放在本地，这样我们就可以使用别人的 image 了。

{{< admonition success 举个栗子>}}

docker for Windows

{{< /admonition >}}

![img](https://www.runoob.com/wp-content/uploads/2016/04/576507-docker1.png)

## Linux Daemon（守护进程）

Linux Daemon（守护进程）是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。它不需要用户输入就能运行而且提供某种服务，不是对整个系统就是对某个用户程序提供服务。Linux 系统的大多数服务器就是通过守护进程实现的。常见的守护进程包括系统日志进程 syslogd、 web 服务器 httpd、邮件服务器 sendmail 和数据库服务器 mysqld 等。

守护进程一般在系统启动时开始运行，除非强行终止，否则直到系统关机都保持运行。守护进程经常以超级用户（root）权限运行，因为它们要使用特殊的端口（1-1024）或访问某些特殊的资源。

一个守护进程的父进程是 init 进程，因为它真正的父进程在 fork 出子进程后就先于子进程 exit 退出了，所以它是一个由 init 继承的孤儿进程。守护进程是非交互式程序，没有控制终端，所以任何输出，无论是向标准输出设备 stdout 还是标准出错设备 stderr 的输出都需要特殊处理。

守护进程的名称通常以 d 结尾，比如 sshd、xinetd、crond 等。

## Docker Daemon

Docker Daemon 即 Docker 的守护进程。

Docker Client 通过命令行与 Docker Damon 通信，Daemon 的主要功能包括镜像管理、镜像构建、REST API、身份验证、安全、核心网络以及编排。

### **工作机制**

Docker Daemon 可以认为是通过 Docker Server 模块接受 Docker Client 的请求，并在 Engine 中处理请求，然后根据请求类型，创建出指定的 Job 并运行，运行过程的作用有以下几种可能：向 Docker Registry 获取镜像，通过 graphdriver 执行容器镜像的本地化操作，通过 networkdriver 执行容器网络环境的配置，通过 execdriver 执行容器内部运行的执行工作等。

## Unix域套接字（Unix Domain Socket）

Docker 守护进程会生成一个 `/var/run/docker.sock`文件来进行本地进程通信，因此只能在本地使用 Docker 客户端或者使用 Docker API 进行操作。
sock 文件是 UNIX 域套接字，它可以通过文件系统（而非网络地址）进行寻址和访问。

**socket按域分类**

| 域        | 描述                 |
| --------- | -------------------- |
| AF_INET   | ipv4因特网域         |
| AF_INET6  | ipv6因特网域         |
| AF_UNIX   | Unix域（本地套接字） |
| AF_UPSPEC | 未指定               |

Unix 域套接字用于在**同一台机器上运行的进程之间**的通信。虽然因特网域套接字可用于同一目的，但 Unix 域套接字的**效率更高**。UNIX 域套接字**仅复制数据**；它们并不执行协议处理，不需要添加或删除网络报头，无需计算检验和，不要产生顺序号，无需发送确认报文。

Unix 域套接字提供流和数据报两种接口。UNIX域数据报服务是可靠的，既不会丢失消息也不会传递出错。Unix 域套接字是套接字和管道之间的混合物。

{{< admonition info 管道>}}

pipe 和 socket 都是比较常用的 IPC 方式（Inter-Process Communication）

管道是一种两个进程间进行单向通信的机制。因为管道传递数据的单向性，管道又称为**半双工管道**。

每一个 socket 都有两个数据缓冲区，读缓冲区，写缓冲区。用户往 socket 写数据其实就是往 socket 的写缓冲区拷贝数据，然后内核再把写缓冲区的数据拷贝到接收方 socket 的读缓冲区。两个缓冲区。

而对于 pipe，写的时候是在管道一端`pipefd[1]`塞数据，读的时候从另一端`pipefd[0]`读，两端是在共用一个缓冲区。pipe也仅仅是数据拷贝，不需要处理协议。pipe 中写操作是阻塞的，那么当缓冲区满了之后发送方的 write 操作将会被阻塞，而不会发生缓冲区溢出丢包这种情况。

{{< /admonition >}}

## docker in docker（dind）

有时需要在容器内执行 docker 命令，比如：在 jenkins 容器内运行 docker 命令执行构建镜像，但直接在 docker 容器内嵌套安装 docker 未免太过臃肿。

更好的办法是：容器内仅部署 docker 命令行工具（作为客户端），实际执行交由宿主机内的 docker-engine（服务器）。

### 方法

在docker容器内启动一个docker daemon，对外提供服务。

每个运行中的容器，都是一个进程，这个进程都托管在docker daemon中。

优点在于镜像和容器都在一个隔离的环境，保持宿主机的环境。

#### 通过宿主机的docker.sock

只要以数据卷的形式将 Docker 客户端和上述 Unix 域套接字挂载到容器内部，就能实现 "Docker in Docker"，在容器内使用 Docker 命令了。

通过类似`docker run -v /var/run/docker.sock:/var/run/docker.sock`的命令将宿主机 docker.sock 文件挂载到容器， 并且直接

挂载宿主机的`/usr/bin/docker`， 这样容器内就不需安装 Docker 程序。

当容器内使用 docker 命令时，实际上调用的是宿主机的 Docker Daemon 和 docker 命令。

也就是说，容器内实际并未运行 docker server，但是能够通过宿主机执行 docker 任务，从而实现简单的 dind。

需要特别说明的是，真正执行 docker 命令的是跑在宿主机上的 docker-engine（服务器），因此这并不是真正的 "Docker in Docker"，或许可以称之为“Docker outside Docker”。

#### 通过`docker:dind`镜像

先启动一个`docker:dind`容器A，再启动一个 docker 容器 B，容器 B 指定 host 为 A 容器内的 Docker Daemon。

### 示例1

```dockerfile
version: '3.3'
services:
  jenkins-master:
    image: jenkinsci/blueocean:latest
    container_name: jenkins-master
    environment:
      - TZ=Asia/Shanghai  # 时区
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - ./jenkins_home:/var/jenkins_home  # 将容器中的数据映射到宿主机
      - /usr/bin/docker:/usr/bin/docker  # 为容器内部提供 docker 命令行工具（这个随意）
      - /var/run/docker.sock:/var/run/docker.sock  # 容器内部通过 unix socket 使用宿主机 docker engine
    user: root  # 容器以有权限读写 docker.socket 的用户启动
    restart: always
```

### 示例2

本处示例来自https://www.cnblogs.com/anliven/p/13551614.html#_label2

```shell
docker network create jenkins

docker volume create jenkins-docker-certs

docker volume create jenkins-data

docker run --name jenkins-docker -d --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home docker:dind

docker run --name jenkins-blueocean -d --network jenkins --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro --publish 8080:8080 --publish 50000:50000 jenkinsci/blueocean

docker exec jenkins-blueocean cat var/jenkins_home/secrets/initialAdminPassword # 获取初始密码
```

## Docker API

### API

#### 什么是API

> 维基百科：
>
> **应用程序编程接口** （application programming interface，缩写 API） 是计算机之间或计算机程序之间的连接。它是一种软件接口，为其他软件提供服务。描述如何构建或使用此类连接或接口的文档或标准称为 API 规范。符合此标准的计算机系统称为实现或公开API。术语 API 可以指规范或实现。
>
> 与将计算机连接到人的用户界面相比，应用程序编程接口将计算机或软件片段相互连接。它不打算由将它合并到软件中的计算机程序员以外的人（最终用户）直接使用。API通常由不同的部分组成，这些部分充当程序员可用的工具或服务。使用这些部分之一的程序或程序员被称为API的该部分。组成 API 的调用也称为子例程、方法、请求或终结点。API 规范定义了这些调用，这意味着它解释了如何使用或实现它们。
>
> API的一个目的是隐藏系统如何工作的内部细节，只公开程序员会发现有用的部分，并保持它们的一致性，即使内部细节以后发生变化。API可以是针对特定系统对定制的，也可以是允许许多系统之间互操作性的共享标准。

打个比方，程序员 A 开发了一个程序，里面有个功能是图像识别，然后程序员 B 现在也需要这个功能，但是他不想再走一遍图像识别的路，又不可能直接用别人的程序。这时 A 想到了一个办法，我把这个功能单独从整个程序中抽出来，然后给出了一个入口，别人只要按照约定好的方法来调用，就可以实现这个功能。这就是一个简单的 API 的例子。

#### 什么是REST

**REST**（Representational State Transfer），意为表现层状态转化。

REST 是面向资源的，每个资源都有一个唯一的资源定位符（URI）。每个URI代表一种资源（resource），所以URI中不能有动词，只能有名词，而且所用的名词往往与数据库的表名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以URI中的名词也应该使用复数。

##### 什么是表现层？

指资源的表现层。资源用 URI 标识了，我们可以理解为这个资源已经在网络上“表现”了。表现层就是把"资源"具体呈现出来的形式。

##### 什么是状态转化？

| 请求方式 | 含义                                 |
| -------- | ------------------------------------ |
| GET      | 从服务器取出资源（一项或多项）       |
| POST     | 在服务器新建一个资源                 |
| PUT      | 在服务器更新资源（更新完整资源）     |
| PATCH    | 在服务器更新资源， PATCH更新个别属性 |
| DELETE   | 从服务器删除资源                     |

通过上述方法可以对网络上的资源进行状态转化操作。

所以，REST 就是表现层的状态转化，简单粗暴的可以理解为：方法 + URI资源。

##### 简单对比

| 查询     | 传统                                | REST                            |
| -------- | ----------------------------------- | ------------------------------- |
| 查询所有 | http://localhost:8080/employee/list | http://localhost:8080/employees |

即 GET /employees

```java
@RequestMapping(value = "/employees”, method = RequestMethod.GET)
```

### Docker API

Docker API 也遵循 RESETFUL 风格。

docker官方主要有三大对外api

- Docker Registry API

  镜像仓库的 API，通过操作这套 API，你可以自由的自动化、程序化的管理你的镜像仓库

- Docker Hub API

  用户管理操作的 API，docker hub是使用校验和公共 namespaces 的方式来存储账户信息、认证账户、进行账户授权。API 同时也允许操作相关的用户仓库和 library 仓库。

- Docker Remote API

  控制主机 Docker 服务端的 API，等价于 docker 命令行客户端。 能远程操作docker容器，更重要的是你可以通过程序自动化运维docker进程。

#### Docker API使用前准备

首先要开启 Docker RESET API

```shell
vim /usr/lib/systemd/system/docker.service
```

在`ExecStart=/usr/bin/dockerd`后面直接添加`-H tcp://0.0.0.0:8088  -H unix:///var/run/docker.sock`（为任意可用端口）

然后重启即可。

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

接着我们可以用`curl 127.0.0.1:8088/info | python3 -mjson.tool `来测试成果，得到的是docker的状态，此处不作演示。

#### 使用Docker API

我们可以在 Docker 官网上查询各版本的 API 使用手册：[Docker Engine API v1.41 Reference](https://docs.docker.com/engine/api/v1.41/)

##### curl方式

使用`curl -X GET http://127.0.0.1:8088/images/json | python3 -mjson.tool`查看所有镜像

使用`curl -X GET http://127.0.0.1:8088/containers/json | python3 -mjson.tool`查看所有容器

![image-20220405162023948](https://s2.loli.net/2022/04/05/DSFTNo8btyaJHWQ.png)

##### python程序脚本方式

docker给 python 提供了一个非常强大的库 docker，采用 pip 即可安装。

```shell
pip3 install docker
```

###### 示例

```python
import docker

if __name__ == '__main__':
    client = docker.from_env()
    print(client.containers.run('alpine', 'echo hello world'))
```

![image-20220405184605423](https://s2.loli.net/2022/04/05/TGCnyduZMi3Isbr.png)

这段代码能新建一个容器，仅是一个最简单的测试。

现在我们尝试更多的 API。

```python
import docker

if __name__ == '__main__':
    client = docker.DockerClient("unix://var/run/docker.sock")
    # 拉取镜像
    image = client.images.pull("hello-world")
    # 运行镜像
    # detach=False时，返回运行过程中的日志;detach=True时，返回Container对象
    print(client.containers.run("hello-world", detach=False))
    # 查询镜像
    images = client.images.list()
    for img in images:
        print(img)
    # 查询容器
    for container in client.containers.list():
        print(container.name + " image:" + container.image.attrs["RepoTags"][0])
```

![image-20220405180803411](https://s2.loli.net/2022/04/05/HMS6Nju9VsB3Z5x.png)

## 远程连接docker服务端

前面我们已经介绍了 Unix 域套接字来进行进程间通信以及本地连接 docker，而现在我要进行远程连接 docker。对此，我们可以采用 TCP 连接。

### TCP连接

`/etc/systemd/system/docker.service.d/docker.conf`

```shell
vim /etc/systemd/system/docker.service.d/docker.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

`/etc/docker/daemon.json`

```json
vim /etc/docker/daemon.json
{
  "hosts":[
    "unix:///var/run/docker.sock",
    "tcp://0.0.0.0:2375"
  ]
}
```

重启docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

现在我们用客户端测试

```shell
docker -H 104.225.234.14:2375 info
```

显然这对于远程连接是不安全的，我们可以采用 TLS 连接即 TCP + SSL （HTTPS）来实现安全通信。

### TLS（HTTPS）连接

服务器只允许来自服务器 CA 签名的证书进行身份验证的客户端的连接。客户端仅连接到具有由该 CA 签名的证书的服务器。

#### 使用 OpenSSL 创建 CA、服务器和客户端密钥

1. 首先，在 docker 服务器上，生成 CA 私钥和公钥：

   ```shell
   openssl genrsa -aes256 -out ca-key.pem 4096
   openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
   ```

2. 创建服务器密钥和证书签名请求 （CSR），请确保公用名（common name）与您用于连接到 Docker 的主机名匹配：

   ```shell
   openssl genrsa -out server-key.pem 4096
   openssl req -subj "/CN=localhost" -sha256 -new -key server-key.pem -out server.csr
   ```

3. 使用 CA 对公钥进行签名：

   TLS 连接可以通过 IP 地址和 DNS 名称进行，因此在创建证书时需要指定 IP 地址

   ```shell
   echo subjectAltName = DNS:localhost,DNS:bandwagon,IP:104.225.234.14,IP:127.0.0.1 >> extfile.cnf
   ```

4. 将 Docker 守护程序密钥的扩展用法属性设置为仅用于服务器身份验证：

   ```shell
   echo extendedKeyUsage = serverAuth >> extfile.cnf
   ```

5. 现在，生成签名证书：

   ```shell
   openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
   ```

6. 对于客户端身份验证，请创建客户端密钥和证书签名请求：

   ```shell
   openssl genrsa -out key.pem 4096
   openssl req -subj '/CN=client' -new -key key.pem -out client.csr
   ```

7. 要使密钥适合客户端身份验证，请创建一个新的扩展配置文件：

   ```shell
   echo extendedKeyUsage = clientAuth > extfile-client.cnf
   ```

8. 现在，生成签名证书：

   ```shell
   openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
     -CAcreateserial -out cert.pem -extfile extfile-client.cnf
   ```

6. 生成后，您可以安全地删除两个证书签名请求和扩展配置文件：`cert.pem server-cert.pem`

   ```shell
   rm -v client.csr server.csr extfile.cnf extfile-client.cnf
   ```

7. 如果默认值为 022，则密钥对您和您的组来说是*全局可读*和可写的。`umask`

   若要保护密钥免受意外损坏，请删除其写入权限。要使它们只能由您读取，请按如下方式更改文件模式：

   ```shell
   chmod -v 0400 ca-key.pem key.pem server-key.pem
   ```

8. 证书可以全域可读，但您可能希望删除写入访问权限以防止意外损坏：

   ```shell
   chmod -v 0444 ca.pem server-cert.pem cert.pem
   ```

9. 现在，您可以使 Docker 守护程序仅接受来自提供 CA 信任的证书的客户端的连接：

   ```shell
   dockerd \
       --tlsverify \
       --tlscacert=ca.pem \
       --tlscert=server-cert.pem \
       --tlskey=server-key.pem \
       -H=0.0.0.0:2376
   ```

10. 要连接到 Docker 并验证其证书，请提供您的客户端密钥、证书和受信任的 CA，注意修改 Host：

    ```shell
    docker --tlsverify \
        --tlscacert=ca.pem \
        --tlscert=cert.pem \
        --tlskey=key.pem \
        -H=Bandwagon:2376 version
    ```

{{< admonition info 在客户端计算机上运行它>}}

在客户端计算机上运行它，此步骤应在 Docker 客户端计算机上运行。因此，您需要将 CA 证书、服务器证书和客户端证书复制到该计算机。

{{< /admonition>}}

{{< admonition>}}

基于 TLS 的 Docker 应在 TCP 端口 2376 上运行。

{{< /admonition>}}

{{< admonition warning>}}

如上面的示例所示，使用证书身份验证时，不需要运行客户端或组。这意味着任何拥有密钥的人都可以向您的 Docker 守护程序提供任何指令，从而为他们提供对托管守护程序的计算机的根访问权限。像保护 root 密码一样保护这些密钥！`docker sudo docker`

{{< /admonition>}}

如果要在默认情况下保护 Docker 客户端连接，则可以将文件移动到主目录中的目录---并设置 and 变量（而不是在每次调用时传递）`.docker DOCKER_HOST DOCKER_TLS_VERIFY -H=tcp://$HOST:2376 --tlsverify`

```shell
mkdir -pv ~/.docker
cp -v {ca,cert,key}.pem ~/.docker
export DOCKER_HOST=tcp://localhost:2376 DOCKER_TLS_VERIFY=1
```

默认情况下，Docker 现在安全地连接：

```shell
docker ps
```

当你全部执行上述命令后，此时的文件结构如下：

```shell
├── ca-key.pem
├── ca.pem 			# 客户端&服务端
├── ca.srl
├── cert.pem 		# 客户端
├── key.pem			# 客户端
├── server-cert.pem # 服务端
└── server-key.pem  # 服务端
```

