# Secretgen-Controller Overview

The secretgen-controller is part of the [Carvel](https://carvel.dev/) tool suite. It is currently marked as "experimental"
on the Carvel site, but it is made available in all TCE clusters. On managed clusters it is installed automatically.

The secretgen controller does a few things:

1. It can generate secrets, keys, and certificates for use by other Kubernetes resources (we will not cover that usage in this workshop)
1. It provides an operator makes it easy to share secrets across Kubernetes namespaces (we will cover this usage)

The Tanzu CLI includes a function that will create a registry secret. The Tanzu CLI secret functionality
requires the secretgen-controller to be installed! This is not strictly necessary in an unmanaged cluster currently, but we assume it
will be coming shortly.

Full details about the secretgen-controller are here: https://github.com/vmware-tanzu/carvel-secretgen-controller

**Important:** We will not use the secretgen-controller in this workshop so you can safely skip this section. Using the
secretgen-controller is useful if you want to setup different namespaces for different developers.

## Install the secretgen-controller in an Unmanaged Cluster

If you created an unmanaged cluster you will need to install the secretgen-controller. You can see if the controller is already
installed with the following command:

```shell
tanzu package installed list -A
```

If you don't see the secretgen-controller, follow these steps to install it:

1. Determine what version is available on your cluster:

   ```shell
   tanzu package available list secretgen-controller.community.tanzu.vmware.com -n tkg-system
   ```

   On my unmanaged cluster, I can see that version 0.7.1 is available.

1. Install secretgen-controller

   ```shell
   tanzu package install secretgen-controller \
     --package-name secretgen-controller.community.tanzu.vmware.com \
     --version 0.7.1 \
     --namespace tkg-system
   ```

One the package reconciles, you should see it in the installed packages list:

```shell
tanzu package installed list -A
```

## Basics of secretgen-controller

The secretgen-controller adds several CRDs to your cluster. For our purposes, we are interested in two of them:
`SecretExport` and `SecretImport`.

A `SecretExport` is used to offer secrets in one namespace to other or all namespaces. For example, suppose
we create the following secret in the default namespace:

```shell
kubectl create secret docker-registry some-secret \
  --docker-server=harbor.tanzuathome.net \
  --docker-username=admin \
  --docker-password=Harbor12345
```

We can create a `SecretExport` to make the secret available to all other namespaces:

```yaml
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretExport
metadata:
  name: some-secret
  namespace: default
spec:
  toNamespace: '*' # we can also specify a specific namespace, or a list of namespaces
```

At this point nothing has been shared. To share the secret we create a `SecretImport` in another namespace. For
example, create a namespace called `ns2`:

```shell
kubectl create ns ns2
```

Then import the secret with the following:

```yaml
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretImport
metadata:
  name: some-secret
  namespace: ns2
spec:
  fromNamespace: default
```

When the `SecretImport` in `ns2` reconciles, there will also be a copy of the secret from the `default` namespace.
If the secret changes in the `default` namespace, then the secret in `ns2` will be updated by the operator. If you delete
the `SecretImport` in `ns2`, the corresponding secret will also be deleted. If you delete the secret in the `default` namespace,
the secret in the `ns2` namespace will also be deleted (the `SecretExport` and `SecretImport` will remain).

## Secrets and the Tanzu CLI

The Tanzu CLI is also capable of creating secrets when the secretgen-controller is installed in a cluster.

The following command will create both the `Secret`, and the `SecretExport` in a single command:

```shell
tanzu secret registry add some-secret \
  --server harbor.tanzuathome.net \
  --username=admin \
  --password=Harbor12345 \
  --export-to-all-namespaces
```

You will still need to create the `SecretImport` using regular kubectl commands as shown above.

[Next (Kapp-Controller Overview) -&gt;](../kapp-controller/)
