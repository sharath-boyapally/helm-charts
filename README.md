# Edgegallery Helm Chart

Deploy your own private Edgegallery.

## How Do I Enable the Edgegallery Repository for Helm 3?

To add the Helm Edgegallery Charts for your local client, run `helm repo add`:

```
$ helm repo add edgegallery https://edgegallery.github.io/helm-charts
"edgegallery" has been added to your repositories
```

## Prerequisites
* Kubernetes 1.6+
* Helm 3+
* [If enabled] nfs-client-provisioner for dynamic provisioning
* [If enabled] nfs server and RW access to it

## Configuration

By default this chart will not have persistent storage.
```

The following table lists common configurable parameters of the chart and
their default values. See values.yaml for all available options.

| Parameter                               | Description                                             | Default    |
|-----------------------------------------|---------------------------------------------------------|------------|
| `arch`                                  | Architecture (x86 | arm64)                              | `x86`      |
| `persistence.enabled`                   | Whether to use persistent storage                       | `false`    |
| `persistence.type`                      | Type of persistent storage                              | `nfs`      |
| `persistence.persistence.nfs.server`    | NFS server ip                                           | ``         |
| `persistence.persistence.nfs.path`      | NFS storage path                                        | ``         |
| `expose.type`                           | Type of expose (ingress | nodePort | none)              | `none`     |
| `expose.ingress.tls`                    | Enable TLS on the ingress host                          | `false`    |
| `expose.ingress.secretName`             | TLS secret to use (must be manually created)            | ``         |
| `expose.ingress.annotations.body_size`  | Upload body size of ingress                             | `20m`      |
| `expose.ingress.hosts.auth`             | Domain name of auth                                     | ``         |
| `expose.ingress.hosts.developer`        | Domain name of developer                                | ``         |
| `expose.ingress.hosts.appstore`         | Domain name of appstore                                 | ``         |
| `expose.ingress.hosts.mecm`             | Domain name of mecm                                     | ``         |
| `expose.nodePort.ip`                    | IP of host (must set it if use nodePort as expose type) | ``         |
| `expose.nodePort.appstore_fe.port`      | Internal port of appstore_fe                            | `8080`     |
| `expose.nodePort.appstore_fe.nodePort`  | NodePort of appstore_fe                                 | `30091`    |
| `expose.nodePort.developer_fe.port`     | Internal port of developer_fe                           | `8080`     |
| `expose.nodePort.developer_fe.nodePort` | NodePort of developer_fe                                | `30092`    |
| `expose.nodePort.mecm_fe.port`          | Internal port of mecm_fe                                | `8080`     |
| `expose.nodePort.mecm_fe.nodePort`      | NodePort of mecm_fe                                     | `30093`    |
| `expose.nodePort.user_mgmt.port`        | Internal port of user_mgmt                              | `8067`     |
| `expose.nodePort.user_mgmt.nodePort`    | NodePort of user_mgmt                                   | `30067`    |
| `mecm.enabled`                          | Whether to deploy mecm                                  | `true`     |
| `appstore.enabled`                      | Whether to deploy appstore                              | `true`     |
| `developer.enabled`                     | Whether to deploy mecm                                  | `true`     |
| `tool_chain.enabled`                    | Whether to deploy tool_chain                            | `true`     |
| `user_mgmt.enabled`                     | Whether to deploy user_mgmt                             | `true`     |
| `user_mgmt.sms.enabled`                 | Whether to use sms (need to buy huawei cloud service)   | `false`    |
| `service_center.enabled`                | Whether to deploy service_center                        | `true`     |

Specify each parameter using the `--set key=value[,key=value]` argument to
`helm install`.

## Installation

```shell
helm install my-edgegallery edgegallery/edgegallery -f custom.yaml
```

### Ingress

This chart provides support for ingress resources. If you have an ingress controller installed on your cluster, such as [nginx-ingress](https://hub.kubeapps.com/charts/stable/nginx-ingress) or [traefik](https://hub.kubeapps.com/charts/stable/traefik) you can utilize the ingress controller to expose Kubeapps.

To enable ingress integration, please set `expose.type` to `ingress`

#### Example Ingress configuration

```shell
helm install my-edgegallery edgegallery/edgegallery \
  --set expose.type=ingress \
  --set expose.ingress.hosts.auth=auth.edgegallery.domain.com \
  --set expose.ingress.hosts.appstore=appstore.edgegallery.domain.com \
  --set expose.ingress.hosts.developer=developer.edgegallery.domain.com \
  --set expose.ingress.hosts.mecm=mecm.edgegallery.domain.com \
  --set expose.ingress.tls.enabled=true
  --set expose.ingress.tls.secretName=edgegallery-ingress-secret
```

#### Example NodePort configuration

```shell
helm install my-edgegallery edgegallery/edgegallery \
  --set expose.type=nodePort \
  --set expose.nodePort.ip=10.11.12.13
```

#### Example NFS configuration

```shell
helm install my-edgegallery edgegallery/edgegallery \
  --set persistence.enabled=true \
  --set persistence.persistence.nfs.server=10.11.12.13 \
  --set persistence.persistence.nfs.path=/nfs/data
```

#### Example ARM64 configuration

```shell
helm install my-edgegallery edgegallery/edgegallery \
  --set arch=arm64
```

## Uninstall

By default, a deliberate uninstall will result in the persistent volume
claim being deleted.

```shell
helm delete my-edgegallery
```
