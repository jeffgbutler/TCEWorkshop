# Kbld Overview

Kbld (k build) is a simple tool that does two main things...

## Image Building

Kbld can orchestrate image builds for local development. This can be useful if you are doing Dockerfile or Pack
based local development. In this way of working, kbld can do the following:

1. Call Docker or Pack to build an image
1. Push the image to a registry
1. Update a Kubernetes YAML with a tag that uniquely represents the new image

You can see how this could be useful when rapidly iterating during local development. However, in the Tanzu world we typically
want to use a Cartographer supply chain for this type of workflow, so this is not how we'll use kbld in this workshop.

## Image Resolution

In a very simple usage, kbld will seach Kubernetes YAML for image references and will transform the YAML to use
digests instead of labels. This can be useful to convert Kubernetes YAML to use immutable image references.

For example, suppose we have a simple Kubernetes pod definition like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

We can run kbld with a command like this:

```shell
kbld -f nginx-pod.yaml
```

The output will look like this:

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kbld.k14s.io/images: |
      - origins:
        - resolved:
            tag: latest
            url: nginx
        url: index.docker.io/library/nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f
  labels:
    name: nginx
  name: nginx
spec:
  containers:
  - image: index.docker.io/library/nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f
    name: nginx
    resources:
      limits:
        cpu: 500m
        memory: 128Mi
```

Notice two things:

1. The `spec.containers.image` reference was changed to use the digest of the nginx image. Note that we are using the "latest"
   tag so the digest may be different if you run this command
1. Kbld added an annotation `kbld.k14s.io/images` that details the before and after versions that were transformed

Kbld can accept multiple input files, and can also transform all the YAML in a directory. The output will be a single
consolidated YAML composed of all the input files. Foe example, you could transform all YAML in a directory
and pipe it to Kubectl with a command like this:

```shell
kbld -f . | kubectl apply -f-
```

[Next (Kapp Overview) -&gt;](../kapp/README.md)
