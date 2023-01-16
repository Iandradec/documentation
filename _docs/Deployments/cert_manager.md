---
title: Cert Manager 
category: Deployments
order: 1
---

# How to create a cert manager deployment
This chart bootstraps a <a href="https://artifacthub.io/packages/helm/cert-manager/cert-manager" target="_blank"> cert-manager addon </a> using the Helm package manager. 

* Create and select namespace
```
kubectl create ns cert-manager
kbectl config set-context --current --namespace cert-manager
  ```

* Add Helm repo
``` 
helm repo add jetstack https://charts.jetstack.io 
```

* Upgrade or install release
```
helm upgrade --install cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true 
```
  
* Create cert-issuer
```
kubectl apply -f cert-manager/letsencrypt-production.yaml
kubectl apply -f cert-manager/letsencrypt-staging.yaml
```

* Uninstallation
``` 
helm uninstall cert-manager 
```