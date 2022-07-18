# Cartographer Template Reference

Several Cartographer templates emit values for use in other parts of the supply chain. Those values
are made available as either simple template variables, or YTT variables depending on how you configure
the receiving templates. In addition, there are "short cut" variables available if there is only one instance
of a particular template type in a supply chain. The following tables show the possibilities for
the different templates that emit values.

In each section we describe:

1. The template configuration setting - which tells the template where to find a value in the stamped out resource's configuration
1. The simple template variables available in the supply chain
1. The YTT variables available in the supply chain

For example, suppose you configure a `ClusterSourceTemplate` to stamp out a Flux `GitRepository` resource. The `GitRepository`
exposes status values denoting the source URL and revision of the resource (`status.artifact.url` and `status.artifact.revision`
respectively). So we would configure the `ClusterSourceTemplate` to look in those values of the stamped out resource to find
the values:

```yaml
apiVersion: carto.run/v1alpha1
kind: ClusterSourceTemplate
metadata:
  name: git-repository
spec:
  urlPath: .status.artifact.url
  revisionPath: .status.artifact.revision

  ...
```

## ClusterSourceTemplate

A `ClusterSourceTemplate` is responsible for supplying `url` and `revision` variables to a supply chain. Typically the template
will stamp out a resource that will watch a Git repository, download the source code and make it available in the cluster.
The stamped out resource will usually update it's local copy of the source code when there is a commit.

The `source-to-url` supply chain supplied with TCE stamps out a
[Flux GitRepository](https://fluxcd.io/docs/components/source/gitrepositories/) resource to perform this function for each workload.

| Configuration | Template Variable                                                                                | YTT Variable                                                                                                           |
|---------------|--------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|
| spec.urlPath  | `$(source.url)$` (Single Template) <br/> `$(sources.<name>.url)$` (Multiple Templates)           | `#@ data.values.source.url` (Single Template) <br/> `#@ data.values.sources.<name>.url` (Multiple Templates)           |
| spec.revision | `$(source.revision)$` (Single Template) <br/> `$(sources.<name>.revision)$` (Multiple Templates) | `#@ data.values.source.revision` (Single Template) <br/> `#@ data.values.sources.<name>.revision` (Multiple Templates) |

## ClusterImageTemplate

A `ClusterImageTemplate` is responsible for supplying an `image` variable to a supply chain. Typically the template will stamp
out a resource that will receive `url` and `revision` variables from a `ClusterSourceTemplate`, build an image based on the
source code, and publish the image to a repository. The stamped out resource will usually build and publish a new version of the
image when the input `url` or `revision` changes.

The `source-to-url` supply chain supplied with TCE stamps out a
[Kpack Image](https://github.com/pivotal/kpack/blob/main/docs/image.md) resource to perform this function for each workload.

| Configuration  | Template Variable                                                                  | YTT Variable                                                                                             |
|----------------|------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| spec.imagePath | `$(image)$` (Single Template) <br/> `$(images.<name>.image)$` (Multiple Templates) | `#@ data.values.image` (Single Template) <br/> `#@ data.values.images.<name>.image` (Multiple Templates) |

## ClusterConfigTemplate

A `ClusterConfigTemplate` is responsible for supplying a `config` variable to a supply chain. Typically the template will
stamp out a resource that will read Kubernetes configurations and make their values available. The stamped out resource
will make the new configuration values available when the underlying configuration changes.

The `source-to-url` supply chain supplied with TCE does not use a `ClusterConfigurationTemplate`. You could create a
`ClusterConfigurationTemplate` that stamped out a common Kubernetes ConfigMap, set some values in the ConfigMap, and them
made those values available to the supply chain.

| Configuration   | Template Variable                                                                     | YTT Variable                                                                                                |
|-----------------|---------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| spec.configPath | `$(config)$` (Single Template) <br/> `$(configs.<name>.config)$` (Multiple Templates) | `#@ data.values.config` (Single Template) <br/> `#@ data.values.configs.<name>.config` (Multiple Templates) |
