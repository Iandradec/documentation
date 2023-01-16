---
title: Oauth2 proxy
category: Deployments
order: 5
---

### How to create a oauth2-proxy deployment
This chart bootstraps a <a href="https://github.com/oauth2-proxy/manifests/tree/main/helm/oauth2-proxy" target="_blank"> oauth2-proxy </a> deployment on a Kubernetes cluster using the Helm package manager.

<a href="https://github.com/oauth2-proxy/manifests/tree/main/helm/oauth2-proxy" target="_blank"> oauth2-proxy </a> is a reverse proxy and static file server that provides authentication using Providers (Google, GitHub, and others) to validate accounts by email, domain or group.

* Create and select namespace
```bash
kubectl create ns proxy  
kubectl config set-context --current --namespace=proxy
```

* Add Helm repo
```bash 
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests 
```

* Upgrade or install release
Refer to values.yaml for the deployment values:
```bash
---
RELEASE_NAME: <your_release_name>
CHART_NAME: <your_chart_name>
---
```

```bash
helm install $RELEASE_NAME $CHART_NAME -f ./values-dev.yaml
```

* Uninstallation
For a complete uninstallation, make sure to delete pvc created.

```bash
helm uninstall $RELEASE_NAME
kubectl delete pvc $RELEASE_NAME-pvc
```
---
#### Values
The following table lists the configurable parameters of the oauth2-proxy chart and their default values.
Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Parameter | Description | Default
--- | --- | ---
`affinity` | node/pod affinities | None
`authenticatedEmailsFile.enabled` | Enables authorize individual email addresses | `false`
`authenticatedEmailsFile.persistence` | Defines how the email addresses file will be projected, via a configmap or secret | `configmap`
`authenticatedEmailsFile.template` | Name of the configmap or secret that is handled outside of that chart | `""`
`authenticatedEmailsFile.restrictedUserAccessKey` | The key of the configmap or secret that holds the email addresses list | `""`
`authenticatedEmailsFile.restricted_access` | [email addresses](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/oauth_provider#email-authentication) list config | `""`
`authenticatedEmailsFile.annotations` | configmap or secret annotations | `nil`
`config.clientID` | oauth client ID | `""`
`config.clientSecret` | oauth client secret | `""`
`config.cookieSecret` | server specific cookie for the secret; create a new one with `openssl rand -base64 32 \| head -c 32 \| base64` | `""`
`config.existingSecret` | existing Kubernetes secret to use for OAuth2 credentials. See [secret template](https://github.com/oauth2-proxy/manifests/blob/master/helm/oauth2-proxy/templates/secret.yaml) for the required values | `nil`
`config.configFile` | custom [oauth2_proxy.cfg](https://github.com/oauth2-proxy/oauth2-proxy/blob/master/contrib/oauth2-proxy.cfg.example) contents for settings not overridable via environment nor command line | `""`
`config.existingConfig` | existing Kubernetes configmap to use for the configuration file. See [config template](https://github.com/oauth2-proxy/manifests/blob/master/helm/oauth2-proxy/templates/configmap.yaml) for the required values | `nil`
`config.cookieName` | The name of the cookie that oauth2-proxy will create. | `""`
`alphaConfig.enabled` | Flag to toggle any alpha config related logic | `false`
`alphaConfig.annotations` | Configmap annotations | `{}`
`alphaConfig.serverConfigData` | Arbitrary configuration data to append to the server section | `{}`
`alphaConfig.metricsConfigData` | Arbitrary configuration data to append to the metrics section | `{}`
`alphaConfig.configData` | Arbitrary configuration data to append | `{}`
`alphaConfig.existingConfig` | existing Kubernetes configmap to use for the alpha configuration file. See [config template](https://github.com/oauth2-proxy/manifests/blob/master/helm/oauth2-proxy/templates/configmap-alpha.yaml) for the required values | `nil`
`customLabels` | Custom labels to add into metadata | `{}` |
`config.google.adminEmail` | user impersonated by the google service account | `""`
`config.google.serviceAccountJson` | google service account json contents | `""`
`config.google.existingConfig` | existing Kubernetes configmap to use for the service account file. See [google secret template](https://github.com/oauth2-proxy/manifests/blob/master/helm/oauth2-proxy/templates/google-secret.yaml) for the required values | `nil`
`config.google.groups` | restrict logins to members of these google groups | `[]`
`extraArgs` | key:value list of extra arguments to give the binary | `{}`
`extraEnv` | key:value list of extra environment variables to give the binary | `[]`
`extraVolumes` | list of extra volumes | `[]`
`extraVolumeMounts` | list of extra volumeMounts | `[]`
`hostAlias.enabled`  | provide extra ip:hostname alias for network name resolution.
`hostAlias.ip`  | `ip` address `hostAliases.hostname` should resolve to.
`hostAlias.hostname`  | `hostname` associated to `hostAliases.ip`.
`htpasswdFile.enabled` | enable htpasswd-file option | `false`
`htpasswdFile.entries` | list of [encrypted user:passwords](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview#command-line-options) | `{}`
`htpasswdFile.existingSecret` | existing Kubernetes secret to use for OAuth2 htpasswd file | `""`
`httpScheme` | `http` or `https`. `name` used for port on the deployment. `httpGet` port `name` and `scheme` used for `liveness`- and `readinessProbes`. `name` and `targetPort` used for the service. | `http`
`image.pullPolicy` | Image pull policy | `IfNotPresent`
`image.repository` | Image repository | `quay.io/oauth2-proxy/oauth2-proxy`
`image.tag` | Image tag | `v7.3.0`
`imagePullSecrets` | Specify image pull secrets | `nil` (does not add image pull secrets to deployed pods)
`ingress.enabled` | Enable Ingress | `false`
`ingress.className` | name referencing IngressClass | `nil`
`ingress.path` | Ingress accepted path | `/`
`ingress.pathType` | Ingress [path type](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types) | `ImplementationSpecific`
`ingress.extraPaths` | Ingress extra paths to prepend to every host configuration. Useful when configuring [custom actions with AWS ALB Ingress Controller](https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/annotation/#actions). | `[]`
`ingress.annotations` | Ingress annotations | `nil`
`ingress.hosts` | Ingress accepted hostnames | `nil`
`ingress.tls` | Ingress TLS configuration | `nil`
`livenessProbe.enabled`  | enable Kubernetes livenessProbe. Disable to use oauth2-proxy with Istio mTLS. See [Istio FAQ](https://istio.io/help/faq/security/#k8s-health-checks) | `true`
`livenessProbe.initialDelaySeconds` | number of seconds | 0
`livenessProbe.timeoutSeconds` | number of seconds | 1
`nodeSelector` | node labels for pod assignment | `{}`
`deploymentAnnotations` | annotations to add to the deployment | `{}`
`podAnnotations` | annotations to add to each pod | `{}`
`podLabels` | additional labesl to add to each pod | `{}`
`podDisruptionBudget.enabled`| Enabled creation of PodDisruptionBudget (only if replicaCount > 1) | true
`podDisruptionBudget.minAvailable`| minAvailable parameter for PodDisruptionBudget | 1
`podSecurityContext` | Kubernetes security context to apply to pod | `{}`
`priorityClassName` | priorityClassName | `nil`
`readinessProbe.enabled` | enable Kubernetes readinessProbe. Disable to use oauth2-proxy with Istio mTLS. See [Istio FAQ](https://istio.io/help/faq/security/#k8s-health-checks) | `true`
`readinessProbe.initialDelaySeconds` | number of seconds | 0
`readinessProbe.timeoutSeconds` | number of seconds | 1
`readinessProbe.periodSeconds` | number of seconds | 10
`readinessProbe.successThreshold` | number of successes | 1
`replicaCount` | desired number of pods | `1`
`resources` | pod resource requests & limits | `{}`
`service.portNumber` | port number for the service | `80`
`service.type` | type of service | `ClusterIP`
`service.clusterIP` | cluster ip address | `nil`
`service.loadBalancerIP` | ip of load balancer | `nil`
`service.loadBalancerSourceRanges` | allowed source ranges in load balancer | `nil`
`service.nodePort` | external port number for the service when service.type is `NodePort` | `nil`
`serviceAccount.enabled` | create a service account | `true`
`serviceAccount.name` | the service account name | ``
`serviceAccount.annotations` | (optional) annotations for the service account | `{}`
`tolerations` | list of node taints to tolerate | `[]`
`securityContext.enabled` | enable Kubernetes security context on container | `false`
`securityContext.runAsNonRoot` | make sure that the container runs as a non-root user | `true`
`proxyVarsAsSecrets` | choose between environment values or secrets for setting up OAUTH2_PROXY variables. When set to false, remember to add the variables OAUTH2_PROXY_CLIENT_ID, OAUTH2_PROXY_CLIENT_SECRET, OAUTH2_PROXY_COOKIE_SECRET in extraEnv | `true`
`sessionStorage.type` | Session storage type which can be one of the following: cookie or redis | `cookie`
`sessionStorage.redis.existingSecret` | Name of the Kubernetes secret containing the redis & redis sentinel password values (see also `sessionStorage.redis.passwordKey`) | `""`
`sessionStorage.redis.password` | Redis password. Applicable for all Redis configurations. Taken from redis subchart secret if not set. sessionStorage.redis.existingSecret takes precedence | `nil`
`sessionStorage.redis.passwordKey` | Key of the Kubernetes secret data containing the redis password value | `redis-password`
`sessionStorage.redis.clientType` | Allows the user to select which type of client will be used for redis instance. Possible options are: `sentinel`, `cluster` or `standalone` | `standalone`
`sessionStorage.redis.standalone.connectionUrl` | URL of redis standalone server for redis session storage (e.g. `redis://HOST[:PORT]`). Automatically generated if not set. | `""`
`sessionStorage.redis.cluster.connectionUrls` | List of Redis cluster connection URLs (e.g. `["redis://127.0.0.1:8000", "redis://127.0.0.1:8000"]`) | `[]`
`sessionStorage.redis.sentinel.existingSecret` | Name of the Kubernetes secret containing the redis sentinel password value (see also `sessionStorage.redis.sentinel.passwordKey`). Default: `sessionStorage.redis.existingSecret` | `""`
`sessionStorage.redis.sentinel.password` | Redis sentinel password. Used only for sentinel connection; any redis node passwords need to use `sessionStorage.redis.password` | `nil`
`sessionStorage.redis.sentinel.passwordKey` | Key of the Kubernetes secret data containing the redis sentinel password value | `redis-sentinel-password`
`sessionStorage.redis.sentinel.masterName` | Redis sentinel master name | `nil`
`sessionStorage.redis.sentinel.connectionUrls` | List of Redis sentinel connection URLs (e.g. `["redis://127.0.0.1:8000", "redis://127.0.0.1:8000"]`) | `[]`
`topologySpreadConstraints` | List of pod topology spread constraints | `[]`
`redis.enabled` | Enable the redis subchart deployment | `false`
`checkDeprecation` | Enable deprecation checks | `true`
`metrics.enabled` | Enable Prometheus metrics endpoint | `true`
`metrics.port` | Serve Prometheus metrics on this port | `44180`
`metrics.nodePort` | External port for the metrics when service.type is `NodePort` | `nil`
`metrics.servicemonitor.enabled` | Enable Prometheus Operator ServiceMonitor | `false`
`metrics.servicemonitor.namespace` | Define the namespace where to deploy the ServiceMonitor resource | `""`
`metrics.servicemonitor.prometheusInstance` | Prometheus Instance definition  | `default`
`metrics.servicemonitor.interval` | Prometheus scrape interval | `60s`
`metrics.servicemonitor.scrapeTimeout` | Prometheus scrape timeout | `30s`
`metrics.servicemonitor.labels` | Add custom labels to the ServiceMonitor resource| `{}`
`extraObjects` | Extra K8s manifests to deploy | `[]`


