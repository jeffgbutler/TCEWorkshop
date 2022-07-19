# Deploying Applications with Knative

[Knative](https://knative.dev/) is an open source Kubernetes subproject with two primary components:

1. Knative serving is a serverless framework for Kubernetes. It includes scale-to-zero capabilities, but also
   has capabilities that make it easy to deploy many types of web applications on Kubernetes
2. Knative eventing is a framework for building event driven applications on Kubernetes

In this workshop, we will focus on Knative serving as that is the part of Knative that is packaged for easy
installation on a TCE cluster.

**Important** the commands below assume you have a terminal window open in the same directory as this file.

Let's deploy an application with Knative as a test. This can be done in two ways - with the Knative CLI or with Kubectl
and a YAML file.

## Deploy a Knative Application with the Knative CLI

The Knative CLI is a very simple way to deploy an application to Knative. It does not have as much
flexibility as using the full Kubectl/YAML method, but it does have sensible defaults for many situations.
Let's deploy Kuard with Knative:

```shell
kn service create kuard --image gcr.io/kuar-demo/kuard-amd64:blue
```

Once the command completes, the application should be available at http://kuard.default.127-0-0-1.nip.io/ (substitute
your domain name for `127-0-0-1.nip.io` if you changed the default during installation)

Take a look at what got created in your cluster:

```shell
kubectl get all
```

You should see pods, services, a deployment, a replica set, and several other objects related to Knative. You can see
that Knative is doing quite a lot with a simple command! Knative has also generated a URL for this application and
configured the ingress controller (Contour in this case). By default, the URL is calculated as
`app_name.namespace.base_domain` - where `base_domain` is what we configured when we installed the app toolkit
(`127-0-0-1.nip.io` in this case).

By default, Knative sets up "scale to zero" functionality for the services it deploys. This because Knative started
as a "serverless" toolkit for Kubernetes. You can see this in action by watching the pod - it will eventually go away
and only the other Knative objects will remain. Once the pod is gone, you can still hit the application, but it
will be slow initially as Knative will spin up a new pod once traffic starts flowing again.

Once you are finished experimenting with Knative, you can delete the service with the following command:

```shell
kn service delete kuard
```

## Deploy a Knative Application with Kubectl

The file [kuard-service.yaml](kuard-service.yaml) contains YAML for achieving the
same deployment of Kuard we did above with one exception - we turn off "scale to zero" functionality. This can
also be accomplished with the CLI (see the `--scale-min` parameter).

Deploy the service with Kubectl:

```shell
kubectl apply -f kuard-service.yaml
```

The app should be available at http://kuard.default.127-0-0-1.nip.io/

Everything should be the same as before except that with this deployment the app will not scale to zero.

Once you are finished experimenting, you can delete the service with the CLI as before, or also with Kubectl: 

```shell
kn service delete kuard

kubectl delete -f config/kuard-service.yaml
```

Knative has many features and configuration options. For details see the official documentation
here: https://knative.dev/docs/

[Next (Explore Kpack) -&gt;](../05-kpack/README.md)
