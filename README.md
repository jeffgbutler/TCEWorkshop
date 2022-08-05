# Tanzu Community Edition Workshops

Workshops for Tanzu Community Edition (TCE) covering basic installation, Knative, Kpack, and Cartographer.

**Important:** these workshops are based on version 0.12.1 or greater of Tanzu Community Edition.
The instructions may not work if you use earlier versions.

## Workshop Goals

In the workshops we are going to focus on building and configuring a Kubernetes cluster that is customized
for developer experience. The cluster we build will have the following characteristics:

1. It could easily run on a developer workstation
1. It will automatically build and publish images from source code
1. It will automatically deploy images and expose a URL through a Kubernetes ingress controller using Knative

In the VMware parlance, this is an "iterate" cluster - meaning that it is meant to be used as a cluster
where developers can iterate on application development.

We will start with installing TCE, creating a cluster, and deploying a simple workload. Then we will dig in
to the details and what's going on behind the scenes.

## How to Proceed

To proceed, start with the pre-requisites page and follow the links through the workshop. You should follow the steps
in order through #4 below (Install the App Toolkit). After that, you can skip any of the following sections
or take them in any order you wish.

1. [Install the Pre-Requisites](00-basic-setup/README.md)
1. [Create a Cluster](01-creating-clusters/README.md)
1. [Explore the Cluster](02-explore-the-cluster/README.md)
   - [Basics of YTT](02-explore-the-cluster/ytt/README.md) (Optional)
   - [Basics of the Secretgen Controller](02-explore-the-cluster/secretgen-controller/README.md) (Optional)
   - [Basics of the Kapp Controller](02-explore-the-cluster/kapp-controller/README.md) (Optional) (TODO)
1. [Install the App Toolkit](03-app-toolkit/README.md)
1. [Deploy Applications with Knative](04-knative/README.md) (Optional)
1. [Build OCI Images with Kpack](05-kpack/README.md) (Optional)
1. [Software Supply Chains with Cartographer](06-cartographer/README.md) (Optional)
