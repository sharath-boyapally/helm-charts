# AppLCM Helm Chart
Deploy your own private AppLCM.

## How Do I Enable the Edgegallery Repository for Helm 2?
To add the Helm Edgegallery Charts for your local client, run `helm repo add`:

```
$ helm repo add edgegallery https://edgegallery.github.io/helm-charts
"edgegallery" has been added to your repositories
```

## Prerequisites
* Kubernetes 1.6+
* **Helm 2**
* [If enabled] nfs server and RW access to it
* [If enabled] nfs-client-provisioner for dynamic provisioning
```
helm install --name nfs-client-provisioner --set nfs.server=<nfs_sever_ip> --set nfs.path=/nfs/data --set image.tag=v2.0.1 stable/nfs-client-provisioner 
```
* [If enabled] nginx-ingress-controller for ingress
```
kubectl label node <node_name> node=edge
helm install --name nginx-ingress-controller stable/nginx-ingress --set controller.kind=DaemonSet --set controller.nodeSelector.node=edge --set controller.hostNetwork=true
```
## Configuration
By default this chart will not have persistent storage.
```

The following table lists common configurable parameters of the chart and
their default values. See values.yaml for all available options.

| Parameter                               | Description                                   | Default                  |
|-----------------------------------------|-----------------------------------------------|--------------------------|
| `image.repository`                      | Image repository                              | `edgegallery/mecm-applcm`|
| `image.tag`                             | Image tag                                     | `stable`                 |
| `image.pullPolicy`                      | Image pullPolicy                              | `Always`                 |
| `persistence.enabled`                   | Whether to use persistent storage             | `false`                  |
| `expose.port`                           | Expose port                                   | `8282`                   |
| `expose.nodePort`                       | Expose nodePort                               | `30101`                  |
| `jwt.publicKeySecretName`               | JWT public key secret                         | ``                       |
| `ssl.enabled`                           | Whether to enable ssl                         | `false`                  |
| `ssl.secretName`                        | SSL keystore secret name                      | ``                       |

Specify each parameter using the `--set key=value[,key=value]` argument to
`helm install`.
```

### JWT configuration
Edgegallery use jwt to implement authentication between center and edges, so need to config public key and private key 
for usermgmt,and config public key for applcm,**the public key of applcm should be the same as the public key of user-mgmt**.
```shell
## Generate a RSA keypair through openssl
openssl genrsa -out rsa_private_key.pem 2048
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -passout pass:te9Fmv%qaq -out encrypted_rsa_private_key.pem

## Create a jwt secret for usermgmt
kubectl create secret generic user-mgmt-jwt-secret \
  --from-file=publicKey=rsa_public_key.pem \
  --from-file=encryptedPrivateKey=encrypted_rsa_private_key.pem \
  --from-literal=encryptPassword=te9Fmv%qaq

## Create a jwt public key secret for applcm
kubectl create secret generic applcm-jwt-public-secret \
  --from-file=publicKey=rsa_public_key.pem

## Installation
helm install --name my-applcm edgegallery/applcm \
  --set jwt.publicKeySecretName=applcm-jwt-public-secret
```

#### SSL configuration
If you want to enable SSL of applcm, need to provide a keystore secret:
```shell
## Generate a keystore.p12 through keytool
keytool -genkey -alias edgegallery \
  -dname "CN=edgegallery,OU=edgegallery,O=edgegallery,L=edgegallery,ST=edgegallery,C=CN" \
  -storetype PKCS12 -keyalg RSA -keysize 2048 -storepass te9Fmv%qaq -keystore keystore.p12 -validity 365

## Create a keystore secret
kubectl create secret generic applcm-keystore-secret \
  --from-file=keystore.p12=keystore.p12 \
  --from-literal=keystorePassword=te9Fmv%qaq \
  --from-literal=keystoreType=PKCS12 \
  --from-literal=keyAlias=edgegallery

## Installation
helm install --name my-applcm edgegallery/applcm \
  --set jwt.publicKeySecretName=applcm-jwt-public-secret \
  --set ssl.enabled=true \
  --set ssl.secretName=applcm-keystore-secret
```

#### Example NodePort configuration
```shell
helm install --name my-applcm edgegallery/applcm \
  --set jwt.publicKeySecretName=applcm-jwt-public-secret \
  --set expose.nodePort=30103
```

#### Example persistence configuration
```shell
helm install --name my-applcm edgegallery/applcm \
  --set jwt.publicKeySecretName=applcm-jwt-public-secret \
  --set persistence.enabled=true
```

## Uninstall
By default, a deliberate uninstall will result in the persistent volume
claim being deleted.

```shell
helm delete my-applcm --purge
```
