# Create a Cluster for TAP

## Create a Cluster

Created a managed cluster for TAP:

CLUSTER_NAME: tap-cluster
CLUSTER_PLAN: dev
VSPHERE_CONTROL_PLANE_ENDPOINT: 192.168.140.243
VSPHERE_WORKER_DISK_GIB: "100"
VSPHERE_WORKER_MEM_MIB: "16384"
VSPHERE_WORKER_NUM_CPUS: "4"
WORKER_MACHINE_COUNT: "5

Create the cluster:

```shell
tanzu cluster create --file tap-cluster.yaml
```

Add the cluster to local Kubectl context:

```shell
tanzu cluster kubeconfig get tap-cluster --admin
```

Export the config so you can use it on different machines:

```shell
tanzu cluster kubeconfig get tap-cluster --admin --export-file tap-cluster-kubeconfig.yaml
```

## Install MetalLB

```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

Create a config file:

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
      - 192.168.140.180-192.168.140.199
```

```shell
kubectl apply -f tap-metallb-config.yaml
```

