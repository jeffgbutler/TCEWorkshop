# Tanzu Community Edition Workshops

Workshops for Tanzu Community Edition (TCE) covering basic installation, Knative, Kpack, and Cartographer.

**Important:** these workshops are based on version 0.12.1 or greater of Tanzu Community Edition.
The instructions may not work if you use earlier versions.

## Workshop Goals

In the workshops we are going to focus on building and configuring a Kubernetes cluster that is customized
for developer experience. The cluster we build will have the following characteristics:

1. It could easily run on a developer workstation
1. It will automatically build and publish imagaes from source code
1. It will automatically deploy images and expose a URL through a Kubernetes ingress controller using Knative

In the VMware parlance, this is an "iterate" cluster - meaning that it is meant to be used as a cluster
where developers can iterate on appliction development.

We will start with installing TCE, creating a cluster, and deploying a simple workload. Then we will dig in
to the details and what's going on behind the scenes.

## How to Proceed

1. [Install the Pre-Requisites](PreRequisites.md)
1. [Install Tanzu Community Edition](InstallTCE.md)
1. Create a Cluster
   - [Create an unmanaged cluster using Kind](01-creating-clusters/CreateUnmanagedCluster.md)
   - [Create and configure a managed cluster](01-creating-clusters/CreateManagedCluster.md)
1. [Explore Packages](02-explore-packages/)
1. [Install the App Toolkit](03-app-toolkit/)
1. [Deploy Applications with Knative](04-knative/)
1. [Build OCI Images with Kpack](05-kpack/)
1. [Software Supply Chains with Cartographer](06-cartographer/)
