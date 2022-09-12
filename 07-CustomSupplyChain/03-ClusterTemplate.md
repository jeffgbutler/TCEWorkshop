# Deploy the Image with a ClusterTemplate

Currently, our supply chain can retrieve source code and build an image from it. The last step is to deploy the image
to our cluster. To make this happen, we will add a `ClusterTemplate` to the supply chain.

TODO

## Create and Test the Updated Supply Chain

Update the supply chain with this command:

```shell
ytt -f ./solution/step3/. --data-values-file ./solution/values.yaml | kapp deploy -a cartographer-workshop-supply-chain -y -f-
```

You can watch the workload update with

```shell
tanzu apps workload tail java-payment-calcuator
```

This time the supply chain should create a deployment, service, and ingress. You should be able to navigate to the application
at http://java-payment-calculator.127-0-0-1.nip.io/ (or whatever domain you configured on TAP).
