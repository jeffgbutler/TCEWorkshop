# Create and Test a Managed Cluster

TCE can create managed clusters on many infrastructure providers. We will not cover the details
of creating a managed cluster as the official documentation does a good job at that. See this page for
a starting point: https://tanzucommunityedition.io/docs/v0.12/installation-planning/ 

When you create a managed cluster with TCE, the cluster will have some basic components installed that make
it a "Tanzu" cluster. These include:

- The Carvel Kapp Controller
- The Carvel secretgen controller

The cluster will not have the TCE package repository configured.

## Gain Access to You Cluster

Once the cluster is created, you will need to configure Kubectl so that you can access the cluster.
The following commands are the easiest way to do that - they must be run on the same machine you
you used to create the cluster, and they will configure Kubectl on that machine only.

The following commands assume that you have created a managed cluster named "workload-cluster".

```shell
tanzu cluster kubeconfig get workload-cluster --admin
```

```shell
kubectl config use-context workload-cluster-admin@workload-cluster
```

If you would rather access the cluster from a different machine, you can export a standalone
Kubeconfig file with the following command:

```shell
 tanzu cluster kubeconfig get workload-cluster --admin --export-file workload-cluster-kubeconfig.yaml
 ```

 After running this command you can copy the file `workload-cluster-kubeconfig.yaml` to another
 machine and use it to access the cluster.

 ## Install MetalLB (Optional)

 If you did not configure a load balancer when you created the cluster, you can install MetalLB
 to provide load balancing services.

```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

Create a configuration for MetalLB. This is what I did on my vSphere homelab:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.140.200-192.168.140.219
```

Apply the config map:

```shell
kubectl apply -f workload-metallb-config.yaml
```

Test it:

```shell
kubectl run kuard --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:blue

kubectl expose pod kuard --type=LoadBalancer --port=80 --target-port=8080
```

You can see the external IP address of the load balancer with this command:

```shell
kubectl get service kuard
```

In my case it was `192.168.140.200`, so I could access Kuard at `http://192.168.140.200`

Once you are satisfied with the test, clean up with the following:

```shell
kubectl delete service kuard

kubectl delete pod kuard
```

## Setup the TCE Package Catalog

The exercises in this workshop require that you have access to the TCE package repository.
With unmanaged clusters this access is setup automatically when the cluster is created.
With managed clusters, you must configure this as a separate step.

You can run the following command to see what packages are available in the cluster by default:

```shell
tanzu package available list -A
```

You can run the following command to see what is installed in the cluster by default:

```shell
tanzu package installed list -A
```

You can run the following command to see what package repositories are installed:

```shell
tanzu package repository list -A
```

By default, very few items are installed - only the bare minimum of infrastructure components and the
secretgen-controller (the kapp controller is also installed - it is what allow several of the tanzu subcommands
to function).

With managed clusters, you must configure access to the TCE package repository with the following command:

```shell
tanzu package repository add tce-repo \
  --url projects.registry.vmware.com/tce/main:0.12.0 \
  --namespace tanzu-package-repo-global
```

This will configure the TCE package repository in a global namespace that is open to all users of the cluster.
Run the command to list the available packages again:

```shell
tanzu package available list -A
```

You can see that the list of available packages has expanded significantly!

[Next (Explore the Cluster) -&gt;](../02-explore-the-cluster/README.md)
