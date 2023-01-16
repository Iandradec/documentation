---
title: Zulip
category: Deployments
order: 8
---

### How to create a Zulip deployment
This chart bootstraps a <a href="https://github.com/zulip/docker-zulip/tree/main/kubernetes/chart/zulip" target="_blank"> Zulip server </a> deployment on a Kubernetes cluster using the Helm package manager, based on https://github.com/zulip/docker-zulip

* Create and select namespace
``` 
kubectl create ns zulip
kubectl config set-context --current --namespace=zulip
```

* Clone official repository 
```
git clone https://github.com/zulip/docker-zulip.git
```

* Build Helm dependencies
```
cd kubernetes/chart/zulip/                   # Change to template directory
helm dependency update                       # Get helm dependency charts
```

* Create decrypted secrets.yaml file
The following command example creates a new file with zulip and it's dependencies credentials in plain text.
```
    cat <<EOF > secrets-hetzner-dec.yaml
    zulip:
        password: <your_zulip_secret_password>
        environment:
            SECRETS_social_auth_oidc_secret: <your_keycloack_oidc>
            SECRETS_email_password: <your_email_server_password>

    memcached:
        memcachedPassword: <your_memcached_password>

    rabbitmq:
        auth:
            password: <your_rabbitmq_password>
            erlangCookie: <your_erlang_cookie>

    redis:
        auth:
            password: <your_redis_password>

    postgresql:
        auth:
            postgresqlPassword: <your_admin_psql_password>
            password: <your_zulip_psql_password> 
    EOF
```

* Encrypt secrets.yaml file   
The following command will encrypt the previous secrets-hetzner-dec.yaml file using <a href="https://github.com/mozilla/sops" target="_blank"> SOPS: Secrets OperationS </a> and AWS Key Managament Service.
``` 
sops -e  --kms '<YOUR_AWS_KMS_KEY>' secrets-hetzner-dec.yaml > secrets-hetzner.yaml 
```

* Upgrade or install release 
```
helm secrets upgrade --install zulip . -f values-hetzner.yaml -f secrets-hetzner.yaml 
```
> This will show a message on how to reach your Zulip installation and how to create your first realm, wait for all your pods to be ready before you continue.
> Run the commands to create a Realm, and you can reach Zulip following the instructions as well.

```
export POD_NAME=$(kubectl get pods --namespace chat-prod -l "app.kubernetes.io/name=zulip" -o jsonpath="{.items[0].metadata.name}")                         
kubectl -n chat-prod exec -it "$POD_NAME" -c zulip -- sudo -u zulip /home/zulip/deployments/current/manage.py generate_realm_creation_link
```

#### Values

| Key       | Type      | Default       | Description       |
|-----------|-----------|---------------|-------------------|
| affinity | object | `{}` | Affinity for pod assignment. Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity |
| fullnameOverride | string | `""` | Fully override common.names.fullname template. |
| image.pullPolicy | string | `"IfNotPresent"` | Pull policy for Zulip docker image. Ref: https://kubernetes.io/docs/user-guide/images/#pre-pulling-images |
| image.repository | string | `"zulip/docker-zulip"` | Defaults to hub.docker.com/zulip/docker-zulip, but can be overwritten with a full HTTPS address. |
| image.tag | string | `"5.6-0"` | Zulip image tag (immutable tags are recommended) |
| imagePullSecrets | list | `[]` | Global Docker registry secret names as an array. |
| ingress.annotations | object | `{}` | Can be used to add custom Ingress annotations. |
| ingress.enabled | bool | `false` | Enable this to use an Ingress to reach the Zulip service. |
| ingress.hosts[0] | object | `{"host":"zulip.example.com","paths":[{"path":"/"}]}` | Host for the Ingress. Should be the same as `zulip.environment.SETTING_EXTERNAL_HOST`. |
| ingress.hosts[0].paths | list | `[{"path":"/"}]` | Serves Zulip root of the chosen host domain. |
| ingress.tls | list | `[]` | Set a specific secret to read the TLS certificate from. If you use cert-manager, it will save the TLS secret here. If you do not, you need to manually create a secret with your TLS certificate. |
| livenessProbe | object | `{"enabled":true,"failureThreshold":3,"initialDelaySeconds":10,"periodSeconds":10,"successThreshold":1,"timeoutSeconds":5}` | Liveness probe values. Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes |
| memcached | object | `{"memcachedUsername":"zulip@localhost"}` | Memcached settings, see [Requirements](#Requirements). |
| nameOverride | string | `""` | Partially override common.names.fullname template (will maintain the release name). |
| nodeSelector | object | `{}` | Optionally add a nodeSelector to the Zulip pod, so it runs on a specific node. Ref: https://kubernetes.io/docs/user-guide/node-selection/ |
| podAnnotations | object | `{}` | Custom annotations to add to the Zulip Pod. |
| podLabels | object | `{}` | Custom labels to add to the Zulip Pod. |
| podSecurityContext | object | `{}` | Can be used to override the default PodSecurityContext (fsGroup, runAsUser and runAsGroup) of the Zulip _Pod_. |
| postSetup.scripts | object | `{}` | The Docker entrypoint script runs commands from `/data/post-setup.d` after the Zulip application's Setup phase has completed. Scripts can be added here  as `script_filename: <script contents>` and they will be mounted in `/data/post-setup.d/script_filename`. |
| postgresql | object | `{"auth":{"database":"zulip","username":"zulip"},"image":{"repository":"zulip/zulip-postgresql","tag":14},"primary":{"containerSecurityContext":{"runAsUser":0}}}` | PostgreSQL settings, see [Requirements](#Requirements). |
| rabbitmq | object | `{"auth":{"username":"zulip"},"persistence":{"enabled":false}}` | Rabbitmq settings, see [Requirements](#Requirements). |
| redis | object | `{"architecture":"standalone","master":{"persistence":{"enabled":false}}}` | Redis settings, see [Requirements](#Requirements). |
| resources | object | `{}` |  |
| securityContext | object | `{}` | Can be used to override the default SecurityContext of the Zulip _container_. |
| service | object | `{"port":80,"type":"ClusterIP"}` | Service type and port for the Kubernetes service that connects to Zulip. Default: ClusterIP, needs an Ingress to be used. |
| serviceAccount.annotations | object | `{}` | Annotations to add to the service account. |
| serviceAccount.create | bool | `true` | Specifies whether a service account should be created. |
| serviceAccount.name | string | `""` | The name of the service account to use. If not set and create is true, a name is generated using the fullname template |
| startupProbe | object | `{"enabled":true,"failureThreshold":30,"initialDelaySeconds":10,"periodSeconds":10,"successThreshold":1,"timeoutSeconds":5}` | Startup probe values. Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes |
| statefulSetAnnotations | object | `{}` | Custom annotations to add to the Zulip StatefulSet. |
| statefulSetLabels | object | `{}` | Custom labels to add to the Zulip StatefulSet. |
| tolerations | list | `[]` | Tolerations for pod assignment. Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/ |
| zulip.environment.DISABLE_HTTPS | bool | `true` | Disables HTTPS if set to "true". HTTPS and certificates are managed by the Kubernetes cluster, so by default it's disabled inside the container |
| zulip.environment.SECRETS_email_password | string | `"123456789"` | SMTP email password. |
| zulip.environment.SETTING_EMAIL_HOST | string | `""` |  |
| zulip.environment.SETTING_EMAIL_HOST_USER | string | `"noreply@example.com"` |  |
| zulip.environment.SETTING_EMAIL_PORT | string | `"587"` |  |
| zulip.environment.SETTING_EMAIL_USE_SSL | string | `"False"` |  |
| zulip.environment.SETTING_EMAIL_USE_TLS | string | `"True"` |  |
| zulip.environment.SETTING_EXTERNAL_HOST | string | `"zulip.example.com"` | Domain Zulip is hosted on. |
| zulip.environment.SETTING_ZULIP_ADMINISTRATOR | string | `"admin@example.com"` |  |
| zulip.environment.SSL_CERTIFICATE_GENERATION | string | `"self-signed"` | Set SSL certificate generation to self-signed because Kubernetes manages the client-facing SSL certs. |
| zulip.environment.ZULIP_AUTH_BACKENDS | string | `"EmailAuthBackend"` |  |
| zulip.persistence | object | `{"accessMode":"ReadWriteOnce","enabled":true,"size":"10Gi"}` | If `persistence.existingClaim` is not set, a PVC is generated with these specifications.

## Uninstall release
For a complete uninstall, make sure to delete pvc created.

``` 
helm uninstall $RELEASE_NAME
kubectl delete pvc $RELEASE_NAME-pvc
```