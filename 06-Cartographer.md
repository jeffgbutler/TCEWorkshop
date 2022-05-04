# Exercise 6: Configure and Test a Supply Chain in Cartographer 

> Important Concepts to cover in an overview:
>
> - Overview of Cartographer (templates, supply chains, workloads)

**Important!** Shell commands on this page should be executed from a terminal open in the root directory of the
git repo you cloned in the pre-requisites!

So far we've seen that we can simply build images with Kpack, and we can simply deploy applications with Knative.
But there's a lot of manual work involved:

- We have to define a Kpack image for the source code
- Once the image is built, we need to retrive the new SHA
- We need to deploy or update the Knative service with the new image

We have not talked about how the images and services should be updated if a developer were to commit a code
change. This seems like a job for automation.

In this exercise, we'll take a look at Cartographer - an open source project build for exactly this purpose.
Cartographer is a supply chain choreographer. It is built to allow platform operators to define pre-built
paths to production that developers could easily use.

Cartographer is open source and you can read all about it here: https://cartographer.sh/

## Cartographer Overview

Cartographer can be a bit confusing at first because it doesn't actually do anything on it's own! Rather than
implementing everything needed for supply chains, Cartographer choreographs the activities that are performed
by other objects in a cluster. Cartographer can coordinate virtually anything that can be expressed
as a Kubernetes CRD.

Another interesting concept is that a "supply chain" is meant to be reusable. It describes a particular path to
production and required steps along the way. A "workload" is a particular instance of a supply chain.

For example, in this exercise we will define a supply chain that knows how to build an image from source code
(using Kpack) and then deploy that image as a Knative application. When we create a workload instance, we will specify
the things that make the workload unique - the name of the workload and the Git repository where the source code lives.

Supply chains are composed of "templates". Templates are CRDs that know how to create other Kubernetes resources.
They also understand required inputs and outputs so they can be chained together. For example, the "ClusterImageTemplate"
knows how to create an object that can supply container images. For example, the Kpack Image CRD knows how to take source
code and create an image. A ClusterImageTemplate can be configured to create a Kpack Image.

Supply chains and templates are just definitions of how to create things and how to knit things together. Nothing
else gets created until a workload is created.

It gets very meta!

## RBAC

Before we create the supply chain, we will need to give some permissions to our service account so that
it will be able to create the objects needed for the supply chain. Take a look at
[config/cartographer/app-operator/rbac.yaml](config/cartographer/app-operator/rbac.yaml). The permissions in that 
file are sufficient for the supply chain in this exercise, but you may need to create other permissions
if you create a different supply chain.

## Templates

We'll create a supply chain with three templates:

1. A ClusterSourceTemplate that knows how to create an instance of a FLuxCD GitRepository CRD that will watch a Git repository
2. A ClusterImageTemplate that knows how to create an instance of a Kpack Image CRD that will build a container image from source
3. A ClusterTemplate that knows how to create an instance of a Kapp that will deploy to Knative

First let's look at the `ClusterSourceTemplate`:
[config/cartographer/app-operator/git-source-template.yaml](config/cartographer/app-operator/git-source-template.yaml).
This is a simple template that will accept values from a workload

## Supply Chain

## Install the Supply Chain

Let's install the supply chain. The following command will install the templates and supply chain as well as
configuring RBAC for Cartographer. Essentially, the command will consolodate all YAML files in the
`config/cartographer/app-operator` directoy into a single file, apply the YTT overrides, then pipe it to
Kubectl.

```shell
ytt -f config/cartographer/app-operator --data-values-file config/values.yaml --ignore-unknown-comments | kubectl apply -f-
```

## Workload

```shell
ytt -f  config/cartographer/developer/workload.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

## Debugging

Things don't start happening until a workload is deployed. The first thing to check is the workload itself:

```shell
kubectl describe workload.carto.run java-payment-calculator
```

This check can be useful for finding permissions issues. Note that a status reason of `MissingValueAtPath` may be perfectly
normal. This indicates that a step in the supply chain is waiting for a prior CRD to come ready and publish a result value.

The supply chain we are using creates instances of Kpack Images, Knative services, FluxCD GitRepositories, and Kapp Apps.
You can use normal debugging techniques for each of those items to check on individual issues.

You can look at the FluxCD GitRepository object with this command:

```shell
kubectl describe GitRepository java-payment-calculator
```

You can follow the image build with normal Kpack commands:

```shell
kp build logs java-payment-calculator
```

You can get information about the Knative service as normal:

```shell
kn service describe java-payment-calculator
```

Kapp is something we haven't looked at previously in much detail. It is part of the Carvel tooling (https://carvel.dev/)
installed when the cluster is first created. If you are familiar with Kapp and have the CLI installed you can use it to show
information about the application.

You can also look at the Kapp application object with a command like this:

```shell
kubectl describe app java-payment-calculator
```

You can use the the Kubectl Tree plugin (https://github.com/ahmetb/kubectl-tree) to get a good picture of how everything is related:

```shell
kubectl tree workload java-payment-calculator
```

[&lt;- Previous](05-Kpack.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](99-Cleanup.md)
