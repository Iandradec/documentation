---
title: Ingress
category: Deployments
order: 3
---

<!-- ## How to create a ingress controller deployment -->
###
This chart bootstraps an <a href="https://kubernetes.github.io/ingress-nginx/" target="_blank"> Ingress Controller </a>  using the Helm package manager. 

* Create and select namespace
```
kubectl create ns ingress-nginx
kubectl config set-context --current --namespace ingress-nginx
```

* Add Helm repo
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
```

* Upgrade or install release
``` 
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx -f ingress/values.yaml  
```

* Uninstallation
```
helm uninstall ingress-nginx  
```