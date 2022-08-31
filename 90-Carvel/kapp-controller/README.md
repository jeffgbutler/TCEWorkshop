# Kapp-Controller Overview

The kapp-controller is a Kubernetes controller that has two main functions:

1. It can run kapp in a cluster - this is the usage we will discuss in this workshop
1. It can be used to package software for easy installation in a cluster, and it understands how to install and modify
   software packages. All of the software available for TCE from VMware is packaged for easy use with the kapp-controller.
   We have already seen the package repositories and installed packages available in the TCE cluster.

Full details about the Kapp controller are here: https://carvel.dev/kapp-controller/

The kapp-controller also has an associated CLI called "kctrl" to interact with the kapp-controller installed in a cluster.

## Running Kapp in a Cluster

The kapp-controller runs kapp in a cluster. But what does that mean exactly?

When we looked at running kapp on a workstation, we saw that:

1. We needed to supply input files to kapp that contain Kubernetes YAML
1. We might want to run those input files through ytt or kbld (or both!) before we send them to kapp
1. Ultimately we want kapp to create resources in Kubernetes based on these, possibly transformed, input files

The kapp-controller does exactly this. When we install the kapp-controller in a cluster, we enable a new CRD
of kind `App` with API version `kappctrl.k14s.io/v1alpha1`. This allows us to define "applications" that the
kapp-controller will deploy and manage.

Configuration of that CRD contains three major sections:

### spec.fetch

This is where we define where to get the input YAML for the application. It can come from a variety of
sources including:

1. Inline in the YAML
1. From Git
1. From a Helm chart
1. From an arbitrary URL
1. etc.

This is the first step in what we described above as a three step process.

### spec.template

This is where we define the templating we want to apply to the input YAML. Valid templating engines include:

1. ytt
1. kbld
1. helmTemplate
1. etc.

This is the second step in what we described above as a three step process.

### spec.deploy

This is where we specify options for kapp on the deployment. This is the third step in what we described above as a three step process.

This may seem a bit abstract, so let's look at a simple example.

## Simple Application


