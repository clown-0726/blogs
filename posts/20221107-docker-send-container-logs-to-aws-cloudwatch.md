# Docker - 发送 Container 日志到 AWS CloudWatch

> 发布日期：2022-11-07 16:10:09

关于将 docker 应用程序日志发送到 AWS CloudWatch 的教程。

## 关于

docker 默认推荐将程序运行的日志都输出到标准输出（STDOUT），同时，docker 支持不同的 [log driver](https://docs.docker.com/config/containers/logging/awslogs/#awslogs-region) 可以将日志发送到不同的日志中心。

这篇文章是关于如何配置 docker 容器并其应用程序日志发送到 AWS CloudWatch。发送成功的日志可以从 AWS 管理控制台进行检索。

## 测试环境

虚拟机环境选择的是 ubuntu 的 18.04.2 版本。
```
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 5.4.0-1088-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Nov  7 08:35:49 UTC 2022

  System load:                    0.7
  Usage of /:                     24.5% of 193.81GB
  Memory usage:                   26%
  Swap usage:                     0%
  Processes:                      124
  Users logged in:                0
  IP address for ens5:            172.31.45.149
  IP address for docker0:         172.17.0.1
  IP address for br-eb2f1162c70e: 172.18.0.1
```

docker 的版本为 `Docker version 20.10.21, build baeda1f`。

测试 docker 镜像使用的是 docker 官方的 `alpine:latest`（直接运行输出一句话到 STDOUT 即可）。

## 如何配置

#### 创建 AWS Credentials

现在我们已经准备好了示例应用程序，我们可以将 docker 日志推送到 AWS CloudWatch。为此，我们需要 AWS 帐户的访问凭据以便我们的日志可在 AWS 进行查看。我们将在 AWS 创建一个具有 CloudWatch 访问权限的一个单独的帐户，并将其与 docker daemon（docker 的守护进程）绑定在一起使用。步骤如下：
- 创建 CloudWatch 的 IAM policy
- 使用该 policy 创建 IAM group
- 创建 IAM user 并将其添加到此 group

##### 创建 IAM policy

- 从 AWS 控制台进入 IAM 控制台
- 选择“Policies”，并点击“Create Policy”
- 在“Create Policy”的窗口选择
  - Service = CloudWatch Logs
  - Actions = CreateLogStream, GetLogRecord, DescribeLogGroups, DescribeLogStreams, GetLogEvents, CreateLogGroup, PutLogEvents
  - Resources = All

选择所需权限后，Policies 摘要如下：
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DockerContainerLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:GetLogRecord",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams",
                "logs:GetLogEvents",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

##### 创建 IAM 组和用户
- 在 IAM 控制台创建一个新的 IAM group，并且给一个合适的名字比如“docker-logs-group”
- 将上面创建的 policy 赋给刚创建的 IAM group
- 创建一个新的 IAM user（勾选 Access key - Programmatic access），并且给一个合适的名字比如“docker-logs-user”
- 保存 access key ID 和 secret access key
- 将刚创建的 user 添加到 IAM group

#### 为 docker daemon 配置 AWS credentials

为了配置 docker daemon 使用 AWS credentials，可以在终端执行命令 `sudo systemctl edit docker`。一个新的文本编辑器会打开，添加下面的配置到文本编辑器中。注意，替换 `my-aws-access-key` 和 `my-secret-access-key` 为新创建用户的双 key。

```
[Service]
Environment="AWS_ACCESS_KEY_ID=my-aws-access-key"
Environment="AWS_SECRET_ACCESS_KEY=my-secret-access-key"
```

上面执行的命令会更新凭证 `/etc/systemd/system/docker.service.d/override.conf`。可以通过下面的命令进行验证是否更新成功。

```
$ cat /etc/systemd/system/docker.service.d/override.conf
[Service]
Environment="AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXX"
Environment="AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXXX"
```

对 Docker daemon 进行更改后，我们需要重新启动它：
- 刷新变化 `sudo systemctl daemon-reload`
- 重启 Docker daemon：`sudo systemctl restart docker`

#### 使用 awslogs driver 运行 docker 容器

我们可以使用下面命令运行 docker 进行，并选择使用 awslogs driver：
```bash
sudo docker run \
    --log-driver=awslogs \
    --log-opt awslogs-region=us-west-2 \
    --log-opt awslogs-group=myLogGroup \
    --log-opt awslogs-create-group=true \
    alpine:latest echo $PATH
```

- `log-driver` 配置日志所使用的驱动程序。默认驱动程序为“json-file”，awslogs 用于 CloudWatch
- `awslogs-region` 指定 AWS CloudWatch 日志的 region
- `awslogs-group` 指定 CloudWatch 的日志组
- `awslogs-create-group` 表示如果提供的日志组在 CloudWatch 上不存在，则创建一个

#### 在 CloudWatch 上查看输出日志

在 CloudWatch 的控制台选择 `Log Groups` 然后选择我们上面创建的日志组 `myLogGroup`。

[![xzkXl9.png](https://s1.ax1x.com/2022/11/08/xzkXl9.png)](https://imgse.com/i/xzkXl9)

值得注意的是，在日志组中，默认的 log streams 的名就是 docker container 运行时的 ID，以下是运行测试的三个退出的 container。

```
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS                      PORTS     NAMES
9287238d1b74   alpine:latest   "echo /home/ubuntu/m…"   31 seconds ago   Exited (0) 30 seconds ago             infallible_heyrovsky
4fb16a172fcd   alpine:latest   "echo /home/ubuntu/m…"   32 seconds ago   Exited (0) 31 seconds ago             romantic_zhukovsky
e44d0076e764   alpine:latest   "echo /home/ubuntu/m…"   35 seconds ago   Exited (0) 34 seconds ago             zen_noether
```

## 踩坑日志
#### docker 无法用 systemctl 重启
由于我们是通过 systemctl 来传递 AWS credentials 到 docker daemon 的，因此 systemctl 必须能管理 docker，但是通过命令 `sudo systemctl restart docker` 重启 docker，但是一直报错：
```
Failed to restart docker.service: Unit docker.service not found.
```

这是因为安装 docker 没有使用官方安装包，也没有使用 apt，而是安装 Ubuntu 的时候我选了一同安装 docker，所以其实是用 snap 安装的。

可以通过命令 `snap list` 查看 snap 安装的包，如果存在则卸载掉，然后安装官方的安装包即可。

#### docker 无法找到 AWS credentials
如果 docker daemon 无法找到AWS credentials，那么它将生成一条错误消息，如下所示：
```
docker: Error response from daemon: failed to initialize logging driver: failed to create Cloudwatch log stream: NoCredentialProviders: no valid providers in chain. Deprecated.
        For verbose messaging see aws.Config.CredentialsChainVerboseErrors.
```

如果收到此消息，则需要重新检查传递给 docker daemon 的 credentials 是否正确。

值得注意的是，在 Windows 上，无法将 AWS 凭据传递给 docker daemon。在 MAC OS 上运行 docker 也有类似的问题。有关此讨论，请参阅下面的链接：
- https://github.com/docker/for-win/issues/9684

Docker [官方文档](https://docs.docker.com/config/containers/logging/awslogs/#credentials) 提到可以通过下面两种方式传递 AWS credentials（亲测都不好用）：
- 通过设置 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 到环境变量的方式进行传递。
- 通过使用 AWS credentials 文件，也就是使用创建 `~/.aws/credentials` 文件的方式进行传递。

## 参考文档
[1] Docker - Send Container Logs to AWS CloudWatch https://hassaanbinaslam.github.io/myblog/docker/python/aws/cloudwatch/2022/04/11/docker-logs-cloudwatch.html
[2] Amazon CloudWatch Logs logging driver https://docs.docker.com/config/containers/logging/awslogs/#awslogs-region
