# Going Further with Cartographer

We have covered many of Cartographer's capabilities, but not all of them. Here's a short overview of other Cartographer
capabilities.

## ClusterConfigTemplate

This is a template that is similar to `ClusterSourceTemplate` and `ClusterImageTemplate` except that its output
is configuration. The template should stamp out a resource that builds or exposes configuration data. A supply
chain is able to forward configuration data to other templates in the supply chain.

## Deliverable and ClusterDelivery

`ClusterDelivery` is a different type of blueprint. It is similar to `SupplyChain` in that it contains references to
other templates. `ClusterDelivery` supports `ClusterSourceTemplate`, `ClusterTemplate`, and `ClusterDeploymentTemplate`.

`ClusterDelivery` is designed to support continuous delivery and multi-cluster operations. For example, a `SupplyChain`
could be configured to build, test, and publish an artifact. Then a `ClusterDelivery` could be configured to deploy
that artifact in different clusters (like dev, test, prod).

A `Deliverable` is an instance of a `ClusterDelivery` - similar to `Workload` being an instance of a `SupplyChain`.

## ClusterDeploymentTemplate

A `ClusterDeploymentTemplate` can be added to a `ClusterDelivery` to configure a cluster based on input from
some external resource. For example, in the TAP out-of-the-box supply chains, there is a `ClusterDeliveryTemplate`
that deploys an application using Knative based on inputs stored in a Git repository.

## Runnable

One of the key concepts in Cartographer is that Cartographer will update the definition of a stamped out resource,
and then trust that resource to react to configuration changes and reconcile. This is a key concept of the Kubernetes
CRD model and many Kubernetes resources function this way - but not all.

