# Create a Custom Supply Chain

Let's suppose for some reason that an application cannot be deployed with Knative. In this exercise,
we will create a new supply chain that reuses parts of the out of the box supply chain, but changes
the deployment model from a Knative service to a standard Kubernetes deployment.

So we'll create a new supply chain that does the following:

1. Read source code from git - for this we will construct a new template as an exercise to learn the basics
   of template and supply chain construction
1. Build and publish an image based on that source code - for this we will reuse an existing template
1. Deploy the image as a Kubernetes deployment and configure Contour for ingress - for this we will
   construct a new template based on YTT

Along the way we will learn how Cartographer is configured and extended.

[Next (Creating the Cluster Source Template) -&gt;](01-ClusterSourceTemplate.md)
