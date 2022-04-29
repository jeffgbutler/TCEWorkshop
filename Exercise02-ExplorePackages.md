# Exercise 2: Explore Tanzu Packages

Your TCE unmanaged cluster comes pre-configured with access to a rich catalog of packages that can be easily
installed into the cluster. To see a list of available packages, enter the following command:

```shell
tanzu package available list
```

You should see a list of 15-20 packages. Many are standard open source building blocks of a Kubernetes platform
like Prometheus, Grafana, Cert Manager, External DNS, etc. Tanzu makes it very easy to install these componants.

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

