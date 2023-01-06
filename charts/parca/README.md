# parca

![Version: 4.0.0](https://img.shields.io/badge/Version-4.0.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 0.14.0](https://img.shields.io/badge/AppVersion-0.14.0-informational?style=flat-square)

Open Source Infrastructure-wide continuous profiling

## Source Code

* <https://github.com/parca-dev/helm-charts>

### Changes

#### 4.0.0

The chart now supports parca-agent 0.10.0. Older versions of parca-agent are no longer supported.

* `agent.storeAddress` is renamed to `agent.remoteStoreAddress`.
* `agent.socketPath` has been removed as it is no longer supported by parca-agent
* `agent.podLabelSelector` has been removed as it is no longer supported by parca-agent

#### 3.0.0

In chart version 3.0.0, the following has changed:

* The PodSecurityPolicy for the agents is now disabled by default as PSPs are removed with Kubernetes 1.25. Use `agent.enablePsp: true` if you want to keep it.

#### 2.3.1
In the chart version 2.3.1, the following has changed:
the _server.service.annotations_ is now available, so the parca server service manifest can have additional annotations.

#### 2.0.0
In chart version 2.0.0, following things has changed:

* agent.socketPath is now unset by default. If you need custom path to CRI socket (ie for k3s it is /run/containerd/containerd.sock) you will need to specify it manually now.
* server.retentionTime has been removed from parca, use server.storageActiveMemory instead

### Usage

Please consult [official docs](https://www.parca.dev/docs/overview) on usage.
Chart includes prometheus-style discovery rule with annotations:

If you want k8s prometheus-like scraping for pprof endpoints, all you need is to add annotations for pod:
```
parca.dev/scrape: 'true'
```

to furter adjust scope of scraping, you can use
```
parca.dev/port: 'debug'
```

## Chart Repo

Add the following repo to use the chart:

```console
helm repo add parca https://parca-dev.github.io/helm-charts
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| agent.enablePsp | bool | `false` | If the PodSecurityPolicy should be enabled |
| agent.enabled | bool | `true` | Allows disabling parca agent |
| agent.extraArgs | list | `[]` | additional arguments to pass to the agent |
| agent.extraEnv | list | `[]` | Additional container environment variables for agent |
| agent.image.pullPolicy | string | `"IfNotPresent"` | Overrides pullpolicy |
| agent.image.repository | string | `"ghcr.io/parca-dev/parca-agent"` | Overrides the image repository |
| agent.image.tag | string | `"v0.11.0"` | Overrides the image tag |
| agent.logLevel | string | `"info"` | Agent log level |
| agent.nodeSelector | object | `{}` | node selector for scheduling agent pods |
| agent.podAnnotations | object | `{}` | Additional annotations for pods |
| agent.podSecurityContext | object | `{}` | Additional pod secutiry context |
| agent.remoteStoreAddress | string | `""` | Address of parca server to send profiles. If not defined, will be generated based on deployment settings. |
| agent.remoteStoreBearerToken | string | `""` |  |
| agent.remoteStoreInsecure | bool | `true` |  |
| agent.remoteStoreInsecureSkipVerify | bool | `true` |  |
| agent.resources | object | `{}` | resource limits and requests |
| agent.securityContext | object | `{"privileged":true,"readOnlyRootFilesystem":true}` | Security context, needs to be prilileged for ful functionality |
| agent.service.port | int | `7071` | service port for agent |
| agent.service.type | string | `"ClusterIP"` | service type for agent |
| agent.serviceMonitor.enabled | bool | `false` | enables prometheus servicemonitor for agent |
| agent.serviceMonitor.jobLabel | string | `"parca-agent"` |  |
| agent.tolerations | list | `[{"effect":"NoSchedule","operator":"Exists"},{"effect":"NoExecute","operator":"Exists"}]` | node tolerations for scheduling agent pods |
| fullnameOverride | string | `""` | Overrides helm-generated chart fullname |
| imagePullSecrets | list | `[]` | specifies pull secrets for image repository |
| ingress.annotations | object | `{}` | additional annotations for ingress |
| ingress.className | string | `""` | specifies ingress class name (ie nginx) |
| ingress.enabled | bool | `false` | enables ingress for server UI |
| ingress.hosts[0].host | string | `"chart-example.local"` |  |
| ingress.hosts[0].paths[0].path | string | `"/"` |  |
| ingress.hosts[0].paths[0].pathType | string | `"ImplementationSpecific"` |  |
| ingress.tls | list | `[]` |  |
| nameOverride | string | `""` | overrides chart name |
| server.config | object | `{"object_storage":{"bucket":{"config":{"directory":"./tmp"},"type":"FILESYSTEM"}}}` | parca server config block |
| server.corsAllowedOrigins | string | `"*"` | CORS setting |
| server.enabled | bool | `true` | Allows disabling parca server |
| server.extraArgs | list | `[]` | additional arguments to pass to the server |
| server.extraEnv | list | `[]` | additional container environment variables for server |
| server.image.pullPolicy | string | `"IfNotPresent"` | Overrides pull policy for server |
| server.image.repository | string | `"ghcr.io/parca-dev/parca"` | Overrides the image repository for server |
| server.image.tag | string | `"v0.15.0"` | Overrides the image tag for server |
| server.logLevel | string | `"info"` | logging level of parca server |
| server.nodeSelector | object | `{}` | node selector for scheduling server pod |
| server.otlpAddress | string | `""` | OpenTelemetry collector address to send traces to |
| server.podAnnotations | object | `{}` | additional annotations for server pod |
| server.podSecurityContext | object | `{}` | additional security context for server pod |
| server.resources | object | `{}` | resource limits and requests for server pod |
| server.scrapeConfigs | list | `[{"job_name":"kubernetes-pods","kubernetes_sd_configs":[{"role":"pod"}],"relabel_configs":[{"action":"keep","regex":true,"source_labels":["__meta_kubernetes_pod_annotation_parca_dev_scrape"]},{"action":"replace","regex":"(.+)","source_labels":["__meta_kubernetes_pod_annotation_parca_dev_path"],"target_label":"__metrics_path__"},{"action":"replace","regex":"([^:]+)(?::\\d+)?;(\\d+)","replacement":"$1:$2","source_labels":["__address__","__meta_kubernetes_pod_annotation_parca_dev_port"],"target_label":"__address__"},{"action":"labelmap","regex":"__meta_kubernetes_pod_label_(.+)"},{"action":"replace","source_labels":["__meta_kubernetes_namespace"],"target_label":"kubernetes_namespace"},{"action":"replace","source_labels":["__meta_kubernetes_pod_name"],"target_label":"kubernetes_pod_name"}],"scrape_interval":"1m","scrape_timeout":"10s"}]` | scrape configs for parca server |
| server.securityContext | object | `{}` | additional security context for server |
| server.service.annotations | object | `{}` | annotations to be added for the server service |
| server.service.port | int | `7070` | service port for server |
| server.service.type | string | `"ClusterIP"` | service type for server |
| server.serviceMonitor.enabled | bool | `false` | enables servicemonitor for server monitoring |
| server.serviceMonitor.jobLabel | string | `"parca-server"` |  |
| server.storageActiveMemory | int | `536870912` | Active memory dedicated for storage. 512MB my default |
| server.tolerations | object | `{}` | tolerations for scheduling server pod |
| serviceAccount.annotations | object | `{}` | Annotations to add to the service account |
| serviceAccount.create | bool | `true` | Specifies whether a service account should be created |
| serviceAccount.name | string | `""` | The name of the service account to use. If not set and create is true, a name is generated using the fullname template |

## Upgrading

A major chart version change indicates that there is an incompatible breaking change needing manual actions.
