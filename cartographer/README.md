# Exploring Cartographer

The information and commands in this section assume you have created and configured a cluster
as described in [README.md](../tce-the-easy-way/README.md), and have deployed a workload as described in
[workload.md](../tce-the-easy-way/workload.md).

TCE ships with an "out of the box" supply chain does three things:

1. Watch a git repo for source code changes
1. Build an image based on the current source code and push the image into a repository
1. Deploy the image as a Knative application using Kapp

We'll take a closer look at how this all fits together.

## Examining the Workload

When we ran the `tanzu apps workload create...` command, we created a workload - which
is an instance of a supply chain. We'll define those terms in more detail later. The workload
created three custom resources in the cluster - one for each of the steps outlined above.
In addition, there is a workload resource that manages the interactions between the three other
resources. You can see these object using the "tree" plugin for kubectl:

```shell
kubectl tree workload java-payment-calculator
```

The output looks like this:

![Workload Tree](images/WorkloadTree.png)

Here you see a Workload resource named "java-payment-calculator" that is the parent of three other resources:
an App, a GitRepository, and an Image. These correspond to the three parts of the supply chain. Notice
that the Image resource also has child resources - we'll leave that alone for now.

From this you can know that there is something called a "workload" that knows about three other things.
In Cartographer, we say that the workload "choreographs" the interactions between those three other things.

## Examining the GitRepository Resource
