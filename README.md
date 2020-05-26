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