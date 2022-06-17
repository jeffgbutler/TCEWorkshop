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

We will start with install TCE, create a cluster, and deploy a simple workload. Then we will dig in
to the details and what's going on behind the scenes.

## How to Proceed

1. [Install the Pre-Requisites](PreRequisites.md)
1. [Install Tanzu Community Edition](InstallTCE.md)
1. Create a Cluster
   - [Create an unmanaged cluster using Kind](creating-clusters/CreateUnmanagedCluster.md)
1. [Install the App Toolkit](app-toolkit/README.md)

## Workshop Agenda

In this workshop we are going to focus on building a Kubernetes cluster that is customized for developer
experience. The cluster we build will have the following characteristics:

1. It will easily run on a developer workstation
1. It will automatically build and publish imagaes from source code
1. It will automatically deploy images and expose a URL through a Kubernetes ingress controller using Knative

In the VMware parlance, this is an "iterate" cluster - meaning that it is meant to be used as a cluster
where developers can iterate on appliction development.

There are three ways to do the workshop:

1. The "hard way" - where we install each component individually
1. The "easy way" - where we install and configure everything in minimal steps
1. The "middle way" - where install all the components, but we configure some components
   manually.

When we do this as a guided workshop, we'll follow the "hard way" path so we can discuss each step
along the way. When I'm setting up a cluster for my own use, I use the "easy way" just to get going
faster.

## The Middle Way

- [Pre-Requisites](tce-the-middle-way/00-PreReqs.md)
- [Exercise 1 - Install TCE](tce-the-middle-way/01-InstallTCE.md)
- [Exercise 2 - Create and Test a Cluster](tce-the-middle-way/02-CreateCluster.md)
- [Exercise 3 - Explore Packages](tce-the-middle-way/03-ExplorePackages.md)
- [Exercise 4 - Install, configure, and test App Toolkit](tce-the-middle-way/04-AppToolkit.md)
- [Exercise 5 - Deploy applications with Knative](tce-the-middle-way/05-Knative.md)
- [Exercise 6 - Configure and test Kpack](tce-the-middle-way/06-Kpack.md)
- [Exercise 7 - Configure and test Cartographer](tce-the-middle-way/07-Cartographer.md)
- [Cleanup](tce-the-middle-way/99-Cleanup.md)
