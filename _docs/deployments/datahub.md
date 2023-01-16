---
title: Datahub
category: deployments
order: 2
---

## How to create a datahub deployment
This chart bootstraps <a href="https://github.com/acryldata/datahub-helm" target="_blank"> Datahub </a> and it's dependencies (Elasticsearch, optionally Neo4j, MySQL, and Kafka) on a Kubernetes cluster using the Helm package manager. 

## Components
Datahub consists of 4 main components: GMS, MAE Consumer (optional), MCE Consumer (optional), and Frontend. The main components are powered by 4 external dependencies, and must be deployed before deploying Datahub:

* Kafka
* Local DB (MySQL, Postgres, MariaDB)
* Search Index (Elasticsearch)
* Graph Index (Supports either Neo4j or Elasticsearch)
 
* Create and select namespace
```
kubectl create ns datahub
kubectl config set-context --current --namespace datahub
```
  
* Add Helm repo
``` helm repo add datahub https://helm.datahubproject.io/ ```

* Create prerequisites secrets
The following command creates prerequisites credentials as secrets for a fresh deployment. 

>   * MYSQL_ROOT_PASSWORD: <YOUR_MYSQL_PASSWORD>
>   * NEO4J_ROOT_PASSWORD: <YOUR_NEO4J_PASSWORD>

``` 
kubectl create secret generic mysql-secrets --from-literal=mysql-root-password=$MYSQL_ROOT_PASSWORD
kubectl create secret generic neo4j-secrets --from-literal=neo4j-password=$NEO4J_ROOT_PASSWORD
```

* Create decrypted secrets.yaml file
The following command example creates a new file with datahub frontend credentials in plain text.

 ```
  cat <<EOF > frontend-creds-dec.yaml
  datahub-frontend:
    extraEnvs:
    - name: DATAHUB_ROOT_USERNAME
      value: < super_username_example >
    - name: DATAHUB_ROOT_PASSWORD
      value: < super_root_password_example >
    oidcAuthentication:
        clientId: < google_client_id >
        clientSecret: < google_client_secret >
  EOF
  ```

* Encrypt secrets.yaml file   
The following command will encrypt the previous frontend-creds-dec.yaml file using <a href="https://github.com/mozilla/sops" target="_blank"> SOPS: Secrets OperationS </a> and AWS Key Managament Service.

``` 
sops -e  --kms '<YOUR_AWS_KMS_KEY>' frontend-creds-dec.yaml > frontend-creds.yaml 
```

* Verify release upgrade
``` 
helm diff upgrade prerequisites datahub/datahub-prerequisites -f helm_values/values-prerequisites.yaml --no-hooks
helm diff upgrade datahub datahub/ -f helm_values/values-hetzner.yaml --no-hooks
```

* Upgrade release
``` 
helm upgrade --install prerequisites datahub/datahub-prerequisites -f helm_values/values-prerequisites.yaml
helm secrets upgrade --install datahub datahub/ -f helm_values/values-hetzner.yaml -f helm_secrets/frontend-creds.yaml --no-hooks
```
  
* Install release 
```
helm install prerequisites datahub/datahub-prerequisites -f helm_values/values-prerequisites.yaml
helm secrets --install datahub datahub/ -f helm_values/values-hetzner.yaml -f helm_secrets/frontend-creds.yaml --no-hooks
```

* Uninstall release
For a complete uninstall, make sure to delete pvc created.

``` 
helm uninstall $RELEASE_NAME
kubectl delete pvc $RELEASE_NAME-pvc
```