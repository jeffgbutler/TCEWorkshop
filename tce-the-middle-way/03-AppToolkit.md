# Exercise 3: Configure and Install the App Toolkit

> Important Concepts to cover in an overview:
>
> - Overview of the App Toolkit Package

**Important** the commands below assume you have a terminal window open in the same directory as this file.

The App Toolkit is a composite package (it bundles other packages together). It is a simple way to install
a group of packages that are relevant to application developers including:

- Contour - a Kubernetes ingress controller
- Kpack - an image building tool
- Knative Serving - a Kubernetes framework that simplifies application deployments
- Cartographer - a tool for creating software supply chains
- Several others

With this exercise we will install and configure the app toolkit for use on a local workstation. First take a look at
the file `config/app-toolkit/app-toolkit-values.yaml`. The file contains configuration values for two packages in the app toolkit
(Contour, and Knative). For now, the important thing to know is that applications deployed to the app toolkit will
have generated DNS names that are sub-domains of `127-0-0-1.nip.io`. This is a convenient DNS trick - the IP address
for this domain is `127.0.0.1` - or `localhost` - so it will work on everyone's workstation.

Now we can install the app toolkit with the following command:

Windows PowerShell:
```powershell
tanzu package install app-toolkit --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.2.0 --values-file config/app-toolkit/app-toolkit-values.yaml
```

MacOS/Linux:
```shell
tanzu package install app-toolkit --package-name app-toolkit.community.tanzu.vmware.com \
  --version 0.2.0 --values-file config/app-toolkit/app-toolkit-values.yaml
```

This command will run for a few minutes. Once the package install finishes reconciling, you can see the full list
of installed packages with the following command:

```shell
tanzu package installed list -A
```

[&lt;- Previous](02-ExplorePackages.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](04-Knative.md)
