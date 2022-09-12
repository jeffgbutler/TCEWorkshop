# Create a Custom Supply Chain

In this exercise, we will create a custom supply chain that:

1. Retrives source code from Git
1. Builds and publishes an images with Kpack
1. Uses a Kubernetes deployment, services, and ingress to deploy the application

This is similar to the out of the box supply chain except that it does not use Knative to deploy the application.
Our supply chain will be simpler than the out of the box supply chain - at the loss of some flexability.
We will also reuse one part of the existing supply chain to show how Cartographer can compose supply
chains from reusable parts.

Along the way we will learn how Cartographer is configured and extended.

In this exercise we will assume that you are familiar with the Carvel tools ytt, kapp, kapp-controller, and kbld.
If you are not yet familiar with those tools we suggest you go through the
[Carvel Tutorial](../90-Carvel/README.md) first.

This exercise will work equally well on a TCE cluster with the app-toolkit installed, or a TAP cluster
with the"full" profile installed. There are a few configuration differences we will point out as we progress.

[Next (Creating the Cluster Source Template) -&gt;](01-ClusterSourceTemplate.md)
