# Reuse an Existing ClusterImageTemplate

Templates are meant to be reusable components. This makes it easier to build new supply chains based on existing
functionality.

For this exercise, we will build a new supply chain that has two stpes:

1. It will retrieve source code from Git using the template we created in the last step
1. It will build and publish and image based on that source code using the existing `ClusterImageTemplate`
   in the out of the box supply chain

In this exercise, we will learn how Cartographer choreographs the interactions between stamped out resources.

## The New Supply Chain

YAML for the new supply chain is as follows:

```yaml
#@ load("@ytt:data", "data")
---
apiVersion: carto.run/v1alpha1
kind: ClusterSupplyChain
metadata:
  name: source-to-image-template
spec:
  selector:
    apps.tanzu.vmware.com/workload-type: source-to-image-template

  params:
    - name: registry
      default: #@ data.values.cartographer_catalog.registry

  resources:
    - name: source-provider
      templateRef:
        kind: ClusterSourceTemplate
        name: git-repository-template
    - name: image-builder
      templateRef:
        kind: ClusterImageTemplate
        name: image
      sources:
        - resource: source-provider
          name: source
```

Notice that there are now two items under `spec.resources`: our `git-repository-template` as before, and a reference
to a `ClusterImageTemplate` named `image`. Where did that `ClusterImageTemplate` come from? The answer is that it is
provided by the out of the box supply chain. You can see it with the following command:

```shell
kubectl describe ClusterImageTemplate image
```

If you look closely, you will see that this template is configured with YTT and is significantly more complex than the
simple `ClusterSourceTemplate` we created in the last exercise. But the truth is, we don't really care about that.
We know it works so we can simply reuse it.

You will also notice that this bit of YAML is a YTT template - you will need to run this through YTT before
sending it to the cluster. The reason for this is the parameter named `registry`. The `ClusterImageTemplate` supplied with the out of the box supply chain requires this parameter - it needs to know where to publish the image! Using this YTT template, we can reuse the registry
information from our initial configuration of the app-toolkit. You might ask how I learned this. Then answer is simple - trial and error.
I could have decoded the configuration of the `ClusterImageTemplate` and found it also.

## Template Dependencies and Choreography

Take a closer look at the definition of the `ClusterImageTemplate`:

```yaml
    - name: image-builder
      templateRef:
        kind: ClusterImageTemplate
        name: image
      sources:
        - resource: source-provider
          name: source
```

There is a `sources` section that references our source provider. This sets up a dependenciy between the templates:
the `image-builder` is dependent on the `source-provider`. But this is a different kind of dependency than you might imagine.

In a traditional CI/CD system like Jenkins, you might expect that the `image-builder` resource would be created after the
`source-provider` because of this dependency. But Cartographer doesn't work this way. Instead, Cartogrpher will
will create both resources at the same time. But how can we create an image builder if we don't yet have
a source path to provide it? Won't the `image-builder` be in a bad state when it is created? Yes, it will. But this is not
a problem in Kubernetes. Kubernetes is constantly reconciling the current state of things with the desired state.

Once the `source-provider` is able to provide source code, Cartographer will update the `image-builder` and then the
image builder can reconcile to a good state.

So Cartographer doesn't need to concern itself with the order of creating things. It only needs to concern itself
with passing information from one thing to another. Cartographer calls this "choreography" and it is a
major distinction in how Cartographer works compared with traditional CI/CD systems.

What happens when in the course of daily development, the `source-provider` notices a new commit in the Git repository?
In that case, the `source-provider` will download the new source code and change it's state to expose a new URL and revision (we
say that the `GitRepository` resource is self-mutating). When this happens, Cartographer will see that the URL and revision
have changed and simply pass that new information to the `image-builder` by updating its spec. That's all Cartographer does -
it keeps track of output variable changes and updates the spec of dependent resources. Cartographer is working hand-in-hand
with the Kubernetes reconciler itself. Cartographer does not need to implement a reconciliation engine because it can make
use of Kubernetes - which already has the most powerful reconcilation engine on the planet.

So how is information passed from one resource to another?

## Parameter Passing in Cartographer

Cartographer defines standards for parameter names so the templates do not need to know the details of their dependent
templates. In this case, the `image-provider` is dependent on templates specified in the `sources` spec. Cartographer
only allows templates of type `ClusterSourecTemplate` to be referenced in a `sources` spec.

Now remember that when we built the `ClusterSourceTemplate` we had to specify where the template could find the URL and revision for
source code. We used the `spec.urlPath` and `spec.revisionPath` values to set this up. Cartographer know these values are available
and will pass them along to the `ClusterImageTemplate` in a standard way. You can look in the [Cartographer Template Reference](../06-cartographer/CartographerTemplateReference.md) page for details about the variable names.

In this supply chain, there is only one source template. In this case we can shortcut the naming and just use the name
`source` by convention. If there were more than one source template, we would need to specify the name or names
of the different templates accordingly.

## Create and Test the New Supply Chain

Create the new supply chain with this command:

```shell
ytt -f ./source-to-image-template-supply-chain.yaml --data-values-file ../03-app-toolkit/config/app-toolkit-values.yaml | kubectl apply -f-
```

Create a new workload with this command:

```powershell
tanzu apps workload create source-to-image-template `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type source-to-image-template `
  --yes `
  -n default
```

This time the supply chain will take a bit longer to run because it will use Kpack to build and publish an
image. When the supply chain completes, you should see a new image named `source-to-image-template-default` in your
repository.

[Next (Create a Cluster Template) -&gt;](03-ClusterTemplate.md)
