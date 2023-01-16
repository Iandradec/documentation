---
title: Welcome
---

![](infrastructure.png)

## Prerequisites Setup
* Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
  curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256" && echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check && \
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && kubectl version --client --output=yaml
```

* Helm 3.0.0+
```  
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null && \
  sudo apt-get install apt-transport-https --yes && \
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list && \
  sudo apt-get update && sudo apt-get install helm  
```

* Helm upgrade plugin diff
```
helm plugin install https://github.com/databus23/helm-diff
```
* Fresh deployment, must create and select namespace
``` 
kubectl create ns $NAMESPACE  
kubectl config set-context --current --namespace=$NAMESPACE
```
* envsubst
  * macos:
    `brew install gettext`
  * debian/ubuntu:
    `apt-get install gettext-base`

## How to create a simple PostgresSQL deployment
This chart bootstraps a PostgreSQL deployment using the Helm package manager.

### Create and select namespace
``` 
kubectl create ns postgres  
kubectl config set-context --current --namespace=postgres
```
  
### Create a new Helm chart
```
helm create postgres-chart
```

### Create decrypted secrets.yaml file
The following command example creates a file with credentials for postgresql in plain text.

  ```
  cat <<EOF > secrets-dec.yaml
  db:
    user: super_user_example
    password: super_pass_example
  EOF
  ```

### Encrypt secrets.yaml file   
The following command will encrypt the previous secrets-dec.yaml file using <a href="https://github.com/mozilla/sops" target="_blank"> SOPS: Secrets OperationS </a> and AWS Key Managament Service.

``` sops -e  --kms '<YOUR_AWS_KMS_KEY>' secrets-dec.yaml > secrets.yaml```

### Upgrade or install release
Refer to values.yaml for the database values:
```yaml
---
RELEASE_NAME: <YOUR_RELEASE_NAME>
CHART_NAME: <YOUR_CHART_NAME>
---
```

```helm secrets upgrade --install $RELEASE_NAME $CHART_NAME -f raw.values.yaml -f secrets.yaml ```
  
### Access to postgresql
  ```kubectl exec -it < POSTGRESQL-POD-NAME > -- bash ```
  
### Uninstall release
For a complete uninstall, make sure to delete pvc created.

``` 
helm uninstall $RELEASE_NAME
kubectl delete pvc $RELEASE_NAME-pvc
```

## Deployments on Hetzner cloud
This section shows more in detail on how to deploy each app on dyvenia infrastructure.

### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/datahub" target="_blank"> Datahub deployment</a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/cert-manager" target="_blank"> Cert-manager deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/ingress" target="_blank"> Ingress Controller deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/postgresql" target="_blank"> PostgreSQL deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/privatebin" target="_blank"> PrivateBin deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/timeflow" target="_blank"> Timeflow deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/oauth2" target="_blank"> Proxy-oauth2 deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/keycloak" target="_blank"> Keycloak deployment </a>
### <a href="https://github.com/dyvenia/infrastructure/tree/main/deployments/zulip" target="_blank"> Zulip deployment </a> 
<!-- 
This is the **Edition** template from [CloudCannon](http://cloudcannon.com/).
**Edition** is perfect for documenting your product, application or service.
It's populated with example content to give you some ideas.

ChatApp is a fictional chat application for sending messages and media to others.
Teams and friend groups would use ChatApp to stay up to date if it existed.

> [Sign up](http://example.com/signup) or learn more about ChatApp at [example.com](http://example.com/).

### Getting Started

Getting a message sent is quick and easy with ChatApp:

1. Sign up for an account
2. Add your friends from their email addresses
3. Type a message or send a photo

> Feel free to send us a message at [feedback@example.com](mailto:feedback@example.com) with your feedback.

### Features

Explore more of ChatApp by reading about our features:

#### Media

Send images, videos and other media to people. Sources include your computer, phone and Facebook.

#### Contact Syncing

Sync your contact list with your phone and/or Facebook contacts. Never lose your contacts between devices again!

#### Devices

ChatApp is available everywhere. Find out how to set it up on your all your devices. -->
