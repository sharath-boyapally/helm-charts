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
* [If enabled] nfs-client-provisioner for dynamic provisioning
* [If enabled] nfs server and RW access to it

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
| `ssl.keystoreSecretName`                | SSL keystore secret name                      | ``                       |

Specify each parameter using the `--set key=value[,key=value]` argument to
`helm install`.

## Installation

helm install --name my-applcm edgegallery/applcm
```

### JWT configuration

AppLCM use jwt to authenticate HTTP(S) request from meo and developer, need to config public key to verify jwt, **the
public key should be the same as the public key of user-mgmt**. This chart will provide a default secret of public key, 
If you want to change it, create your secret like this:
```shell
## Generate a RSA keypair through openssl

openssl genrsa -out rsa_private_key.pem 2048
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem

## Create a jwt public key secret

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
  -storetype PKCS12 -keyalg RSA -keysize 2048 -storepass Changeme_123 -keystore keystore.p12 -validity 365

## Create a keystore secret

kubectl create secret generic applcm-keystore-secret \
  --from-file=keystore.p12=keystore.p12 \
  --from-literal=keystorePassword=Changeme_123 \
  --from-literal=keystoreType=PKCS12 \
  --from-literal=keyAlias=edgegallery

## Installation

helm install --name my-applcm edgegallery/applcm \
  --set ssl.enabled=true \
  --set ssl.keystoreSecretName=applcm-keystore-secret
```

#### Example NodePort configuration

```shell
helm install --name my-applcm edgegallery/applcm \
  --set expose.nodePort=30103
```

#### Example persistence configuration

```shell
helm install --name my-applcm edgegallery/applcm \
  --set persistence.enabled=true
```

## Uninstall

By default, a deliberate uninstall will result in the persistent volume
claim being deleted.

```shell
helm delete my-applcm --purge
```
