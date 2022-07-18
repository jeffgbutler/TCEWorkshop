# Tanzu Community Edition CLIs

When you install TCE you are actually installing the Tanzu CLI and its associated plugins. We'll explain about that
on this page. There are other CLIs and utilities that may be useful to install depending on the cluster features you
intend to use.

## The Tanzu CLI

The Tanzu CLI is an extensible command line tool. It is open source at https://github.com/vmware-tanzu/tanzu-framework
The Tanzu CLI supports "plugins" to extend its capabilities. One of the goals is to provide a unified interface to the
various different packages installed on a Tanzu-ified Kubernetes cluster. For this reason you will see that there
are different ways to accomplish the same task. For a given task...

1. There is always a way to accomplish the task using Kubectl and YAML - we are on Kubernetes after all!
1. Some tools - like Kpack and Knative - have their own CLIs and you can use them to accomplish certain tasks
   in an easier way than using Kubectl
1. Lastly, the Tanzu CLI will provide a unified environment for some tools, and will also add capabilities
   that are not in other tools

Good examples are the following:

1. Interacting with the Kapp controller can be accomplished with Kubectl, the Kapp CLI, or the Tanzu
   package plugin
1. Creating unmanaged clusters can only be accomplished with the Tanzu CLI

When you install Tanzu Community Edition, you are installing the Tanzu CLI and a set of CLI plugins for creating
and interacting with TCE clusters. You can see the installed plugins and their versions with the following command:

```shell
tanzu plugin list
```

**Important:** Some of VMware's commercial products also use the Tanzu CLI - notably Tanzu Kubernetes Grid Multi-Cloud
Edition (TKGm) and Tanzu Application Platform (TAP). As of this writing, the different flavors of Tanzu CLI are not
strictly compatible. For that reason, we recommend installing TKGm and TAP using virtual machines (jump boxes) in the
cloud infrastructure you are using. This will keep your workstation free for Tanzu Community Edition. This
incompatibility should be resolved in future releases.

## The Kpack CLI

The Kpack CLI (KP) is open source here: https://github.com/vmware-tanzu/kpack-cli. KP is used to interact with an installation
of Kpack on a Kubernetes cluster. With the CLI you can create clusterstores, clusterstacks, builders, images etc.
You can also use KP to see the status of builds in the cluster and to easily follow the progress of a build by tailing logs.

All of these tasks can be accomplished with Kubectl and YAML, but some are more difficult. One other difference
is that creating clusterstacks and clusterstores with the KP CLI actually relocates the supporting images to your
private container registry. This can be done with other tools, but KP makes it easy.

KP can be installed with Homebrew on MacOS or Linux:

```shell
brew tap vmware-tanzu/kpack-cli
brew install kp
```

On Windows, you will need to download the binary from GitHub and put it in your path manually.

**Important:** As of this writing, KP is not supported on MacOS with Apple Silicon (arm64 architecture).

## The Knative CLI

The Knative CLI (KN) is open source here: https://github.com/knative/client. KN is used to interact with an installation
of Knative on a Kubernetes cluster. You can use the CLI to create Knative "services" - which are essentially
Kubernetes deployments with automatic ingress configuration and scale-to-zero capability.

This can also be accomplished with Kubectl and YAML - which may be preferable. But the CLI provides a quick way
to test a Knative installation.

KN can be installed with Homebrew on MacOS or Linux:

```shell
brew install kn
```

On Windows, you will need to download the binary from GitHub and put it in your path manually.

## The Carvel Tools

The Carvel project (https://carvel.dev/) publishes many tools that are useful when working with Kubernetes.
One such example is the Kapp Controller which VMware uses as a package manager for Kubernetes - and you can
use it to package your Kubernetes applications as well. Kapp is in the same space as Helm, but has a better
ability to clearly document the changes to a cluster that will happen with an install. Kapp will also use
the Kubernetes reconciler to keep applications running and up-to-date.

The other carvel CLIs are ytt, imgpkg, and vendir. In this workshop we will only use ytt.

The YAML Templating Tool (YTT) is used to create YAML from templates using substitution values
and YAML templates. One distinction about YTT is that it parses and fully understands YAML syntax and structure
rather than just doing simple text substitution. YTT is packaged into some tools installed on
clusters - most notably Cartographer - but it is also incredibly useful on your own workstation or in
a devops toolchain.

You can install YTT with HomeBrew or Chocolatey:

```shell
brew tap vmware-tanzu/carvel

brew install ytt
```

```shell
chock install ytt
```

[Back](index.md)
