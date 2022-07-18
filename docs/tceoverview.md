# Tanzu Community Edition Overview

Tanzu Community Edition (TCE) is a free and open source Kubernetes based platform from VMware. It includes
many of the components that undergird VMware's commercial offerings.

So, what do we mean by "Kubernetes Based Platform"?

## Kubernetes vs. A Kubernetes Based Platform

One common misconception about Kubernetes is that it is a complete platform for modern cloud based applications -
it is not. Rather, Kubernetes provides a tremendous starting point for building platforms and clusters that make
sense in different contexts. For example, you might see the following types of Kubernetes clusters:

1. A cluster built for developers that provides automation and runtime support for rapid and
   iterative application development
1. A cluster focused built for production runtimes that provides a very secure, reliable, and observable
   environment for production workloads
1. A cluster built for management of other clusters with multi-cluster control planes deployed and
   visible to platform operators

It is very common for platform operators to build different types of clusters to meet these different requirements.
TCE fits into this space by providing:

1. An open source Kubernetes distribution that can be deployed on many different platforms - and local workstations
1. Many open source components - packaged for easy installation - that can be used to create bespoke clusters for
   many different use cases

## TCE Cluster Types

Tanzu Community Edition supports two types of Kubernetes clusters:

1. Managed Clusters
1. Unmanaged Clusters

We'll talk a bit about what this means...

### Managed Clusters

In Tanzu Community Edition, a "managed cluster" is a cluster that is intended to be long-lived. Managed clusters are
appropriate for production workloads. TCE manages clusters using [Cluster API](https://cluster-api.sigs.k8s.io/). Cluster API is a Kubernetes
sub-project providing tooling for provisioning, upgrading, and operating Kubernetes clusters.

The way this works is that we first create a "management cluster" whose sole purpose is to run Cluster API and maintain the configuration
information for other "workload clusters". TCE can create a management cluster either using an interactive browser based UI, or through
APIs.

Currently, TCE can create management and workload clusters on the following infrastructure:

1. Amazon EC2
1. Microsoft Azure
1. VMware vSphere
1. Docker - although this would be an unusual usage

We won't say more about managed clusters as they are not the focus of this workshop. But the tools we install in the workshop should
work just fine in managed clusters.

### Unmanaged Clusters

In Tanzu Community Edition, an "unmanaged cluster" is a short-lived Kubernetes cluster that is not managed by Cluster API. Unmanaged clusters
start very quickly and consume fewer resources than managed clusters. They are appropriate for developer use and experimentation.

Currently, TCE can create unmanaged clusters using either [Kind](https://kind.sigs.k8s.io/) or [Minikube](https://minikube.sigs.k8s.io/docs/).

Under the covers, an unmanaged cluster is a Kind or Minikube cluster that has been "Tanzu-ified".

## Tanzu-ified Clusters

With either managed or unmanaged clusters, TCE creates a Kubernetes cluster and adds components to it.

The primary component added to Tanzu clusters is the [kapp controller](https://carvel.dev/kapp-controller/) from the Carvel tools. The kapp controller
is an open source package manager for Kubernetes. It adds CRDs to clusters that support easy installation and management of applications.

With unmanaged clusters, TCE also configures access to a package repository so you can start installing packages right away.

## What is Actually Installed with TCE?

When you install TCE, you are actually installing a command line tool - or CLI - that is used to manage other parts of TCE. We'll explain
more about this in the [next section](tce-clis.md).

[Back](index.md)
