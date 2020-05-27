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
* [If enabled] nfs server and RW access to it
* [If enabled] nfs-client-provisioner for dynamic provisioning
```
helm install nfs-client-provisioner --set nfs.server=<nfs_sever_ip> --set nfs.path=/nfs/data --set image.tag=v2.0.1 stable/nfs-client-provisioner 
```
* [If enabled] nginx-ingress-controller for ingress
```
kubectl label node <node_name> node=edge
helm install nginx-ingress-controller stable/nginx-ingress --set controller.kind=DaemonSet --set controller.nodeSelector.node=edge --set controller.hostNetwork=true
```
