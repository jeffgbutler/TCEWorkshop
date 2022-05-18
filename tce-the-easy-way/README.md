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
1. Supply chain for cartographer

**Important** the commands below assume you have a terminal window open in the same directory as this README.

## Create a Cluster

```shell
tanzu unmanaged-cluster create tceworkshop --port-map '80:80,443:443'
```

## App-Toolkit
### Install App-Toolkit

Don't need secretgen controller! Doesn't work on arm! But the Tanzu CLI secret plugin requires it,
so we will create secrets with normal Kubectl commands

```powershell
tanzu package install secretgen-controller `
  --package-name secretgen-controller.community.tanzu.vmware.com `
  --version 0.7.1 `
  --namespace tkg-system
```

```powershell
tanzu secret registry add registry-credentials `
  --server harbor.tanzuathome.net `
  --username admin `
  --password Harbor12345 `
  --export-to-all-namespaces
```


```shell
kubectl delete secret registry-credentials
```

```powershell
kubectl create secret docker-registry registry-credentials `
  --docker-server=harbor.tanzuathome.net `
  --docker-username=admin `
  --docker-password=Harbor12345
```

```powershell
tanzu package install app-toolkit `
  --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.2.0 `
  --values-file config/app-toolkit/app-toolkit-values.yaml
```

```powershell
tanzu apps workload create java-payment-calculator `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type web `
  --label app.kubernetes.io/part-of=hello-world `
  --yes `
  -n default
```

```shell
tanzu apps workload tail java-payment-calculator
```

```shell
tanzu apps workload get java-payment-calculator
```
