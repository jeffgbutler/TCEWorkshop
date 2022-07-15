# Install The App Toolkit

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

## Update Configuration Values

The file [config/app-toolkit-values.yaml](config/app-toolkit-values.yaml) must be modified to
values suitable for your environment. The following table has details of what needs to change. There are example for
the various registry values on the [pre-requisites](../PreRequisites.md) page.

| Configuration Value | Notes |
|---|---|
| contour.envoy.service.type | Change this to "LoadBalancer" if your cluster has a load balancer available |
| knative_serving.domain.name | If you have a LoadBalancer and DNS available, specify the domain name you will use. If you are running in Kind, the default value is sufficient. |
| kpack.kp_default_repository | This is a repository where certain images for Kpack will be pushed during install |
| kpack.kp_default_repository_username | User name for the repository |
| kpack.kp_default_repository_password | Password for the repository |
| cartographer_catalog.registry.server | Server name for images created by the out of the box supply chain. This value will be used to compose tags for images |
| cartographer_catalog.registry.repository | Tag prefix for images created by the out of the box supply chain. This value will be prepended to image tags created by Cartographer |


## App Toolkit Pre-Requisites

The app toolkit only needs a registry secret. The secret must be named "registry-credentials". 

On a fully "Tanzu-ified" cluster, the Carvel secretgen-controller will be installed. When the secretgen-controller is installed,
you should create the secret using the Tanzu CLI. If the secretgen-controller is not installed, you can create the secret
using kubectl.

You can check to see if the secretgen-controller is installed with the following command:

```shell
tanzu package installed list -A
```

On a fully Tanzu-ified cluster, you will see the secretgen-controller - usually in the tkg-system namespace.

If you have the secretgen-controller installed, create the secret with a command like this:

Powershell...
```powershell
tanzu secret registry add registry-credentials `
  --server harbor.tanzuathome.net `
  --username admin `
  --password Harbor12345 `
  --export-to-all-namespaces
```

Linux/MacOS shell...
```shell
tanzu secret registry add registry-credentials \
  --server harbor.tanzuathome.net \
  --username admin \
  --password Harbor12345 \
  --export-to-all-namespaces
```

If you do not have the secretgen-controller installed, create the secret with Kubectl:

```powershell
kubectl create secret docker-registry registry-credentials `
  --docker-server=harbor.tanzuathome.net `
  --docker-username=admin `
  --docker-password=Harbor12345
```

Linux/MacOS shell...
```shell
kubectl create secret docker-registry registry-credentials \
  --docker-server=harbor.tanzuathome.net \
  --docker-username=admin \
  --docker-password=Harbor12345
```

## Install App-Toolkit

Powershell...
```powershell
tanzu package install app-toolkit `
  --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.2.0 `
  --values-file config/app-toolkit-values.yaml
```

Linux/MacOS shell...
```shell
tanzu package install app-toolkit \
  --package-name app-toolkit.community.tanzu.vmware.com \
  --version 0.2.0 \
  --values-file config/app-toolkit-values.yaml
```

## Setup DNS on an Unmanaged Cluster (Optional)

If you created an unmanaged cluster and specified a domain name other than the default (an `nip.io` domain),
then you should add a DNS wildcard record using the domain you specified with and answer of the cluster's
IP address. The IP address can be:

1. `127.0.0.1` (localhost) if you installed everything locally and only want to access applications from a local browser
1. The IP address of the VM if you used the Guard Dog OVA to create a VM

## Setup DNS With a LoadBalancer (Optional)

If you enabled load balancing and a domain in your app toolkit install, then you should add a DNS record.

First find the LoadBalancer's address:

```shell
kubectl get service envoy -n projectcontour
```

Then add a wildcard DNS "A" record using the domain name you specified in the configuration file. In my case
I added a record "*.tce.tanzuathome.net" with IP address 192.168.140.200.

## Test the App Toolkit (Optional)

Once the app toolt has finished reconciling, you can try a simple test with a Knative deployment:

```shell
kn service create kuard --image gcr.io/kuar-demo/kuard-amd64:blue
```

Once the command completes, the application should be available at http://kuard.default.127-0-0-1.nip.io/. If you
used a load balancer and changed the domain name, you can retrive the URL with this command:

```shell
kn service describe kuard
```

When you are finished experimenting with Knative, you can delete the service with the following command:

```shell
kn service delete kuard
```

[Next (Explore Knative) -&gt;](../04-knative/)
