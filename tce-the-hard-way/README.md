# TCE the Hard Way

These instructions create a TCE based Kubernetes platform based on individual packages.
This is "the hard way" because the "App-Toolkit" package does many of these steps in a
single install. This method is useful if you want understand the underlying componants -
or perhaps make some changes to your platform configuration.

Here's what we'll install:

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

## Contour
### Install Contour

```powershell
tanzu package install contour `
  --package-name contour.community.tanzu.vmware.com `
  --version 1.20.1 `
  --values-file config/contour/contour-values.yaml
```

### Test Contour

```shell
kubectl apply -f config/contour-test.yaml
```

Nginx should be available at http://nginx-example.127-0-0-1.nip.io/

```shell
kubectl delete -f config/contour/contour-test.yaml
```

## Knative Serving
### Install Knative Serving

```powershell
tanzu package install knative-serving `
  --package-name knative-serving.community.tanzu.vmware.com `
  --version 1.0.0 `
  --values-file config/knative/knative-values.yaml
```

### Test Knative Serving

```shell
kn service create kuard --image gcr.io/kuar-demo/kuard-amd64:blue
```

Service should be available at http://kuard.default.127-0-0-1.nip.io/

```shell
kn service delete kuard
```

```shell
kubectl apply -f config/kuard-service.yaml
```

Service should be available at http://kuard.default.127-0-0-1.nip.io/

```shell
kubectl delete -f config/kuard-service.yaml
```

## Kpack
### Install Kpack

```powershell
tanzu package install kpack `
  --package-name kpack.community.tanzu.vmware.com `
  --version 0.5.3 `
  --values-file config/kpack/kpack-values.yaml
```

### Configure Kpack with KP CLI

```shell
docker login harbor.tanzuathome.net
```

```powershell
kp clusterstore save default `
  --buildpackage gcr.io/paketo-buildpacks/dotnet-core `
  --buildpackage gcr.io/paketo-buildpacks/java `
  --buildpackage gcr.io/paketo-buildpacks/nodejs
```

```powershell
kp clusterstack save base `
  --build-image paketobuildpacks/build:base-cnb `
  --run-image paketobuildpacks/run:base-cnb
```

```powershell
kp builder save my-builder `
  --tag harbor.tanzuathome.net/tce/my-builder `
  --stack base `
  --store default `
  --order config/kpack/kpack-builder-order.yaml `
  -n default
```

```powershell
kp secret create tutorial-registry-credentials `
  --registry harbor.tanzuathome.net `
  --registry-user admin `
  -n default
```

### Test Kpack with KP CLI

```powershell
kp image save tutorial-image `
  --tag harbor.tanzuathome.net/tce/kp-cli-spring-pet-clinic `
  --git https://github.com/spring-projects/spring-petclinic `
  --git-revision 82cb521d636b282340378d80a6307a08e3d4a4c4 `
  --builder my-builder `
  -n default
```

### Configure Kpack with Kubectl

```shell
kubectl apply -f config/kpack/kpack-resources.yaml
```

```powershell
kp image save tutorial-image-k `
  --tag harbor.tanzuathome.net/tce/k-spring-pet-clinic `
  --git https://github.com/spring-projects/spring-petclinic `
  --git-revision main `
  --builder my-builder-k `
  -n default
```

## FluxCD Source Controller
### Install FluxCD Source Controller

```powershell
tanzu package install fluxcd-source-controller `
  --package-name fluxcd-source-controller.community.tanzu.vmware.com `
  --version 0.21.5
```

## Certmanager
### Install Certmanager

```powershell
tanzu package install cert-manager `
   --package-name cert-manager.community.tanzu.vmware.com `
   --version 1.8.0
```

## Cartographer
### Install Cartographer

```powershell
tanzu package install cartographer `
   --package-name cartographer.community.tanzu.vmware.com `
   --version 0.3.0
```

### Configure a Supply Chain
