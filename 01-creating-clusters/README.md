# Create and Configure a Cluster

There are different ways to create a TCE cluster.

An "unmanaged" cluster is typically created using "Kind" (Kubernetes in Docker). Unmanaged clusters are perfect for
workshops such as this one - they are meant to be short-lived, and they create very quickly.

A "managed" cluster requires more planning. It is meant to be a long-lived cluster - even a production cluster.
Managed clusters are life-cycle managed by Cluster API.

For this workshop, we recommend using an unmanaged cluster if you are running on your local workstation. If you
have access to a managed workload cluster, we provide the actions you will need to take to make that cluster 
ready for the workshop as well.

- [Create an unmanaged cluster using Kind](CreateUnmanagedCluster.md)
- [Create and configure a managed cluster](CreateManagedCluster.md)
