# k8s 维护记录

> 发布日期：2023-09-12 19:03:35

<img src="https://s1.ax1x.com/2023/09/12/pP2VThR.jpg" style="zoom:0%;" />



## 维护日志

#### EKS AWS Load Balancer Controller 无法创建新的 ALB Ingress

问题描述：

AWS EKS 部署了 AWS Load Balancer Controller 用来创建 ALB Ingress，提供对外的服务访问。当时用了一段时间之后，突然无法创建了，通过命令 `kubectl describe xxx-ing` 检查要创建的 `xxx-ing` Ingress 的事件发现下面错误。

```tex
eksctl-eksprod2-addon-iamserviceaccount-kube-Role1-IE0RPT302EC6/1694481770336105310 is not authorized to perform: elasticloadbalancing:AddTags on resource: arn:aws:elasticloadbalancing:us-west-2:675525421289:targetgroup/k8s-default-orbitext-80a4338d7c/* because no identity-based policy allows the elasticloadbalancing:AddTags action
           status code: 403, request id: 50a86c39-16e1-45b4-b325-a0dd8d003e01
  Warning  FailedDeployModel  6m11s  ingress  Failed deploy model due to AccessDenied: User: arn:aws:sts::675525421289:assumed-role/eksctl-eksprod2-addon-iamserviceaccount-kube-Role1-IE0RPT302EC6/1694481770336105310 is not authorized to perform: elasticloadbalancing:AddTags on resource: arn:aws:elasticloadbalancing:us-west-2:675525421289:targetgroup/k8s-default-orbitext-80a4338d7c/* because no identity-based policy allows the elasticloadbalancing:AddTags action
           status code: 403, request id: ee411302-9d8d-48c0-a1d7-cbbe87088c0f
......
```

问题分析：

上述原因是 AWS Load Balancer Controller 没有对应的权限操作 AWS 得 Load Balancer。但是之前的却是好用的，考虑应该是 AWS 内部的升级导致的。

解决方案是找到之前的。IAM Policy(AWSLoadBalancerControllerIAMPolicy) 并更新其新的权限控制。但是如何得到最新的权限控制呢？可以去最新的[AWS Load Balancer Controller 安装文档](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/deploy/installation/)中只操作 Configure IAM 这一部分，这样会在 AWS 的 IAM 中建立出新的 IAM Policy，将新的 IAM Policy 的规则复制到之前的 IAM Policy 规则中保存即可。

参考文档：

[1] AWS Load Balancer Controller https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/deploy/installation/

[2] Troubleshooting IAM https://docs.aws.amazon.com/eks/latest/userguide/security_iam_troubleshoot.html

#### How To Fix The “Cannot Autolaunch D-Bus Without X11 $DISPLAY” Error

问题描述：

从 AWS ECR 拉取镜像的时候，往往需要先进行登录操作，但是有时候会因为当前机器上上没有适当的 credential helpers 而报下面的错误。

```tex
Error saving credentials: error storing credentials - err: exit status 1, out: Cannot autolaunch D-Bus without X11 $DISPLAY
```

问题分析：

需要在当前机器上安装适当的 credential helpers。

By default, Docker looks for the native binary on each of the platforms; i.e., “osxkeychain” on macOS, “wincred” on windows, and “pass” on Linux. A special case is that on Linux, Docker will fall back to the “secretservice” binary if it cannot find the “pass” binary.

参考文档：

[1] How To Fix The “Cannot Autolaunch D-Bus Without X11 $DISPLAY” Error https://anto.online/guides/cannot-autolaunch-d-bus-without-x11-display/
