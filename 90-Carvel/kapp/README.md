# Kapp Overview

The Kapp CLI understands how to install and update applications on Kubernetes.
In the Kapp way of thinking, an "application" is a collection of Kubernetes resources that can be considered
parts of a single thing.

Full details about the Kapp CLI are here: https://carvel.dev/kapp/

## Creating a Kubernetes Application

Suppose we want to install an application and make it available on our cluster. This might involve the
following:

1. Create a namespace
1. Create a deployment
1. Create a service

These resources work together to comprise the application. But they are independent resources and could be
managed independantly. If we need to make changes, or delete the application, we need to remember to work
with every resource.

Further need to make these resources in the correct order. The namespace would need to exist before the other
resources could be added to it.

```shell
kapp deploy -a kuard -f .\kuard-application\

kapp inspect -a kuard
```

[Next (Kapp-Controller Overview) -&gt;](../kapp-controller/README.md)
