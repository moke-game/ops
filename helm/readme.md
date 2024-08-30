# moke-game k8s 集群配置

## Requirements:

* [AWS CLI install](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-configure.html)
* [kubectl install](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/create-kubeconfig.html)
* [Helm install](https://helm.sh/docs/intro/install/)

## 集群访问/管理：

 ```shell
   aws configure // 配置aws cli 需要相关权限文件
   aws eks update-kubeconfig --name {ClusterName} --region ap-southeast-1
   winget install k9s // 安装k9s
   k9s // 查看集群状态
 ```

## 部署

### 部署configmap

```shell
kubectl apply -f ./configmap/dev-config.yaml
```

## 部署service

```shell
# fix {appname} to service name
# 这里建议使用Jenkins/argoCD自动化发布服务
helm upgrade --install {appname} ./helm/base  -f ./helm/{appname}/values.yaml 
```