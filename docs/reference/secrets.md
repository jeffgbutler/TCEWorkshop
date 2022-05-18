# Secrets

A TCE install usually requires secrets - most commonly registry secrets. On this page we'll detail
the many ways to create secrets with Tanzu CLIs.

## Kubectl - Imperative

This is the most basic way to create a secret. It has the drawback that the password must be scripted, or placed in an environment variable,
or entered interactively.

```shell
kubectl create secret docker-registry SECRET_NAME \
    --docker-username=REGISTRY_USR \
    --docker-password=REGISTRY_PASSWORD \
    --docker-server=REGISTRY_URL
```

If you omit the `--docker-password` parameter, you will be prompted interactively.

## Kubectl - Declarative with YTT

Sometimes it is inconvenient to create a secret interactively - if you have many Kubernetes objects
that you are creating with YAML, then you can also use YAML to create a registry secret. The difficulty
with this approach is that the registry credentials need to be encoded and in a specific format. This
can be cumbersome with plain old YAML. YTT has functions to make this easier.

First create a secret template file like this:

```yaml
#@ load("@ytt:data", "data")
#@ load("@ytt:json", "json")
---
apiVersion: v1
kind: Secret
metadata:
  name: #@ data.values.registry.secret.name
type: kubernetes.io/dockerconfigjson
stringData:
  #@ registry_creds = {"username": data.values.registry.username, "password": data.values.registry.password}
  .dockerconfigjson: #@ json.encode({"auths": {data.values.registry.server: registry_creds}})
```

The template can be reused for many different secrets. Next create a values file like this:

```yaml
registry:
  server: REGISTRY_URL
  username: REGISTRY_USER
  password: REGISTRY_PASSWORD
  secret:
    name: SECRET_NAME
```

Create the secret with the following command:

```shell
ytt -f secret-template.yaml --data-value-file values.yaml | kubectl apply -f-
```

## Tanzu CLI

The Tanzu CLI can create registry secrets if, and only if, the Carvel secretgen controller is installed on the cluster.

```shell
tanzu secret registry add SECRET_NAME \
  --server REGISTRY_URL \
  --username REGISTRY_USER \
  --password REGISTRY_PASSWORD \
  --export-to-all-namespaces
```

The `--export-to-all-namespaces` parameter is optional. If you specify it, the command will also create a SecretExport
resource.

See [Sharing Secrets with the SecretGen Controller](#sharing-secrets-with-the-secretgen-controller) below for more details.

## KP CLI

The kpack cli (kp) can also create registry secrets if you have it installed. This method can only
be used if the Kpack package is installed on your cluster. This method can only accept the password
interactively or through the `GIT_PASSWORD` environment variable.

This method can also create secrets for private Git repositories.

```shell
kp secret create SECRET_NAME \
  --registry REGISTRY_URL \
  --registry-user REGISTRY_USER \
```

## Sharing Secrets with the Secretgen Controller

