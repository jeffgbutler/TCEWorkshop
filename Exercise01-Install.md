# Exercise 1: Install and Test Tanzu Community Edition

> Important Concepts to cover in an overview:
>
> - Unmanaged vs. Managed Clusters
> - Tanzu CLI and CLI plugins
> - Tanzu Package Management with Kapp

Install Tanzu Community Edition (TCE) with Chocolatey:

```powershell
choco install tanzu-community-edition --version 0.11.0
```

Note that it is important to specify the version! TCE is currently a moderated package at Chocolatey and this version is not
set as the default.

Installing TCE does not create Kubernetes clusters. Rather, it installs the Tanzu CLI and several plugins that enable
you to create clusters and manage packages in clusters.

Create a cluster:

```shell
tanzu unmanaged-cluster create tceworkshop --port-map '80:80,443:443'
```

This will create a single node unmanaged Kubernetes cluster using **Kind** (Kubernetes in Docker) on your local workstation.
Unmanaged clusters are suitable for short lived experimentation and learning (such as this workshop). They also start very quickly.
This cluster will have the following characteristics:

- Ports 80 and 443 are exposed on your workstation to allow easier access to workloads deployed on the cluster
- The kapp controller will be installed to support Tanzu package management
- The cluster will have the package catalog for Tanzu configured

Unmanaged clusters are not suitable for production workloads. For production workloads, TCE can create "managed clusters"
in a variety of cloud based environments, on vSphere, and even on Docker on your workstation if you so desire.

Once the cluster is up, `kubectl` should be configured to connect to it. You can test this by running the following:

```shell
kubectl get nodes
```

You should see a single node named something like `tceworkshop-control-plane`.

You can deploy a test image with the following command:

```shell
kubectl run kuard --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:blue
```

Once the pod is running, enter the following command to access it:

```shell
kubectl port-forward kuard 8080:8080
```

Now you should be able to access the pod in a browser at http://localhost:8080

Assuming all this works, then enter `ctrl-c` to stop the port forward, then enter the following command to delete
the pod:

```shell
kubectl delete pod kuard
```
