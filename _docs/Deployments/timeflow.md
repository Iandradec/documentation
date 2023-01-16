---
title: Timeflow
category: Deployments
order: 7
---

### Components
Timeflow is a web application build with 3 main components:

* Local DB (PostgreSQL)
* Backend (FastAPI)
* Frontend (SvelteKit web framework)

### Prerequisites setup
* <a href="https://iandradec.github.io/documentation/index.html" target="_blank"> Install kubectl </a> 
* <a href="https://iandradec.github.io/documentation/index.html" target="_blank"> Helm 3.0.0+ </a> 
* <a href="https://svelte.dev/" target="_blank"> SvelteKit web framework </a>
* <a href="https://fastapi.tiangolo.com/" target="_blank"> FastAPI </a>
* <a href="https://github.com/mozilla/sops" target="_blank"> Mozilla SOPS </a>

### How to create a Timeflow deployment
* Create and select namespace
``` 
kubectl create ns timeflow  
kubectl config set-context --current --namespace=timeflow
```
* Create a new Helm chart
```
helm create timeflow
```
* Create decrypted secrets.yaml file
The following command example creates a file with secrets variables in plain text.
  ```
  cat <<EOF > secrets-dec.yaml
  db:
      extraEnvs:
          - name: POSTGRES_USER
            value: <psql_user_example>
          - name: POSTGRES_PASSWORD
            value: <psql_password_example>
          - name: POSTGRES_DB
            value: <psql_db_name_example>
      fastapi:
          extraEnvs:
              - name: SECRET_KEY
                value: <backend_secret_key_example>
  EOF
  ```
* Encrypt secrets.yaml file   
The following commands will manage the encryption of previous secrets-dec.yaml file using <a href="https://github.com/mozilla/sops" target="_blank"> SOPS: Secrets OperationS </a> and AWS Key Managament Service.
``` 
sops -e  --kms '<YOUR_AWS_KMS_KEY>' secrets-dec.yaml > secrets.yaml 
```

* Install release
```
helm secrets install timeflow . -f values-hetzner.yaml -f secrets-hetzner.yaml
```

* Upgrade release 
```
helm secrets upgrade timeflow . -f values-hetzner.yaml -f secrets-hetzner.yaml 
```
 
* Uninstall release
For a complete uninstall, make sure to delete pvc created.
``` 
helm uninstall $RELEASE_NAME
kubectl delete pvc $RELEASE_NAME-pvc
```

