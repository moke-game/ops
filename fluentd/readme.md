# Deploy fluentd

```bash
# Create the configmap and deploy the daemonset
kubectl create configmap fluentd-config --from-file=fluentd/td-agent.conf --namespace=kube-system 
kubectl create -f fluentd/fluentd-daemonset.yaml
```