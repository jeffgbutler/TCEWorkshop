# Kapp-Controller Overview

The kapp-controller is a Kubernetes controller that has two main functions:

1. It can run kapp in a cluster - this is the usage we will discuss in this workshop
2. It can be used to package software for easy installation in a cluster, and it understands how to install and modify
   software packages. All the software available for Tanzu from VMware is packaged for easy use with the kapp-controller.
   If you are interested in learning more about package authoring, there is a good introductory post on the Carvel site
   here: https://carvel.dev/blog/kctrl-pkg-authoring-cmds/

Every Tanzu cluster has the kapp-controller pre-installed. In fact, having the kapp-controller installed is one of the
defining characteristics of a "Tanzu cluster" (the other being the secretgen-controller). But the kapp-controller can be
easily installed and used on any Kubernetes cluster - Tanzu or not.

Full details about the Kapp controller are here: https://carvel.dev/kapp-controller/

The kapp-controller also has an associated CLI called "kctrl" that can interact with the kapp-controller installed in a
cluster.

## Running Kapp in a Cluster

The kapp-controller runs kapp in a cluster. But what does that mean exactly?

When we ran kapp on a workstation, we saw that:

1. We supply input files to kapp that contain Kubernetes YAML
2. We might want to run those input files through ytt or kbld (or both!) before we send them to kapp
3. Ultimately we want kapp to create resources in Kubernetes based on these, possibly transformed, input files

The kapp-controller does exactly this. When the kapp-controller is installed in a cluster, there is a new CRD
of kind `App` with API version `kappctrl.k14s.io/v1alpha1`. This allows us to define input sources for kapp and
transforms that should occur on those inputs before kapp creates the resources. Since the kapp-controller is
Kubernetes native, any change to the definition of an App will cause the controller to reconcile again. So you
can imagine a scenario where a pull request to a Git repository will be picked up by the kapp-controller and the
app will be updated accordingly.

Configuration of the App CRD contains three major sections:

1. `spec.fetch` where we define input sources for YAML or templates. Inputs can come from a variety of
   sources including:
   - Inline in the App spec
   - From Git
   - From a Helm chart
   - From an arbitrary URL
   - others
2. `spec.template` where we define the transforms we want to apply to the input YAML. Valid templating engines include:
   - ytt
   - kbld
   - helmTemplate
   - others
3. `spec.deploy` where we specify options for kapp on the deployment

This may seem a bit abstract, so we will walk through a simple example. But first we need to look at security.

## Security for the Kapp Controller

The kapp-controller will do it's work using a service account that you specify when you create the App spec. You will need
to make sure that the service account has permission to all the API resources that can be created or updated by the app.
The file [rbac.yaml](rbac.yaml) in this directory contains a `ClusterRole` with permissions for all the API resources used in
these examples. It also includes a `ClusterRoleBinding` for the default service account in the default namespace. These are
cluster resources rather than namespaced resources because we want the kapp-controller to be able to work across namespaces.

In addition to permissions for the resources you create, the kapp-controller service account also needs permissions
to create ConfigMaps - which makes sense as that is where kapp stores application configuration. You can read more
about the kapp-controller security model here: https://carvel.dev/kapp-controller/docs/v0.40.0/security-model/

## Simple Application

This is a very simple App spec:

```yaml
apiVersion: kappctrl.k14s.io/v1alpha1
kind: App
metadata:
  name: kuard-kapp
spec:
  serviceAccountName: default
  fetch:
    - inline:
        paths:
          deployment.yaml: |
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              labels:
                app: kuard
              name: kuard
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: kuard
              template:
                metadata:
                  labels:
                    app: kuard
                spec:
                  containers:
                  - image: gcr.io/kuar-demo/kuard-amd64:blue
                    name: kuard
  template:
    - kbld: {}
  deploy:
    - kapp: {}
```

Important things to notice in this spec:

1. `spec.fetch` has a single hardcoded deployment spec. It's in a "file" named "deployment.yaml" but that is really
   just to keep it separate from other hardcoded entries. There can be as many inline entries as you wish.
2. The kapp-controller will run this input through kbld before applying it. This is specified by the following YAML:

   ```yaml
   template:
     - kbld: {}
   ```

   That looks a little strange, but it is basically telling the kapp controller we want to run kbld. You can specify
   some options for kbld if desired that roughly correspond to command line options on the kbld CLI. We don't need to
   specify anything in this case, so we just supply an empty map (`{}`)

3. `spec.deploy` has `kapp` as it's only value. Again we can specify kapp command line options if desired, but we don't need
   any here, so we supply an empty map

This spec creates an App resource that runs kapp and deploys the application. The App resource will check for updates every 30 seconds
or so and make changes to the app. Very few changes are expected here since everything is hardcoded. If the label on the Kuard
image changes it would trigger an update because kbld would move to the new digest.

To deploy this, we need to run rbac.yaml as well as simple-example.yaml. Sounds like a job for kapp! Yes - we can use
kapp to deploy an application based on the kapp-controller.

```shell
kapp deploy -a kapp-controller-simple-example -f rbac.yaml -f simple-example.yaml
```

## Inspecting Applications with the kapp CLI

The kapp-controller runs kapp, so we would expect the kapp CLI to work on applications deployed through the kapp-controller.
And it does. The only caveat is that the application name is not what you might expect. In the example above, we created
an App resource names "kuard-kapp". If you do "kapp list" you will see that the application name is "kuard-kapp-ctrl". The
kapp-controller appends "-ctrl" to all the application names it creates.

Once you know that, you can use any kapp CLI command. For example:

```shell
kapp logs -a kuard-kapp-ctrl
```

## Working With the kctrl CLI

The kapp-controller's CLI is "kctrl". We'll show several useful commands below.

List all applications managed by the kapp-controller:

```shell
kctrl app list
```

Get current application details:

```shell
kctrl app get -a kuard-kapp
```

Get current application status (shows how many reconciliations have happened):

```shell
kctrl app status -a kuard-kapp
```

Pause reconciliation:

```shell
kctrl app pause -a kuard-kapp
```

Force an app to reconcile again:

```shell
kctrl app kick -a kuard-kapp
```

## More Complete Example

The file [complex-example.yaml](complex-example.yaml) in this directory contains a more complete example of using the
kapp-controller. It has the following characteristics:

1. There is inline YAML for a namespace, deployment, and service. These are the same files we used from the
   kapp exercise - they are ytt templates that accept configuration values. There is also inline YAML for the ytt
   schema with default values. This is basically copy/paste of existing YAML templates into an App spec.
2. There is a reference to a Git repository to obtain configuration values. In this case, the reference directory has a single
   file named "values.yaml" that looks something like this:

   ```yaml
   #@data/values
   ---
   namespace: kuard-app-ns
   replicas: 3
   ```
3. The kapp-controller is configured to run both ytt and kbld on the input files before deploying the application with kapp

You can deploy this application with the following command:

```shell
kubectl apply -f complex-example.yaml
```

Note that the rbac.yaml file isn't necessary if you haven't deleted the previous app. Once the app has reconciled,
you will see the deployment in the "kuard-app-ns" namespace:

```shell
kubectl get all -n kuard-app-ns
```

If you want to experiment with this, then we suggest you change the Git reference to a repo where you can commit. Then
deploy the application and watch it create all the resources you expect. If you make a change to the configuration in Git,
then you should see the change reflected in your cluster in a minute or so.

## Cleanup

```shell
kapp delete -a kapp-controller-simple-example
```

```shell
kctrl app delete kuard-kapp-gitops
```
