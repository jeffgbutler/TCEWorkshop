# Create a Cluster Source Template

The out of the box supply chain includes a fully functioning `ClusterSourceTemplate` that we can reuse. However, it is fairly complex
because it includes functionality for secrets and labels. We won't need this for a simple example, so let's recreate it as an exercise.

What we need is a template that will stamp out a `GitRepository` for the first step in our supply chain.

## Template Inputs

Templates can receive inputs from several places:

1. The can access the standard values of the workload they are associated with
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
configure the templatre so that `go-git` is the default, but it can be overridden in the workload if we desire.

Notice also that we've hardcoded a label - this same label will be applied to every `GitRepository` stamped
out by Cartographer.

## The ClusterSourceTemplate

A `ClusterSourceTemplate` needs two main things:

1. It needs to know how to stamp out a resource that will supply source code in the cluster (i.e. it needs a resource template)
1. It needs to know where to get the resulting source code

In Cartographer, templates can be coded in two ways: as a simple Kubernetes template - very like the template definitions
you see in other Kubernetes objects like `Deployments`, or as a YTT based template. YTT offers additional flexability to
templates like conditionals, loops, etc. Choosing templates vs. YTT comes downs to a decision about how flexible the
template definition needs to be. The templates supplied by VMware all use YTT because they are very flexible and need
the programmability offered with YTT. For this exercise, we can use either one.

First we will look at a simple template, then we'll look at it's YTT equivalent.

### Simple ClusterSourceTemplate

A simple template to stamp out a `GitRepository` follows:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: git-repository-template
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

1. The template name is `git-repository-template` - we will need this when building a supply chain
1. The `spec.params` section defines a default value for the parameter `git_implementation`
1. The `spec.template` section contains the `GitRepository` template we showed above and contains parameter
   markers for the various values that can change with every workload.

Notice in particular the parameter markers in the template. When using a simplet template like this,
the parameter markers have the form `$(...)$`. There are two top level variables available: `workload` and `params`.
The `workload` variable provides access to any item in the workload that will stamp out this template. The `params`
variable provides access to and parameter values - parameter values can have defaults as shown here, and can be specified in
workload definitions to override defaults.

Now let's talk about outputs. The CRD for `ClusterSourceTemplate` has two required values that inform Cartographer
where to obtain the source code provided by the stamped out resource: `spec.urlPath` and `spec.revisionPath`. These two values
specify where *in the stamped out resource's spec* to find the source code. The values we have shown are correct
for a `GitRepository` resource.

Let's create the template in our cluster:

```shell
kubectl apply -f git-repository-template.yaml
```

You will notice that not much happened! The template was created, but nothing is using it yet. For that we'll need to build a
supply chain and a workload. But first, we will look at the YTT equivalent of this template.

### YTT Based ClusterSourceTemplate

The YTT equivalent of the template looks very similar:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: git-repository-ytt
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

Let's create this template in our cluster:

```shell
kubectl apply -f git-repository-ytt.yaml
```

Again, not much happens yet.

## Building a Supply Chain

To test this out, let's build a simple supply chain that only uses this one template. It's not much of
a supply chain as it will only download source code! But it will be a good test to make sure the template is working correctly.

Here's the YAML for a supply chain that uses the simple template version of the `ClusterSourceTemplate`:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  name: source-only-template
spec:
  selector:
    apps.tanzu.vmware.com/workload-type: source-only-template

  resources:
    - name: source-provider
      templateRef:
        kind: ClusterSourceTemplate
        name: git-repository-template
```

This is a very simple supply chain. Some important things to notice:

1. The workload type is `source-only-template`. We will use this in the workload definition to specify the supply chain
   we want to run
1. The `spec.resources` section includes a reference to to the simple template based `ClusterSourceTemplate` we created above

Let's create the supply chain:

```shell
kubectl apply -f source-only-template-supply-chain.yaml
```

Again, not much happens. But now we can create a workload to test this out.

## Create a Workload

Just like we did with the full example, we can create a workload:

```powershell
tanzu apps workload create source-only-template `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type source-only-template `
  --yes `
  -n default
```

This creates and applies YAML like the following:

```yaml
apiVersion: carto.run/v1alpha1
kind: Workload
metadata:
  labels:
    apps.tanzu.vmware.com/workload-type: source-only-template
  name: source-only
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

Now, you can inspect the stamped out object with the following command:

```shell
kubectl get GitRepository source-only-template -o json | jq
```

You should see `status.artifact.revision` and `status.artifact.url` values set. We can use these as inputs to the next step
in our supply chain.

## Further Exploration

See if you can create a supply chain that uses the YTT version of the template and a corresponding workload.
