# Deploy manually without Helm

Assuming you have a working cluster with Istio:

```bash
kubectl apply -f smackweb.yaml
kubectl apply -f smackapi.yaml
```