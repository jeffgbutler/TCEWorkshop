# Exercise 1: Install and Test Tanzu Community Edition

> Important Concepts to cover in an overview:
>
> - Unmanaged vs. Managed Clusters
> - Tanzu CLI and CLI plugins

## Install TCE on MacOS/Linux

Install Tanzu Community Edition (TCE) with Homebrew:

```shell
brew install vmware-tanzu/tanzu/tanzu-community-edition
```

There will be a script to run after the install - something like

```shell
/usr/local/Cellar/tanzu-community-edition/v0.12.0/libexec/configure-tce.sh
```

For future reference, uninstall looks like this: `/usr/local/Cellar/tanzu-community-edition/v0.12.0/libexec/uninstall.sh`

## Install TCE on Windows

Install Tanzu Community Edition (TCE) with Chocolatey:

```powershell
choco install tanzu-community-edition --version 0.12.0
```

[&lt;- Previous](00-PreReqs.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](02-CreateCluster.md)
