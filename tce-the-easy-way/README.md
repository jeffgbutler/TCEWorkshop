# TCE the Easy Way

These instructions create a TCE based Kubernetes platform based on the App-Toolkit packages.
This is "the easy way" because the "App-Toolkit" package does many of these steps in a
single install.

The app-toolkit will install and configure the following with a single command:

1. Contour - an ingress controller
1. Knative Serving
1. Kpack
1. Kpack configuration
1. FluxCD source controller
1. Cert Manager
1. Cartographer
1. Out of the box supply chain for cartographer (Cartographer Catalog)

**Important** the commands below assume you have a terminal window open in the same directory as this README.

## Create a Cluster

The app toolkit can install on any cluster that has been "Tanzuified" and has the TCE package
repository registered. The easist way to do this is with an unmanaged cluster.

### Create an Unmanaged Cluster Using Kind

```shell
tanzu unmanaged-cluster create tceworkshop --port-map '80:80,443:443'
```

## App Toolkit Pre-Requisites

The app toolkit only needs a secret registry secret. There are several ways to do this.

### Use the Secret-Gen Controller

The secret-gen controller can be used to create the secret in one namespace, and then make that secret available
to other namespaces.

#### Powershell

```powershell
tanzu package install secretgen-controller `
  --package-name secretgen-controller.community.tanzu.vmware.com `
  --version 0.7.1 `
  --namespace tkg-system
```

This creates a secret in the defaul namespace and makes it avaible for export to other namespaces:

```powershell
tanzu secret registry add registry-credentials `
  --server harbor.tanzuathome.net `
  --username admin `
  --password Harbor12345 `
  --export-to-all-namespaces
```

You can see the configuration of the secret exporter with ths command:

```shell
kubectl get SecretExport registry-credentials
```

If you want to make the secret available in another namespace, do the following:

```shell
kubectl create namespace testnamespace

cat <<EOF | kubectl apply -f -
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretImport
metadata:
  name: registry-credentials
  namespace: testnamespace
spec:
  fromNamespace: default
EOF
```

#### Linux/MacOS Shell

```shell
tanzu package install secretgen-controller \
  --package-name secretgen-controller.community.tanzu.vmware.com \
  --version 0.7.1 \
  --namespace tkg-system
```

```shell
tanzu secret registry add registry-credentials \
  --server harbor.tanzuathome.net \
  --username admin \
  --password Harbor12345 \
  --export-to-all-namespaces
```

### Create a Secret Using Kubectl

#### Powershell

```powershell
kubectl create secret docker-registry registry-credentials `
  --docker-server=harbor.tanzuathome.net `
  --docker-username=admin `
  --docker-password=Harbor12345
```

#### Linux/MacOS Shell

```shell
kubectl create secret docker-registry registry-credentials \
  --docker-server=harbor.tanzuathome.net \
  --docker-username=admin \
  --docker-password=Harbor12345
```

## Install App-Toolkit

### Powershell

```powershell
tanzu package install app-toolkit `
  --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.2.0 `
  --values-file config/app-toolkit/app-toolkit-values.yaml
```

### Linux/MacOS Shell

```shell
tanzu package install app-toolkit \
  --package-name app-toolkit.community.tanzu.vmware.com \
  --version 0.2.0 \
  --values-file config/app-toolkit/app-toolkit-values.yaml
```

