# Deploy manually without Helm

Assuming you have a working cluster with Istio:

```bash
kubectl apply -f .
```

Will deploy web and api, services, virtual services for both, a destination rule for the api and the virtual gateway to expose the web app.