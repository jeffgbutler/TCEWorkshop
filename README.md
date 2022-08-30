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
or take them in any order you wish. If you are not familiar with the Carvel tools, we suggest working through
the sections on Carvel before attempting to build a custom supply chain with Cartographer.

If you already have the pre-requisites installed and want to jump straight into Knative, Kpack, or Cartographer,
you can create and prepare a cluster with the simple steps outlined on the [Quick Start](QuickStart.md).

1. [Install the Pre-Requisites](00-basic-setup/README.md)
1. [Create a Cluster](01-creating-clusters/README.md)
1. [Explore the Cluster](02-explore-the-cluster/README.md)
1. [Install the App Toolkit](03-app-toolkit/README.md)
1. [Deploy Applications with Knative](04-knative/README.md) (Optional)
1. [Build OCI Images with Kpack](05-kpack/README.md) (Optional)
1. [Software Supply Chains with Cartographer](06-cartographer/README.md) (Optional)
   - [Cartographer Deep Dive](06-cartographer/CartographerDeepDive.md)
1. [Create a Custom Supply Chain with Cartographer](07-CustomSupplyChain/README.md) (Optional)

Appendix: Explore the Carvel Tools (Optional):
- [Carvel Overview](90-Carvel/README.md)
- [Secretgen Controller](90-Carvel/secretgen-controller/README.md)
- [YTT](90-Carvel/ytt/README.md)
- [Kbld](90-Carvel/kbld/README.md) (TODO)
- [Kapp](90-Carvel/kapp/README.md) (TODO)
- [Kapp Controller](90-Carvel/kapp-controller/README.md) (TODO)
