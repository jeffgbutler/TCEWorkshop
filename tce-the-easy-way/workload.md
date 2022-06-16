# Deploy a Workload with the Out Of The Box Supply Chain

## Deploy a Sample Workload

### Powershell

```powershell
tanzu apps workload create java-payment-calculator `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type web `
  --label app.kubernetes.io/part-of=hello-world `
  --yes `
  -n default
```

### Linux/MacOS Shell

```shell
tanzu apps workload create java-payment-calculator \
  --git-repo https://github.com/jeffgbutler/java-payment-calculator \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=hello-world \
  --yes \
  -n default
```

## Examine the Pipeline

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

You can use the the Kubectl Tree plugin (https://github.com/ahmetb/kubectl-tree) to get a good picture of how everything is related:

```shell
kubectl tree workload java-payment-calculator
```
