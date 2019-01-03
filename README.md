[![Build Status](https://dev.azure.com/pjawesome/winops-london/_apis/build/status/pierluigi.winops-london?branchName=master)](https://dev.azure.com/pjawesome/winops-london/_build/latest?definitionId=2?branchName=master)

# GitHub + Azure “microservices” Baseline Blueprint 
An all-purpose microservices blueprint to kickstart a successful DevOps workflow on Azure accompanying our [associated partner program that can be taken to market for the purposes of lead generation in this space](https://docs.google.com/document/d/1jmaa6zpGj9I8CI4ENKDcsORC-YmxGIl1ar9UYZ_iLEE/edit#).

This repository contains instructions for partners willing to utilize our baseline blueprint to kickstart the creation of bespoke solutions based on GitHub, Azure, Docker (ACR), Kubernetes (AKS).

# Business Value Proposition

The “microservices” blueprint helps our partners visualize what a modern development workflow looks like and how it could be implemented in organizations at scale, using a baseline definition that can be expanded as needed depending on specific requirements. For more info about the business value proposition, please refer to the [Partner Program](https://docs.google.com/document/d/1jmaa6zpGj9I8CI4ENKDcsORC-YmxGIl1ar9UYZ_iLEE/edit#) document.

# Blueprint Description

The goal of this baseline blueprint is to illustrate how teams can collaborate efficiently on different microservice repositories on GitHub and go from pull request to production with guaranteed zero downtime thanks to a blue/green deployment strategy.

Azure Boards allows project managers and collaborators to leverage an agile workflow while tracking all work items for regulatory purposes. The tight integration offered by the AKS managed K8S solution simplifies the deployment and operations of a Kubernetes based “service mesh” and enables teams to dynamically scale the application infrastructure with confidence and agility.

Key components and their respective implementation status:

| Component | Scope | Status |
| --- | --- | --- |
|GitHub | CI/CD, Pull Request checks, Branch Protection | 🔶 |
| Azure DevOps | Build, Push to private registry (ACR), Release via blue/green strategy (AKS) | ✅ |
| Azure Kubernetes Service | Manually configured cluster with instructions | ✅ |
| Azure Kubernetes Service | Automatic provisionin via ARM or Terraform | 🔴 |
| Azure Boards | Project Management and GitHub integration with Work Items, Releases, Commits | 🔴 |
| Azure Application Insights| Monitoring and metrics-based gated rollouts | 🔴 |

Legend: ✅ = Done, 🔶 = WIP, 🔴 = TODO



## Blue/Green and Canary Deployments with Azure DevOps, Istio and AKS

Follow this guide to implement a blue/green deployment strategy using Azure Pipelines targeting a polyglot application deployed to an Azure Kubernetes Cluster using Helm. Istio is used to shape traffic to different versions of the same microservice giving full control on what your users see and controlling the flow of releases throughout the pipeline.

### Deploy AKS and install Helm

```bash
az group create --location centralus --name aks
az aks create -g aks -n mesh -k 1.11.4
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

# Enable automatic sidecar injector
kubectl label namespace default istio-injection=enabled

# Our pods should be in the Running/Completed state
kubectl get pods -n istio-system
kubectl get svc -n istio-system
```

### Deploy all pods at once

```bash
cd manual_deploy
kubectl apply -f .
```
### Verify web app is running

```bash
kubectl get svc -n istio-system
# Find the row corresponding to this:
# NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)
# {...}
# istio-ingressgateway     LoadBalancer   10.0.134.104   168.61.161.70   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:30482/TCP,8060:30740/TCP,853:31204/TCP,15030:31704/TCP,15031:31097/TCP   38m
# {...}
# and paste the EXTERNAL_IP in your browser to load the web app's dashboard. 
# By default it shows a 50/50 Blue/green deployment

```

### Modify blue/green traffic routing

```yaml
# Modify the weights at the end of smackapi-vs.yaml
# ...
  http:
  - route:
    - destination:
        host: smackapi
        subset: blue
      weight: 100 # max out blue
    - destination:
        host: smackapi
        subset: green
      weight: 0 # zero out green
```

Now apply the updated template:

```bash
kubectl apply -f smackapi-vs.yaml
```

Reload the web app to see all blue nodes.

### Build an Azure DevOps Build definition

Create a project and a build pipeline connected to Github and point it to `azure-pipelines.yml`


## Optional

### Install istioctl on mac

```bash
brew tap ams0/istioctl
brew install istioctl
```

### Add a DNS entry for `istio-ingressgateway`

```bash
kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
# Returns your IP
az network dns record-set a add-record -g dns -z <YOUR_FQDN> -n *.mesh --value <IP>
```

### Build and push the web frontend

```bash
cd app/smackweb
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o smackweb
docker build -t ams0/smackweb:0.4 --build-arg BUILD_DATE=`date -u +"%Y-%m-%dT%H%M%SZ"` --build-arg VCS_REF=`git rev-parse --short HEAD` --build-arg IMAGE_TAG_REF=0.4 .
docker push ams0/smackweb:0.4
```


### Credits

Thanks to [Alessandro Vozza's WinOps 2018 talk](https://github.com/ams0/winops-london).


### Useful links

- [Smackapp source](https://github.com/chzbrgr71/microsmackv2)
- [Azure trials](aka.ms/aztrialsuk)
- [Istio](http://istio.io)
