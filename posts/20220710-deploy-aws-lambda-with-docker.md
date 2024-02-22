# 在 AWS lambda 上部署 docker 应用

> 发布日期：2022-07-10 21:23:34

## 前言

使用 AWS lambda，我们可以在不考虑服务器的情况下上传代码并运行，但是这种方式最大的缺点就是代码包的大小限制，每一个 lambda 函数的代码包不能高于几十M。

现在，AWS lambda 允许以 docker 容器的方式运行，每一个 docker image 的大小允许高达 10G。通过这种方式，还可以轻松构建和部署依赖于较大依赖的工作任务，例如机器学习或数据密集型工作任务。与打包为 ZIP 部署的函数的功能一样，容器化部署的功能也具有相同的操作简单性、自动扩展、高可用性以及与许多服务的天然集成。

AWS lambda 提供了两种方式来部署容器，一种是基于 AWS 提供的镜像的部署方式，另一种是自定义镜像的部署方式，下面我们将对这种中方式的不同和使用做出阐述。

## 准备工作

在进行镜像部署之前，首先应该准备好以下几点：

- 拥有合规的 AWS 的账号
- 为每个将要部署的 lambda 函数提供一个 ECR（Elastic Container Registry）

## 基于 AWS 提供的镜像的部署方式

这种方式的最大好处就是操作比较简单，我们只需要把自己的代码复制到容器中并安装依赖即可。

#### 选择镜像

选择 AWS 官方提供的 python 3.7 镜像 `public.ecr.aws/lambda/python:3.7`。我们可以在 docker hub 上搜索 `amazon/aws-lambda` 来搜索其他语言的官方镜像。

#### 定义 Dockerfile

定义代码结构如下：

```
project
	-- app.py
	-- Dockerfile
	-- requirements.txt
```

这是一个最简单的代码结构，app.py 是函数的运行入口，我们需要按照 lambda 提供的运行模版来定义：

```python
import sys

def handler(event, context):
    print('Hello from AWS Lambda using Python' + sys.version + '!')
    return {}
```

requirements.txt 是 python 程序的依赖包

最后我们定义最重要的 Dockerfile 文件：

```dockerfile
# 选择官方镜像
FROM public.ecr.aws/lambda/python:3.7

# 复制所有的代码到既定的目录下
# ${LAMBDA_TASK_ROOT} --> /var/task
COPY . ${LAMBDA_TASK_ROOT}

WORKDIR ${LAMBDA_TASK_ROOT}

# 安装程序的依赖包
RUN pip install -r requirements.txt

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD [ "app.handler" ]

```

#### 测试代码

由于 AWS 提供的基础镜像已经默认集成了运行时接口模拟器（Runtime Interface Emulator），我们可以直接通过下面命令启动一个容器：

```bash
docker run --rm -p 9000:8080 <IMAGE_NAME>:latest
```

容器运行起来之后我们可以通过 http 请求的方式进行测试请求：

```bash
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"payload":"hello world!"}'\n
```

## 基于自定义的镜像部署方式

有时，您需要使用自定义容器镜像。例如，为了遵循公司的指导原则或使用 AWS 不支持的运行时版本。

#### 选择镜像

这里我们选择 python 的官方镜像 `python:3.7`，我们也可以基于这个镜像预先安装一些常用的依赖包，这样我们就可以基于自己的镜像进行构建了。

#### 定义 Dockerfile

定义代码结构如下：

```
project
	-- app.py
	-- Dockerfile
	-- requirements.txt
	-- entry.sh
```

相对于基于官方镜像构建的方式，项目目录中只是多了一个 `entry.sh` 文件。

app.py 是函数的运行入口，我们仍然需要按照 lambda 提供的运行模版来定义：

```python
import sys

def handler(event, context):
    print('Hello from AWS Lambda using Python' + sys.version + '!')
    return {}
```

requirements.txt 是 python 程序的依赖包

最后我们定义最重要的 Dockerfile 文件，与上面不同的是，我们需要自己安装 AWS lambda 运行需要的依赖包：

```dockerfile
# 定义全局变量
ARG FUNCTION_DIR="/app/"
ARG RUNTIME_VERSION="3.7"

# 选择自定义的 base 镜像
FROM python:${RUNTIME_VERSION}

# 安装 lambda 运行接口客户端（Lambda Runtime Interface Client）
RUN pip install awslambdaric

ARG FUNCTION_DIR

# 创建项目运行目录
RUN mkdir -p ${FUNCTION_DIR}
# 复制所有代码到项目运行目录
COPY . ${FUNCTION_DIR}

WORKDIR ${FUNCTION_DIR}

# 安装项目依赖包
RUN pip install -r requirements.txt

# (可选) 
# 添加 lambda 运行时接口模拟器（Lambda Runtime Interface Emulator）
# 在 ENTRYPOINT 中添加一个本地运行的脚本
ADD https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie /usr/bin/aws-lambda-rie
COPY entry.sh /
RUN chmod 755 /usr/bin/aws-lambda-rie /entry.sh
ENTRYPOINT [ "/entry.sh" ]
CMD [ "app.handler" ]
```

#### 测试代码

由于构建好的镜像中集成了运行时接口模拟器（Runtime Interface Emulator），我们可以直接通过下面命令启动一个容器：

```bash
docker run --rm -p 9000:8080 <IMAGE_NAME>:latest
```

容器运行起来之后我们可以通过 http 请求的方式进行测试请求：

```bash
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"payload":"hello world!"}'\n
```

## 上传部署

文章开头提高我们需要为每一个运行 docker 容器的 lambda 提供一个 ECR（Elastic Container Registry），用于上传定义好的镜像文件。假设我们已经创建好一个 ECR 仓库，这个仓库的地址为 `100000000009.dkr.ecr.us-west-2.amazonaws.com/test-la-pure-func` ，我们依据下面的步骤上传定义好的镜像到这个仓库。

###### 构建 image

```bash
sudo docker build -t 100000000009.dkr.ecr.us-west-2.amazonaws.com/test-la-pure-func:latest .
```

###### 登录 Registry

> 运行这一步必须在电脑安装安装好 aws-cli，并且已经做好相应的配置

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin 100000000009.dkr.ecr.us-west-2.amazonaws.com
```

###### 推动 image

```bash
docker push 100000000009.dkr.ecr.us-west-2.amazonaws.com/test-la-pure-func:latest
```

###### 部署 lambda

建立 lambda 的时候只需要选择使用 docker 运行的方式，然后选择相应的镜像即可。

## 参考文献

[1] Deploy Python Lambda functions with container images https://docs.aws.amazon.com/lambda/latest/dg/python-image.html#python-image-create-alt  
[2] New for AWS Lambda – Container Image Support https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/  
[3] https://hub.docker.com/search?q=amazon%2Faws-lambda