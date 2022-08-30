# Explore Your Tanzu Cluster

When you created a cluster - either managed or unmanaged - you created a "Tanzu" cluster. We will talk briefly about what that
means, and then we'll explore the cluster.

## What is a Tanzu cluster?

There is some debate internally within VMware as to the precise definition of a "Tanzu" cluster. Of course, it is 
just Kubernetes. But it is also Kubernetes with a few additions. In general, a Tanzu cluster
will have the following components installed on top of the standard Kubernetes stuff:

1. The "kapp-controller" - a Kubernetes native package manager that is part of the [Carvel](https://carvel.dev/) suite of tools
1. The "secretgen-controller" - also from the Carvel tools. The secretgen controller makes it easier to share secrets across namespaces in a Kubernetes cluster
1. An unmanaged cluster will also be connected to the main TCE package repository

In an unmanaged cluster, the secretgen-controller is available but not installed by default.

In a managed cluster, the kapp-controller is installed but not automatically connected to the main package repository.

## Exploring Packages
Your TCE cluster is configured with access to a rich catalog of packages that can be easily
installed into the cluster through the use of the kapp-controller. Packages are made available through
package repositories that are configured in the kapp-controller. A TCE unmanaged cluster is automatically configured
with access to a rich repository of packages. A TCE managed cluster requires a separate step to configure access to
the TCE package repository (this is documented on the [Create Unmanaged Cluster Page](../01-creating-clusters/CreateUnmanagedCluster.md)).

To see a list of package repositories, enter the following command:

```shell
tanzu package repository list -A
```

You should see two repositories listed. One is `tkg-core-repository` that contains fundamental Kubernetes support packages like CNIs,
metrics server, Pinniped for authorization, etc. Another is the TCE main package repository that contains many open source packages
like Kpack, Knative, Cartographer, Prometheus, etc.

To see a list of available packages, enter the following command:

```shell
tanzu package available list
```

You should see a list of 15-20 packages from the TCE main repository. Many are standard open source building blocks of a
Kubernetes platform like Prometheus, Grafana, Cert Manager, External DNS, etc. Tanzu makes it very easy to install these
components. If you want to see all the packages from every repository, enter the following command:

```shell
tanzu package available list -A
```

This will list the same packages as above, and will also include packages from the `tkg-core-repository`.

Most packages have several versions available. You can see the available versions of a package with a command like the
following (this will show all available versions of cert manager):

```shell
tanzu package available list cert-manager.community.tanzu.vmware.com
```

Most packages can be configured with values specific to your needs. If you want to see the configuration options for
a package, enter a command like the following:

```shell
tanzu package available get cert-manager.community.tanzu.vmware.com/1.6.1 --values-schema
```

This command will show that cert manager version 1.6.1 accepts a single parameter `namespace` and the default
value is `cert-manager`. Package installs can accept a YAML "values file" that contains configuration parameters.
We will see an example of this in the next section.

## Next Steps

If you are already familiar with the Carvel tools YTT, Kapp, Kbld, Kapp Controller, and Secretgen-controller, you can proceed
to install the app toolkit. If you are not familiar with those tools, we suggest you proceed to the pages
about each one.

[Explore the Carvel Tools](../90-Carvel/README.md)

[Next (Install App Toolkit) -&gt;](../03-app-toolkit/README.md)
