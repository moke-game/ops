# [cert-manager](https://cert-manager.io/docs/installation/helm/)

* Install cert-manager

  ```shell
  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.5/cert-manager.crds.yaml
  helm repo add jetstack https://charts.jetstack.io --force-update
  helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace 
  ```
* [Install cert-manager csi-driver](https://cert-manager.io/docs/usage/csi-driver/installation/)
    ```shell
    helm repo add jetstack https://charts.jetstack.io --force-update
    helm upgrade -i -n cert-manager cert-manager-csi-driver jetstack/cert-manager-csi-driver --wait
   ```

* [Install aws-privateca-issuer](https://github.com/cert-manager/aws-privateca-issuer)
    ```shell
    # 注意：需要创建新的策略添加到节点组IAM角色中，以便节点可以访问私有CA
     helm repo add awspca https://cert-manager.github.io/aws-privateca-issuer
     helm install awspca/aws-privateca-issuer --generate-name
    ```
* create self pca issuer
    ```shell
    kubectl apply -f ./cert-manager/aws-pca-issuer.yaml
  ```
* create agones allocator mtls certificate
    ```shell
    kubectl apply -f ./cert-manager/agones-allocator-tls.yaml
  ```
* create moke mtls certificate
    ```shell
    kubectl apply -f ./cert-manager/moke-tls.yaml
  ```