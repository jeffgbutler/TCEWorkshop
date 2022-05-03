# Exercise 5: Configure and Test Kpack

> Important Concepts to cover in an overview:
>
> - Overview of Kpack (stacks, stores, builders)

**Important!** Shell commands on this page should be executed from a terminal open in the root directory of the
git repo you cloned in the pre-requisites!

Kpack is a system that builds container images from source code and publishes them to a registry. Kpack uses
Cloud Native Buildpacks (https://buildpacks.io/) to build container images from source. Cloud Native Buildpacks
started as a collaboration between Heroku and Pivotal(now VMware) and are now the CNCF's recommended
method for building container images. You will never have to write another Dockerfile!

Buildpacks can inspect source code from many different languages and frameworks, determine if any
particular buildpack can make a contributions to the image, and then build the image. For Java projects,
buildpacks can recognize Gradle or Maven projects. Build packs exist for many other languages including
.Net Core (C#, F#, etc.), NodeJS, Go, Ruby, etc.

Cloud Native Buildpacks define a standard API for creating and executing buildpacks. Another project - Paketo
Buildpacks (https://paketo.io/) - provides open source buildpack implementations for many different languages.

We installed Kpack when we installed the app toolkit previously. In this exercise, we will configure Kpack so that
it knows how to build Java, .Net Core, and NodeJS applications.

Instructions adapted from here: https://github.com/pivotal/kpack/blob/main/docs/tutorial.md

Take a look at the file [config/kpack/kpack-resources.yaml](config/kpack/kpack-resources.yaml). This file contains definitions
for the Kpack resources required in this workshop (Secret, ServiceAccount, ClusterStore, ClusterStack, and Builder). You should be able to
see that we are installing buildpacks for .Net core, Java, and NodeJS. The file contains some placeholder values that we will fill
in using YTT. The following command will merge the configuration values with the template, and then pass the results to kubectl.
A neat trick!

```shell
ytt -f config/kpack/kpack-resources.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

Kpack is now ready to begin building images! As with Knative, you can define image builds with a CLI, or with Kubectl.
We're going to use the Kubectl version because of the way we've created the service account - the Kpack CLI can only work with the
default service account. Feel free to try one or all of the options below!

## .Net Core Image Build with Kubectl

Look at the file [config/kpack/kpack-test-image-dotnet.yaml](config/kpack/kpack-test-image-dotnet.yaml). This file
contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag for where the
image should be published. The file contains some placeholder values that we will fill in using YTT.

Create the image with the following command:

```shell
ytt -f config/kpack/kpack-test-image-dotnet.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

You can follow the build with this comand:

```shell
kp build logs dotnet-sample
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a .Net
SDK!). Subsequent builds would be faster. Once the build completes, an image will be published. You can get the full image
address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0`.

Now we can deploy that image with Knative (you will need to change this command to use the image you built):

Windows Powershell:
```powershell
kn service create dotnet-sample `
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 `
  --pull-secret kpack-registry-credentials
```

MacOS/Linux:
```shell
kn service create dotnet-sample \
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 \
  --pull-secret kpack-registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://dotnet-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete dotnet-sample
```

## Java Image Build with Kubectl

Look at the file [config/kpack/kpack-test-image-java.yaml](config/kpack/kpack-test-image-java.yaml) in this directory.
This file contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag
for where the image should be published. The file contains some placeholder values that we will fill in using YTT.

Create the image with the following command:

```shell
ytt -f config/kpack/kpack-test-image-java.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

You can follow the build with this comand:

```shell
kp build logs spring-pet-clinic
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a Java
SDK!). Once the build completes, an image will be published. You can get the full image address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:57977aa16b7234080a7b9d3fdec2b663d247ee32cc2444add58e2dfd26c51b50`.
Now lets deploy that image with Knative (you will need to change this command to use the image you built):

Windows Powershell:
```powershell
kn service create spring-pet-clinic `
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:57977aa16b7234080a7b9d3fdec2b663d247ee32cc2444add58e2dfd26c51b50 `
  --pull-secret kpack-registry-credentials
```

MacOS/Linux:
```shell
kn service create spring-pet-clinic \
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:57977aa16b7234080a7b9d3fdec2b663d247ee32cc2444add58e2dfd26c51b50 \
  --pull-secret kpack-registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://spring-pet-clinic.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete spring-pet-clinic
```

[&lt;- Previous](04-Knative.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Next -&gt;](06-Cartographer.md)
