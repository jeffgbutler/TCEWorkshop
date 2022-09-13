# Secretgen-Controller Overview

The secretgen-controller is part of the [Carvel](https://carvel.dev/) tool suite. It is available as a package in all
Tanzu clusters. On TCE unmanaged clusters it is available but not installed - on all other Tanzu clusters it is
installed.

The secretgen-controller does a few things:

1. It can generate secrets, keys, and certificates for use by other Kubernetes resources (we will not cover that usage in this workshop)
2. It provides an operator makes it easy to share secrets across Kubernetes namespaces (we will cover this usage)
3. As of version 0.9.0, it includes a CRD `SecretTemplate` that can be used to create secrets with arbitrary fields, and
   secrets composed of other secrets. There are interesting use cases associated with this facility related to service
   binding. We will not cover that use in this workshop, but you can read a good blog post about it here:
   https://tanzu.vmware.com/developer/guides/tanzu-service-secret-sauce/

The Tanzu CLI includes a function that will create a registry secret. The Tanzu CLI secret functionality
requires the secretgen-controller to be installed! This is not strictly necessary in a TCE unmanaged cluster currently,
but we assume it will be coming shortly.

Full details about the secretgen-controller are here: https://github.com/vmware-tanzu/carvel-secretgen-controller

**Important:** We will not use the secretgen-controller in this workshop, so you can safely skip this section. Using the
secretgen-controller is useful if you want to set up different namespaces for different developers.

## Install the secretgen-controller in a TCE Unmanaged Cluster

If you created a TCE unmanaged cluster you will need to install the secretgen-controller. You can see if the controller
is already installed with the following command:

```shell
tanzu package installed list -A
```

If you don't see the secretgen-controller, follow these steps to install it:

1. Determine what version is available on your cluster:

   ```shell
   tanzu package available list secretgen-controller.community.tanzu.vmware.com -n tkg-system
   ```

   On my unmanaged cluster, I can see that version 0.7.1 is available.

2. Install secretgen-controller

   <details><summary>Powershell</summary>
   <p>

   ```powershell
   tanzu package install secretgen-controller `
     --package-name secretgen-controller.community.tanzu.vmware.com `
     --version 0.7.1 `
     --namespace tkg-system
   ```
   
   </p>
   </details>

   <details><summary>MacOS/Linux</summary>
   <p>

   ```shell
   tanzu package install secretgen-controller \
     --package-name secretgen-controller.community.tanzu.vmware.com \
     --version 0.7.1 \
     --namespace tkg-system
   ```

   </p>
   </details>

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
tanzu secret registry add another-secret \
  --server harbor.tanzuathome.net \
  --username=admin \
  --password=Harbor12345 \
  --export-to-all-namespaces
```

You can see the `SecretExport` with this command:

```shell
kubectl get SecretExports.secretgen.carvel.dev
```

You will still need to create the `SecretImport` in the other namespaces using regular kubectl commands as shown above.

If you delete the secret with the Tanzu CLI, the secret export will also be deleted.

[Next (YTT Overview) -&gt;](../ytt/README.md)
