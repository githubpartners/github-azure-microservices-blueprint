[![Build Status](https://cookingwithazure.visualstudio.com/winops/_apis/build/status/winops-CI)](https://cookingwithazure.visualstudio.com/winops/_build/latest?definitionId=23)

# winops-london
Repository to hold my demo files for [my talk](https://www.winops.org/london/agenda/bluegreen.php) at WinOps London 2018


## Blue/Green and Canary Deployments with Azure DevOps, Istio and AKS

Room: CTRL

Talk Time: 2:45 - 3:30pm

In this demo-driven talk I will show how you can implement advanced DevOps concepts like blue/green and canary deployment using Azure Pipelines targeting a polyglot application deployed to an Azure Kubernetes Cluster using Helm. Istio is used to shape traffic to different versions of the same microservice giving full control on what your users see and controlling the flow of releases throughout the pipeline.

### Deploy AKS and install Helm

```bash
az aks create -g aks -n mesh -n 1.11.4
az aks get-credentials -g aks -n mesh

#deploy helm with muy fancy one-liner
kubectl create serviceaccount -n kube-system tiller; kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller; helm init --service-account tiller
```

### Install istio

```bash
curl -L https://git.io/getLatestIstio | sh -
cd istio-1.0.3
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/istio-demo-auth.yaml

kubectl label namespace default istio-injection=enabled

#check all is good:
kubectl get pods -n istio-system
kubectl get svc -n istio-system
```

### Install istioctl on mac

```bash
brew tap ams0/istioctl
brew install istioctl
```

### Add a DNS entry for `istio-ingressgateway`

```bash
kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
az network dns record-set a add-record -g dns -z cookingwithazure.com -n *.mesh --value <IP>
```

### Demo flow

- [ ] Deploy AKS
- [ ] Install Helm
- [ ] Install Istio
- [ ] Deploy app
- [ ] Watch Azure DevOps



### Useful links

[Smackapp source](https://github.com/chzbrgr71/microsmackv2)
[Azure trials](aka.ms/aztrialsuk)
[Isti](http://istio.sh)o