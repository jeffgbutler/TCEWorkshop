# Create a Cluster Source Template

The out of the box supply chain includes a fully functioning `ClusterSourceTemplate` that we can reuse. However, it is fairly complex
because it includes functionality for secrets and labels. We won't need this for a simple example, so let's recreate it as an exercise.

What we need is a template that will stamp out a `GitRepository` for the first step in our supply chain.

## Template Inputs

Templates can receive inputs from several places:

1. They can access the standard values of the workload they are associated with
1. They can access the values of parameters specified in the workload (and use default values if not supplied)
1. They can access the output values of other templates they rely on

For the `ClusterSourceTemplate` we will only interact with the first two.

## Desired Output - GitRepository

The first step is to think about what we would like the stamped out resource to look like. Here's sample YAML
for a simple `GitRepository` resource:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
  kind: GitRepository
  metadata:
    name: # MUST BE UNIQUE
    labels:
      app.kubernetes.io/component: source # HARDCODED
  spec:
    interval: 1m0s
    url: # MUST BE UNIQUE
    ref: # MUST BE UNIQUE
    gitImplementation: # SET A DEFAULT THAT CAN BE OVERRIDDEN
    ignore: '!.git' # HARDCODED
```

(for full details about this CRD see https://fluxcd.io/docs/components/source/gitrepositories/)

Some of the items in this spec are optional, but we will use them to demonstrate capabilities in Cartographer.

This is a template - which means we will need to supply some values to make each stamped out resource unique.
For a particular `GitRepository`, you can think that the name and location of the Git repository should be unique
to each instance. `GitRepository` also supports two different Git implementations: `go-git` and `libgit2`. We will
configure the template so that `go-git` is the default, but it can be overridden in the workload if we desire.

Notice also that we've hardcoded a label - this same label will be applied to every `GitRepository` stamped
out by Cartographer.

## The ClusterSourceTemplate

A `ClusterSourceTemplate` needs two main things:

1. It needs to know how to stamp out a resource that will supply source code in the cluster (i.e. it needs a resource template)
1. It needs to know where to find the resulting source code

In Cartographer, templates can be coded in two ways: as a simple Kubernetes template - very like the template definitions
you see in other Kubernetes objects like deployments, or as a YTT based template. YTT offers additional flexibility to
templates like conditionals, loops, etc. Choosing templates vs. YTT comes downs to a decision about how flexible the
template definition needs to be. The templates supplied by VMware all use YTT because they are very flexible and need
the programming functions offered with YTT. For this exercise, we can use either one.

First we will look at a simple template, then we'll look at its YTT equivalent.

### Simple ClusterSourceTemplate

A simple template to stamp out a `GitRepository` follows:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: cartographer-workshop-git-repository-template
spec:
  params:
  - name: git_implementation
    default: go-git
  urlPath: .status.artifact.url
  revisionPath: .status.artifact.revision
  template:
    apiVersion: source.toolkit.fluxcd.io/v1beta1
    kind: GitRepository
    metadata:
      name: $(workload.metadata.name)$
      labels:
        app.kubernetes.io/component: source
    spec:
      interval: 1m0s
      url: $(workload.spec.source.git.url)$
      ref: $(workload.spec.source.git.ref)$
      gitImplementation: $(params.git_implementation)$
      ignore: '!.git'
```

A few important things to notice here:

1. The template name is `cartographer-workshop-git-repository-template` - we will need this when building a supply chain
1. The `spec.params` section defines a default value for the parameter `git_implementation`
1. The `spec.template` section contains the `GitRepository` template we showed above and contains parameter
   markers for the various values that can change with every workload.

Notice in particular the parameter markers in the template. When using a simple template like this,
the parameter markers have the form `$(...)$`. There are two top level variables available: `workload` and `params`.
The `workload` variable provides access to any item in the workload that will stamp out this template. The `params`
variable provides access to and parameter values - parameter values can have defaults as shown here, and can be specified in
workload definitions to override defaults.

Now let's talk about outputs. The CRD for `ClusterSourceTemplate` has two required values that inform Cartographer
where to obtain the source code provided by the stamped out resource: `spec.urlPath` and `spec.revisionPath`. These two values
specify where *in the stamped out resource's spec* to find the source code. The values we have shown are correct
for a `GitRepository` resource.

### YTT Based ClusterSourceTemplate

The YTT equivalent of the template looks very similar:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: cartographer-workshop-git-repository-template
spec:
  params:
  - name: git_implementation
    default: go-git
  urlPath: .status.artifact.url
  revisionPath: .status.artifact.revision
  ytt: |
    #@ load("@ytt:data", "data")

    apiVersion: source.toolkit.fluxcd.io/v1beta1
    kind: GitRepository
    metadata:
      name: #@ data.values.workload.metadata.name
      labels:
        app.kubernetes.io/component: source
    spec:
      interval: 1m0s
      url: #@ data.values.workload.spec.source.git.url
      ref: #@ data.values.workload.spec.source.git.ref
      gitImplementation: #@ data.values.params.git_implementation
      ignore: '!.git'
```

Several important things to notice:

1. The basic structure of the template and parameters are the same. But instead of `spec.template` we now
   use `spec.ytt`. In `spec.ytt` we can specify any YTT script we want and use any of the YTT functionality.
   Since this is a simple template, there are no conditionals or loops here.
1. The format of the variables has changed - now we are using the YTT variable format `#@ data.values ...`

## Building a Supply Chain

Now let's build a supply chain that uses this template. It's not much of
a supply chain as it will only download source code! But it will be a good test to make sure the template is working correctly.

Here's the YAML for a supply chain that uses the `ClusterSourceTemplate`:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  name: cartographer-workshop-supply-chain
spec:
  selector:
    apps.tanzu.vmware.com/workload-type: source-to-ingress

  resources:
    - name: source-provider
      templateRef:
        kind: ClusterSourceTemplate
        name: cartographer-workshop-git-repository-template
```

This is a very simple supply chain. Some important things to notice:

1. The workload type is `source-to-ingress`. We will use this in the workload definition to specify the supply chain
   we want to run
1. The `spec.resources` section includes a reference to the simple template based `ClusterSourceTemplate` we created above
1. We haven't discussed security yet, so this supply chain may not function as is in your cluster

## Note about YTT and Kapp

We are going to use ytt and kapp to create the supply supply chain. There are a couple of reasons for this:

1. The supply chain will be composed of many different resources and kapp is a natural tool to use for managing 
   multiple resources
1. We will add to the supply chain in a few iterations. Again, kapp is a natural tool for incrementally implementing
   an application.
1. We will provide some sensible default for many values in the supply chain, but also provide a method for overriding the
   default. YTT is a natural for this.

In the [solution](./solution/) directory there is a `values.yaml` file with sensible defaults. We will discuss each value
as we get to it. Also in that directory are subdirectories with the different stages of the supply chain. We have taken the
typical approach with kapp where each resources is in it's own yaml file, and kapp will deploy things in the correct order.

We will also run each file through ytt before sending the yaml to kapp.

## Security and RBAC

Supply chains all run with a service account. By default, the "default" service account will be used from the namespace
where the workload resides. A developer can also specify a service account when creating a workload.

This can be difficult to manage because the service account will need permission to create every kind of resource
stamped out by the supply chain. When we installed TAP/TCE some of this was setup for us and we will reuse it as much
as possible.

In [solution/values.yaml](./solution/values.yaml) we provide a default name for a namespace. We're going to use "default" because it
will match what is setup in TAP/TCE. If for some reason you changes it when you installed TAP/TCE then you should change it here also.

Let's examine the cluster role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cartographer-workshop-role
rules:
  - apiGroups: [source.toolkit.fluxcd.io]
    resources: [gitrepositories]
    verbs: ['*']
```

This role gives access to create `GitRepository` resources as needed by our `ClusterSourceTemplate`. We will add to this role
as we add items to the supply chain.

Now let's examine the role binding:

```yaml
#@ load("@ytt:data", "data")
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cartographer-workshop-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cartographer-workshop-role
subjects:
- kind: ServiceAccount
  name: default
  namespace: #@ data.values.namespace
```

This is a ytt template that will bind the ClusterRole to the service account. Note that the out of the box supply chains have
already given this permission to the default service account - we're including it here for clarity and also to setup for future
permissions we will add in the following sections.

Let's create the supply chain. We're going to use kapp to create and update the supply chain because it is a simple way to
deploy many things as a single "application". In this case, the "application" is the supply chain.

```shell
ytt -f solution/step1/. --data-values-file solution/values.yaml | kapp deploy -a cartographer-workshop-supply-chain -y -f-
```

This command runs all the files in the "step1" directory through ytt, then apply the result with kapp.
This will create the role, role binding, and the supply chain. We can now use this supply chain in a workload.

## Create a Workload

We can create a workload with a command like the following. **Important:** if you are using TAP, change the namespace
below to your configured developer namespace.

```shell
tanzu apps workload create java-payment-calculator \
  --git-repo https://github.com/jeffgbutler/java-payment-calculator \
  --git-branch main \
  --type source-to-ingress \
  --yes \
  -n default
```

This creates and applies YAML like the following:

```yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  labels:
    apps.tanzu.vmware.com/workload-type: source-to-ingress
  name: java-payment-calculator
  namespace: default
spec:
  source:
    git:
      ref:
        branch: main
      url: https://github.com/jeffgbutler/java-payment-calculator
```

Notice that the `tanzu` command maps the Git parameter values as follows:

| Tanzu CLI Parameter | Resulting Spec Value |
|---|---|
| `--git-repo` | `spec.source.git.url` |
| `--git-branch` | `spec.source.git.ref.branch` |

The mapped values exactly match the values we need in the `ClusterSourceTemplate`.

This supply chain should resolve fairly quickly since it is only downloading source cdoe from GitHub. You can check the status with this command:

```shell
tanzu apps workload get java-payment-calculator
```

Now, you can inspect the stamped out object with the following command:

```shell
kubectl get GitRepository java-payment-calculator -o json | jq
```

You should see `status.artifact.revision` and `status.artifact.url` values set. We can use these as inputs to the next step
in our supply chain.

[Next (Reuse an Existing Template) -&gt;](02-ReusingTheClusterImageTemplate.md)
