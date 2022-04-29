# Exercise 3: Configure and Install the App Toolkit

> Important Concepts to cover in an overview:
>
> - Overview of the App Toolkit Package

The App Toolkit is a composite package (it bundles other packages together). It is a simple way to install
a group of packages that are relevant to application developers including:

- Contour - a Kubernetes ingress controller
- Kpack - an image building tool
- Knative Serving - a Kubernetes framework that simplifies application deployments
- Cartographer - a tool for creating software supply chains
- Several others

With this exercise we will install and configure the app toolkit for use on a local workstation. Take a look
at the file [01-app-toolkit-values.yaml](01-app-toolkit-values.yaml) in this directory. This file contains
configuration values for several of the packages in the app toolkit (Contour, Knative, and Kpack). For now, the
important things to know are these:

- Applications deployed to the app toolkit will have generated DNS names that are sub-domains of
  `127-0-0-1.nip.io`. This is a convenient DNS trick - the IP address for this domain is
  `127.0.0.1` - or `localhost` - so it will work on everyone's workstation.
- Kpack is configured to talk to a Harbor instance in Jeff's homelab. You will need to change these
  values to match a container registry you have access to

Once you have updated the default repository values, install the app toolkit with the following command:

```powershell
tanzu package install app-toolkit --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.1.0 --values-file 01-app-toolkit-values.yaml
```

This command will run for a few minutes. Once the package install finishes reconciling, you can see the full list
of installed packages with the following command:

```shell
tanzu package installed list -A
```

Let's deploy an application with Knative as a test. This can be done in two ways - with the Knative CLI or with Kubectl
and a YAML file.

## Deploy a Knative Application with the Knative CLI

The Knative CLI is a very simple way to deploy an application to Knative. It does not have as much
flexability as using the full Kubactl/YAML method, but it does have sensable defaults for many situations.
Let's deploy Kuard with Knative:

```shell
kn service create kuard --image gcr.io/kuar-demo/kuard-amd64:blue
```

Once the command completes, the application should be available at http://kuard.default.127-0-0-1.nip.io/

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
and only the other Knative objects will remain. Once the pod is gone, you can still hit the application but it
will be slow initially as Knative will spin up a new pod once traffic starts flowing again.

Once you are finished experimenting with Knative, you can delete the service with the following command:

```shell
kn service delete kuard
```

## Deploy a Knative Application with Kubectl

The file [02-kuard-service.yaml](02-kuard-service.yaml) in this directory contains YAML for acheiving the
same deployment of Kuard we did above with one exception - we turn off "scale to zero" functionality. This can
also be accomplished with the CLI (see the `--scale-min` parameter).

Deploy the service with Kubectl:

```shell
kubectl apply -f 02-kuard-service.yaml
```

The app should be avilable at http://kuard.default.127-0-0-1.nip.io/

Everything should be the same as before except that with this deployment the app will not scale to zero.

Once you are finished experimenting, you can delete the service with the CLI as before, or also with Kubectl: 

```shell
kn service delete kuard

kubectl delete -f 02-kuard-service.yaml
```

Knative has many features and configuration options. For details see the official documentation
here: https://knative.dev/docs/

