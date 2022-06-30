# Cartographer Template Reference

Several Cartographer templates emit values for use in other parts of the supply chain. Those values
are made available as either simple template variables, or YTT variables depending on how you configure
the receiving templates. In addition, there are "short cut" variables available if there is only one instance
of a particular template type in a supply chain. The following tables show the possibilities for
the different templates that emit values.

## ClusterSourceTemplate

A `ClusterSourceTemplate` is responsible for supplying `url` and `revision` variables to a supply chain. Typically the template
will stamp out a resource that will watch a Git repository, download the source code and make it available in the cluster.
The stamped out resource will usually update it's local copy of the source code when there is a commit.

The `source-to-url` supply chain supplied with TCE stamps out a
[Flux GitRepository](https://fluxcd.io/docs/components/source/gitrepositories/) resource to perform this function for each workload.

| Configuration | Template Variable| YTT Variable|
|---|---|---|
|spec.urlPath | &dollar;&lpar;source.url&rpar;&dollar; (Single Template) <br/> &dollar;(sources.&lt;name&gt;.url)&dollar; (Multiple Templates)| &num;&commat; data.values.source.url (Single Template) <br/> &num;&commat; data.values.sources.&lt;name&gt;.url (Multiple Templates) |
|spec.revision | &dollar;(source.revision)&dollar; (Single Template) <br/> &dollar;(sources.&lt;name&gt;.revision)&dollar; (Multiple Templates)| &num;&commat; data.values.source.revision (Single Template) <br/> &num;&commat; data.values.sources.&lt;name&gt;.revision (Multiple Templates) |

## ClusterImageTemplate

A `ClusterImageTemplate` is responsible for supplying an `image` variable to a supply chain. Typically the template will stamp
out a resource that will receive `url` and `revision` variables from a `ClusterSourceTemplate`, build an image based on the
source code, and publish the image to a repository. The stamped out resource will usually build and publish a new version of the
image when the input `url` or `revision` changes.

The `source-to-url` supply chain supplied with TCE stamps out a
[Kpack Image](https://github.com/pivotal/kpack/blob/main/docs/image.md) resource to perform this function for each workload.

| Configuration | Template Variable| YTT Variable|
|---|---|---|
|spec.imagePath | &dollar;(image)&dollar; (Single Template) <br/> &dollar;(images.&lt;name&gt;.image)&dollar; (Multiple Templates)| &num;&commat; data.values.image (Single Template) <br/> &num;&commat; data.values.images.&lt;name&gt;.image (Multiple Templates) |

## ClusterConfigTemplate

A `ClusterConfigTemplate` is responsible for supplying a `config` variable to a supply chain. Typically the template will
stamp out a resource that will read Kubernetes configurations and make their values avaiable. The stamped out resource
will make the new configuration values available when the underlying configuration changes.

The `source-to-url` supply chain supplied with TCE does not use a `ClusterConfigurationTemplate`. You could create a
`ClusterConfigurationTemplate` that stamped out a common Kubernetes ConfigMap, set some values in the ConfigMap, and them
made those values available to the supply chain.

| Configuration | Template Variable| YTT Variable|
|---|---|---|
|spec.configPath | &dollar;(config)&dollar; (Single Template) <br/> &dollar;(configs.&lt;name&gt;.config)&dollar; (Multiple Templates)| &num;&commat; data.values.config (Single Template) <br/> &num;&commat; data.values.configs.&lt;name&gt;.config (Multiple Templates) |
