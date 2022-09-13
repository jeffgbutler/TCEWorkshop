# Deploy the Image with a ClusterTemplate

Currently, our supply chain can retrieve source code and build an image from it. The last step is to deploy the image
to our cluster. To make this happen, we will add a `ClusterTemplate` to the supply chain.

## About ClusterTemplate

Cartographer's `ClusterTemplate` is a template that knows how to stamp out a single Kubernetes resource. The
`ClusterTemplate` has no outputs - meaning that it cannot be a dependency for other templates. Because `ClusterTemplate`
has no outputs, it is often used as the final step in a supply chain.

The fact that `ClusterTemplate` stamps out a single Kubernetes resource may seem like a limitation. But remember that
a kapp application is a single resource that is composed of many other resources. For this reason, it is very common
to have a `ClusterTemplate` that stamps out a kapp application - and this is exactly what we will do in this section.

## Defining the Application

We will deploy the image we've built using three Kubernetes resources:

1. A Kubernetes Deployment
2. A Kubernetes Service
3. A Kubernetes Ingress 

Finally, we will use the Kapp-Controller to define these three resources as a single application.

### Defining the Deployment

Of course, we will use ytt templates to define the desired output. The following is what we'll use
for the deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: #@ data.values.workload.metadata.name
  name: #@ data.values.workload.metadata.name
spec:
  replicas: 1
  selector:
    matchLabels:
      app: #@ data.values.workload.metadata.name
  template:
    metadata:
      labels:
        app: #@ data.values.workload.metadata.name
    spec:
      containers:
      - image: #@ data.values.image
        name: #@ data.values.workload.metadata.name
```

This is a deployment template that expects two input values:

1. `data.values.workload.metadata.name` - the workload name which we will use in several places to name the deployment,
   add labels, etc.
2. `data.values.image` - the name of the image to deploy. This is a Cartographer standard variable. Cartographer
   will update this value based on the output of a `ClusterImageTemplate` in a supply chain. This variable is appropriate
   when there is only one `ClusterImageTemplate` in a supply chain. If there is more than `ClusterImageTemplate`, then
   you will need to add the template name to the variable name. See
   [Cartographer Template Reference](../06-cartographer/CartographerTemplateReference.md) for details. 

### Defining the Service

This is a ytt template for the service definition:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: #@ data.values.workload.metadata.name
  name: #@ data.values.workload.metadata.name
spec:
  ports:
  - name: default
    port: 8080
    protocol: TCP
    targetPort: #@ data.values.params.default_port
  selector:
    app: #@ data.values.workload.metadata.name
  type: ClusterIP
```

Again, this template expects two input values:

1. `data.values.workload.metadata.name` - the workload name which we will use in several places to name the service,
   add labels, etc.
2. `data.values.params.default_port` - the exposed port of the container in the deployment. We have configured this as a
   supply chain parameter to allow it to be changed per workload, and we will also set a default value of 8080

### Defining the Ingress

This is a ytt template for the ingress definition:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: #@ data.values.workload.metadata.name
  labels:
    app: ingress
spec:
  rules:
  - host: #@ data.values.workload.metadata.name + "." + data.values.params.base_domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: #@ data.values.workload.metadata.name
            port:
              number: 8080
```

Again, this template expects two input values:

1. `data.values.workload.metadata.name` - the workload name which we will use in several places to name the ingress,
   find the service, etc.
2. `data.values.params.base_domain` - the base domain for the ingress route. We will use this to calculate a unique
   route for the ingress - the workload name plus the base route. We will provide a default value based on the configuration
   of our cluster, but it can also be changed by a workload if required.

All of these templates are relatively simple, and they do make some assumptions that may not be valid in all cases. For
example, you may want to configure the ingress template to calculate routes differently. Nevertheless, these templates
are sufficient to get something deployed and working in most cases.

### Defining the Application

To define the application - which is the single thing the `ClusterTemplate` will create - we will use a template for the
kapp-controller's App CRD. Here's an example of what that looks like:

```yaml
apiVersion: kappctrl.k14s.io/v1alpha1
kind: App
metadata:
  name: #@ data.values.workload.metadata.name
spec:
  serviceAccountName: default
  fetch:
  - inline:
      paths:
        deployment.yml : #@ yaml.encode(deployment())
        service.yml : #@ yaml.encode(service())
        ingress.yml : #@ yaml.encode(ingress())
  template:
  - ytt: {}
  - kbld: {}
  deploy:
  - kapp: {}
```

This is a template that defines an App with the workload name. The deployment, service, and ingress are defined
as inline input files whose values are returned by ytt functions. This is a common method of writing App templates
because it makes the App resource relatively compact for clarity.  Here's an example of the `service()` function that
will return the template we showed above:

```yaml
#@ def/end service():
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: #@ data.values.workload.metadata.name
  name: #@ data.values.workload.metadata.name
spec:
  ports:
  - name: default
    port: 8080
    protocol: TCP
    targetPort: #@ data.values.params.default_port
  selector:
    app: #@ data.values.workload.metadata.name
  type: ClusterIP
```

To define the `ClusterTemplate` we will add all of this to the `spec.ytt` section of the template definition and this
will fully define what needs to be stamped out for each workload. You can see the final definition in
[solution/step3/ClusterTemplate.yaml](./solution/step3/ClusterTemplate.yaml)

## Updating the Supply Chain

Once we define the `ClusterTemplate`, we also need to update the supply chain:

1. We need to define the two parameters we added that are used by the template
2. We need to add the template to the supply chain resources

### Updating the Parameters

We've added two parameters to the supply chain, so we need to define them as follows:

```yaml
params:
  - name: base_domain
    default: #@ data.values.base_domain
  - name: default_port
    default: 8080
```

Notice they both have default values. The default value for `base_domain` is taken from the `values.yaml` file
we use when we deploy the supply chain with kapp.

### Updating the Supply Chain Resources

The full supply chain resources section now looks like this:

```yaml
resources:
  - name: source-provider
    templateRef:
      kind: ClusterSourceTemplate
      name: cartographer-workshop-git-repository-template

  - name: image-builder
    templateRef:
      kind: ClusterImageTemplate
      name: #@ data.values.image_template
    sources:
      - resource: source-provider
        name: source

  - name: app-deployer
    templateRef:
      kind: ClusterTemplate
      name: cartographer-workshop-app-deployer
    images:
      - name: image
        resource: image-builder
```

We've added an `app-deployer` resource that references the new `ClusterTemplate` we've defined, and we've also configured
the template to depend on the `image-builder` we defined in the last step. So Cartographer will watch for changes to the
output of the `image-builder` and update the `app-deployer` when there is a change.

## Update the RBAC

Finally, we need to update the `ClusterRole` to give the service account permission to create the new resources we've
used in this new part of the supply chain. We added the following permissions:

```yaml
rules:
  - apiGroups: [kappctrl.k14s.io]
    resources: [apps]
    verbs: ["*"]
  - apiGroups: [apps]
    resources: [deployments]
    verbs: ['*']
  - apiGroups: [networking.k8s.io]
    resources: [ingresses]
    verbs: ['*']
  - apiGroups: [""]
    resources: [services]
    verbs: ['*']
```

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

The new supply chain has stamped out a kapp-controller application named `java-payment-calcuator`. You can use the
`kctrl` CLI to inspect the application as normal. You can also use the `kapp` CLI if you desire - just remember that
for kapp, the application name will be `java-payment-calculator-ctrl`.
