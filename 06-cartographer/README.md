# Exploring Cartographer

The information and commands in this section assume you have created and configured a cluster, and have
installed the app toolkit.

TCE ships with an "out of the box" supply chain does three things:

1. Watch a git repo for source code changes
1. Build an image based on the current source code and push the image into a repository
1. Deploy the image as a Knative application using Kapp

We'll take a closer look at how this all fits together.

## Deploy a Sample Workload

### Powershell

```powershell
tanzu apps workload create java-payment-calculator `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type web `
  --label app.kubernetes.io/part-of=java-payment-calculator `
  --yes `
  -n default
```

### Linux/MacOS Shell

```shell
tanzu apps workload create java-payment-calculator \
  --git-repo https://github.com/jeffgbutler/java-payment-calculator \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=java-payment-calculator \
  --yes \
  -n default
```

## Examine the Supply Chain

Tail the pipeline logs...

```shell
tanzu apps workload tail java-payment-calculator
```

Get workload information...

```shell
tanzu apps workload get java-payment-calculator
```

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
information about the application. With the following commands:

```shell
kapp list

kapp inspect -a java-payment-calculator-ctrl
```

You can also look at the Kapp application object with a command like this:

```shell
kubectl describe app java-payment-calculator
```

You can use the the Kubectl Tree plugin to get a good picture of how everything is related:

```shell
kubectl tree workload java-payment-calculator
```

## Access the Application

Once the supply chain completes, you should be able to access the application in the cluster.
If you are using an unmanaged cluster, the application should be available at http://java-payment-calculator.default.127-0-0-1.nip.io.
If you are using a real domain on a managed cluster, the application should be available
at http://java-payment-calculator.default.YOUR_DOMAIN

[Next (Cartographer Deep Dive) -&gt;](CartographerDeepDive.md)
