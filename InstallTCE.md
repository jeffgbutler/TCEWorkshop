# Install and Test Tanzu Community Edition

When you install Tanzu Community Edition (TCE), you are actually installing a command line interface - the Tanzu CLI - for
running commands in the TCE platform. Further activities like creating clusters and installing packages are all accomplished
with the Tanzu CLI.

**Warning:** This install will conflict with the Tanzu CLI for Tanzu Application Platform (TAP) from VMware. We recommend
that you install Tanzu Community Edition on a different machine than the machine you use to manage TAP.

## Install TCE on MacOS/Linux

Install Tanzu Community Edition with Homebrew:

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
choco install tanzu-community-edition
```

## Test TCE

Run this command:

```shell
tanzu plugin list
```

You should see a list of plugins including apps, builder, unmanaged-cluster, etc. At the time of this writing there are 13 plugins available.
