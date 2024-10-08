# moke-game EKS 集群

## Architecture

![image](./draws/moke.drawio.png)

## GameFlow

![image](./draws/gameflow.drawio.png)

## Requirements:

* [AWS CLI install](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-configure.html)
* [kubectl install](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/create-kubeconfig.html)
* [Helm install](https://helm.sh/docs/intro/install/)

## 集群创建流程

* [EKS Cluster Create](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/create-cluster.html)
    * IAM 角色权限策略： AmazonEKSClusterPolicy、AmazonEKS_CNI_Policy,AmazonEBSCSIDriverPolicy
* [Amazon EBS CSI install](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/ebs-csi.html)
* [NodeGroup Create](https://docs.aws.amazon.com/zh_cn/eks/latest/userguide/create-managed-node-group.html)
    * IAM 角色权限策略：
      AmazonEKSWorkerNodePolicy、AmazonEC2ContainerRegistryReadOnly、AmazonEKS_CNI_Policy、AmazonEBSCSIDriverPolicy

## 集群访问

### 配置权限:

 ```shell
   aws configure // 配置aws cli 需要相关权限文件
   aws eks update-kubeconfig --name moke-prod --region ap-southeast-1
   winget install k9s // 安装k9s
   k9s // 查看集群状态
 ```

### Infrastructure deployment:

* Install [mongodb](https://artifacthub.io/packages/helm/bitnami/mongodb)
   ```shell
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update 
   helm install mongo oci://registry-1.docker.io/bitnamicharts/mongodb 
    ```
* Install [redis](https://artifacthub.io/packages/helm/bitnami/redis)
   ```shell
   helm install redis oci://registry-1.docker.io/bitnamicharts/redis 
    ```
* Install [Nats](https://artifacthub.io/packages/helm/bitnami/nats)
   ```shell
   helm repo add nats https://nats-io.github.io/k8s/helm/charts/
   helm repo update 
   helm upgrade --install nats nats/nats
    ```
* Install [ingress-nginx](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)
    ```shell
       helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
       helm repo update 
       helm install nginx -f .\ingress-nginx\values.yaml ingress-nginx/ingress-nginx 
    ```

## 部署cert-manager

[点击查看详细描述](./cert-manager/readme.md)

## 部署 agones

* [点击查看详细描述](./agones/readme.md)

## 部署 moke services:

* 部署相关secrets，类似于iap-secret,数据库密码等
   ```shell
   #iap-secret.yaml比较敏感，不要上传到git仓库
   kubectl apply -f ./iap/iap-secret.yaml
   kubectl apply -f ./secret.yaml
   ```

## 发布流程：

1. 登录私有docker registry

```shell
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin <your docker private registry>
```

2. 构建镜像

```shell
# fix {appname} to service name
# 这里建议使用Jenkins/GitHub Actions自动化构建镜像
docker buildx build -t {appname}<your docker private registry>:{version} --build-arg APP_NAME={appname} -f ./build/package/docker/Dockerfile .  --push
```

3. 发布服务

* TODO：通过configmap配置helm values.yaml 以实现不同环境下走不同的配置信息
* 修改helm/base/Chart.yaml中的version,appVersion(自增)

```shell
# fix {appname} to service name
# 这里建议使用Jenkins/argoCD自动化发布服务
helm upgrade --install {appname} ./helm/base  -f ./helm/{appname}/values.yaml 
```

### 问题排查

1. Error saving credentials: error storing credentials - err: exit status 1,
   out: `error storing credentials - err: exit status 1, out: `The stub received bad data.

* 删除 用户/.docker/config.json 文件中 { "credsStore": "desktop" }属性