- [DockerFile](#dockerfile)
- [Docker镜像使用](#docker----)
  * [列出镜像列表](#------)
  * [创建镜像](#----)
  * [设置镜像标签](#------)
  * [删除镜像](#----)
- [Docker容器使用](#docker----)
  * [查看所有的容器](#-------)
  * [启动容器](#----)
  * [后台运行](#----)
  * [停止容器](#----)
  * [进入容器](#----)
  * [删除容器](#----)
  * [容器启动为web应用](#-----web--)
  * [查看 WEB 应用程序日志](#---web-------)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

### DockerFile

指令含义

- **FROM**：

  定制的镜像都是基于 FROM 的镜像（可以基于Docker Hub，也可基于自己的阿里云镜像仓库）

- **RUN**

  用于执行后面跟着的命令行命令。因为每运行一次，都会生成一次layer，避免无用分层，合并多条命令成一行 反斜线\换行 &&合并成一行

- **WORKDIR**

  指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

  docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

- **COPY**

  复制指令，从上下文目录中复制文件或者目录到容器里指定路径

- **CMD**

  用于运行程序，格式推荐:

   ```CMD ["<可执行文件或命令>","<param1>","<param2>",...] ```

- **EXPOSE**

  仅仅只是**声明**端口。

  作用：

  - 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
  - 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口

```dockerfile
# 基于Docker Hub中的python3.6镜像
FROM python:3.6  
RUN mkdir -p /opt/apps/hello
# 将debian的源换成阿里云的
RUN sed -i 's#http://deb.debian.org#http://mirrors.aliyun.com#g' /etc/apt/sources.list && sed -i 's#http://security.debian.org#http://mirrors.aliyun.com#g' /etc/apt/sources.list
# 更新、安装包（注意：install后面要加-y，不然打包镜像会报错）
RUN apt-get update -y && apt-get install -y libzbar0 libzbar-dev
COPY requirements.txt /opt/apps/hello/requirements.txt
COPY . /opt/apps/hello
WORKDIR  /opt/apps/hello
# 使用清华源安装python依赖包
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
# docker内的应用要运行在0.0.0.0上，不能运行在127.0.0.1，不然docker外无法请求docker内的应用
CMD ["python3", "app.py", "--host=0.0.0.0"]
```

### Docker镜像使用

#### 列出镜像列表

```$ docker images```

#### 创建镜像

当我们从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

- 1、从已经创建的容器中更新镜像，并且提交这个镜像
- 2、使用 Dockerfile 指令来创建一个新的镜像

这里介绍第二种方法

```$ docker build -t hello_flask:0.1 .```

参数说明：

- **-t** ：指定要创建的目标镜像名，冒号后面是镜像的TAG
- **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

#### 设置镜像标签

我们可以使用 docker tag 命令，为镜像添加一个新的标签。

```$ docker tag 860c279d2fec hello_flask:0.2```

参数说明：

- 860c279d2fec为镜像的IMAGE ID
- hello_flask:0.2为新的镜像名和tag

#### 删除镜像

删除名为hello_flask，tag为0.1的镜像：

```$ docker rmi hello_flask:0.1```

### Docker容器使用

#### 查看所有的容器

```$ docker ps -a```

#### 启动容器

1. 通过镜像启动

   ```$ docker run -it hello_flask:0.1 /bin/bash```

   参数说明：

   - **-i**：交互式操作。
   - **-t**：终端。
   - **-d**：后台运行
   - **hello_flask:0.1**：hello_flask:0.1 镜像。
   - **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

   要退出终端，直接输入 **exit**:

2. 启动已停止运行的容器

   ```$ docker start <容器 ID>``` ```

   b750bbbcfd88 为容器id

#### 后台运行

在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 -d 指定容器的运行模式。

```$ docker run -itd hello_flask:0.1 /bin/bash```

**注：**加了 -d 参数默认不会进入容器，想要进入容器需要使用指令 **docker exec**（下面会介绍到）。

#### 停止容器

```$ docker stop <容器 ID>```

停止的容器可以通过 docker restart 重启：

```$ docker restart <容器 ID>```

#### 进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

1. attach 命令

   ```$ docker attach <容器 ID>```

   **注意：** 如果从这个容器使用exit命令退出，会导致容器的停止。

2. exec 命令

   ```$ docker exec -it <容器 ID> /bin/bash```

#### 删除容器

```$ docker rm -f <容器 ID>```

#### 容器启动为web应用

```$ docker run -d -p 8000:5000 --name hello_app hello_flask:0.1 ```

参数说明：

- **-d**：后台运行。
- **-p**：设置监听端口。docker开放5000端口映射到主机端口8000上
- hello_app为自定义的容器名称
- hello_flask:0.1 为镜像id与tag

#### 查看 WEB 应用程序日志

可以通过错误日志定位容器**启动失败**的原因

docker logs [容器ID或者名字] 可以查看容器内部的标准输出。

```$ docker logs -f <容器id 或 容器名称>```

**-f:** 让 **docker logs** 像使用 **tail -f** 一样来输出容器内部的标准输出。