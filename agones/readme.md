### [Agones Service](https://agones.dev/site/docs/installation/install-agones/helm/)

* Requirements:
    * [installed agones secrets](../secrets/readme.md)

* 安装agones所需的secrets

  ```shell
  kubectl apply -f ./agones/secrets/allocator-client-ca.yaml
  kubectl apply -f ./agones/secrets/allocator-tls-ca.yaml
  ```

* Install Agones:
   ```shell
   helm repo add agones https://agones.dev/chart/stable
   helm repo update 
   helm install agones --namespace agones-system -f ./agones/values.yaml agones/agones --version 1.40.0 --create-namespace
   ```

* install game servers:
    ```shell
    kubectl apply -f ./agones/fleet-ballte.yaml 
    kubeclt apply -f ./agones/fleet-world.yaml
    ```