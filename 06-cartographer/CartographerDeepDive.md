# Cartographer Deep Dive

When we ran the `tanzu apps workload create...` command, we created a `workload` - which
is an instance of a `supply chain`. We'll define those terms in more detail later. The workload, in turn,
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
that the Image resource also has child resources - we'll ignore those for now.

From this you can know that there is something called a "workload" that knows about three other things.
In Cartographer, we say that the workload "choreographs" the interactions between those three other things.

## Examining the GitRepository Resource

The first thing to examine is the `GitRepository` resource. The out of the box supply chain uses the
[Flux Source Controller](https://fluxcd.io/docs/components/source/) to supply source code for a supply chain.
The source controller will download the source code from a Git repository and make it available in the cluster.
You can see the configuration of the controller with the command

```shell
kubectl describe GitRepository java-payment-calculator
```

We'll look at the `spec` section with the following command:

```shell
kubectl get GitRepository java-payment-calculator -o=jsonpath='{.spec}' | jq
```

The output looks like this:

![Flux Configuration](images/FluxConfiguration.png)

You will see two configuration items that we supplied on the workload command: the Git URL and the branch.
There are other parts of the configuration such as the Git implementation and the timeout that came from 
somewhere else. Those items came from a `template`. This illustrates a key concept in Cartographer: resources
are built from templates, and templates can receive input values from different sources.

Now let's look at the status of the resource (all Kubernetes resources expose a status object)

```shell
kubectl get GitRepository java-payment-calculator -o=jsonpath='{.status}' | jq
```

The output looks like this:

![Flux Status](images/FluxStatus.png)

There are two important items in this output: `status.artifact.url` and `status.artifact.revision`. The URL
is a cluster internal URL where the source code is available and the revision is the Git SHA related to the source code.
Cartographer will forward these two values to the next component in the supply chain.