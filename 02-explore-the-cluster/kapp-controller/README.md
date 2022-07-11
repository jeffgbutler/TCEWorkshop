# Kapp-Controller Overview

The Kapp controller is a Kubernetes operator that understands how to install and modify software packages.
All of the software available for TCE from VMware is packaged for use with the Kapp controller.
We have already seen the package repositories and installed packages available in the TCE cluster.
In this section, we will take a quick look at how such packages were created.

There is also a CLI for Kapp that simplifies some of the operations. In addition, the Tanzu CLI
is using Kapp under the covers for several operations (package list, package install, etc.) So, as
in most things with Kubernetes, there are many different ways to accomplish the same task.
We will touch briefly on the Kapp CLI below as well.

Full details about the Kapp controller are here: https://carvel.dev/kapp-controller/

Full details about the Kapp API are here: https://carvel.dev/kapp/

## Creating a Kubernetes Application
