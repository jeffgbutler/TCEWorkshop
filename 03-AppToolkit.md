# Exercise 3: Configure and Install the App Toolkit

> Important Concepts to cover in an overview:
>
> - Overview of the App Toolkit Package

**Important!** Shell commands on this page should be executed from a terminal open in the root directory of the
git repo you cloned in the pre-requisites!

The App Toolkit is a composite package (it bundles other packages together). It is a simple way to install
a group of packages that are relevant to application developers including:

- Contour - a Kubernetes ingress controller
- Kpack - an image building tool
- Knative Serving - a Kubernetes framework that simplifies application deployments
- Cartographer - a tool for creating software supply chains
- Several others

With this exercise we will install and configure the app toolkit for use on a local workstation. First let's create a
configuration file for the app toolkit using ytt:

```shell
ytt -f config/templates/app-toolkit-values.yaml --data-values-file config/values.yaml --output-files config
```

This command will combine the registry configuration you completed in the pre-requisites with a template and
will write the resulting file into `config/app-toolkit-values.yaml`. Let's take a look at that file. The file
contains configuration values for several of the packages in the app toolkit (Contour, Knative, and Kpack).
For now, the important things to know are these:

- Applications deployed to the app toolkit will have generated DNS names that are sub-domains of
  `127-0-0-1.nip.io`. This is a convenient DNS trick - the IP address for this domain is
  `127.0.0.1` - or `localhost` - so it will work on everyone's workstation.
- Kpack is configured to talk to a the registry you configures in the pre-requisites

Now we can install the app toolkit with the following command:

Windows PowerShell:
```powershell
tanzu package install app-toolkit --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.1.0 --values-file config/app-toolkit-values.yaml
```

MacOS/Linux:
```shell
tanzu package install app-toolkit --package-name app-toolkit.community.tanzu.vmware.com \
  --version 0.1.0 --values-file config/app-toolkit-values.yaml
```

This command will run for a few minutes. Once the package install finishes reconciling, you can see the full list
of installed packages with the following command:

```shell
tanzu package installed list -A
```

[&lt;- Previous](02-ExplorePackages.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](04-Knative.md)
