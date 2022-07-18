# Test Kpack

> Important Concepts to cover in an overview:
>
> - Overview of Kpack (stacks, stores, builders)

**Important** the commands below assume you have a terminal window open in the same directory as this file.

Kpack is a system that builds container images from source code and publishes them to a registry. Kpack uses
Cloud Native Buildpacks (https://buildpacks.io/) to build container images from source. Cloud Native Buildpacks
started as a collaboration between Heroku and Pivotal(now VMware) and are now the CNCF's recommended
method for building container images. You will never have to write another Dockerfile!

Buildpacks can inspect source code from many languages and frameworks, determine if any
particular buildpack can make a contributions to the image, and then build the image. For Java projects,
buildpacks can recognize Gradle or Maven projects. Build packs exist for many other languages including
.Net Core (C#, F#, etc.), NodeJS, Go, Ruby, etc.

Cloud Native Buildpacks define a standard API for creating and executing buildpacks. Another project - Paketo
Buildpacks (https://paketo.io/) - provides open source buildpack implementations for many languages.

We installed Kpack when we installed the app toolkit previously. The default app toolkit also includes a
component called "kpack dependencies" that sets up many of the requirements to run Kpack in our cluster.

If you are interested in seeing the details of how kpack is configured, see the page [kpack-the-hard-way.md](kpack-the-hard-way.md).

As with Knative, you can define image builds with a CLI, or with Kubectl.
We're going to use the Kubectl version because of the way we've created the service account - the Kpack CLI can only work with the
default service account. Feel free to try one or all of the options below!

## .Net Core Image Build with Kubectl

Look at the file [kpack-test-image-dotnet.yaml](kpack-test-image-dotnet.yaml). This file
contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag for where the
image should be published. The file contains some placeholder values that we will fill in using YTT.

Create the image with the following command:

```shell
ytt -f config/kpack/kpack-test-image-dotnet.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

You can follow the build with this command:

```shell
kp build logs dotnet-sample
```

If you do not have the "kp" CLI installed, you can obtain the build logs with the following commands (assuming your build
pod is named the same as mine). The build works through a series of init containers before the final container "completion"
is started. The commands below show the order of execution of the containers at the time of this writing:

```shell
kubectl logs spring-pet-clinic-build-1-build-pod -c prepare
kubectl logs spring-pet-clinic-build-1-build-pod -c analyze
kubectl logs spring-pet-clinic-build-1-build-pod -c detect
kubectl logs spring-pet-clinic-build-1-build-pod -c restore
kubectl logs spring-pet-clinic-build-1-build-pod -c build
kubectl logs spring-pet-clinic-build-1-build-pod -c export
kubectl logs spring-pet-clinic-build-1-build-pod -c completion
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a .Net
SDK!). Subsequent builds would be faster. Once the build completes, an image will be published. You can get the full image
address with the following command:

```shell
kp image list
```

If you do not have the "kp" CLI installed, you can obtain the full image with the following command:

```shell
kubectl get cnbimage
```

For me, the image was `harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0`.

Now we can deploy that image with Knative (you will need to change this command to use the image you built):

Windows Powershell:
```powershell
kn service create dotnet-sample `
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 `
  --pull-secret tce-workshop-registry-secret
```

MacOS/Linux:
```shell
kn service create dotnet-sample \
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 \
  --pull-secret tce-workshop-registry-secret
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://dotnet-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete dotnet-sample
```

## Java Image Build with Kubectl

Look at the file [kpack-test-image-java.yaml](config/kpack/kpack-test-image-java.yaml) in this directory.
This file contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag
for where the image should be published. The file contains some placeholder values that we will fill in using YTT.

Create the image with the following command:

```shell
ytt -f config/kpack/kpack-test-image-java.yaml --data-values-file config/values.yaml | kubectl apply -f-
```

You can follow the build with this command:

```shell
kp build logs spring-pet-clinic
```

If you do not have the "kp" CLI installed, you can obtain the build logs with the following commands (assuming your build
pod is named the same as mine). The build works through a series of init containers before the final container "completion"
is started. The commands below show the order of execution of the containers at the time of this writing:

```shell
kubectl logs spring-pet-clinic-build-1-build-pod -c prepare
kubectl logs spring-pet-clinic-build-1-build-pod -c analyze
kubectl logs spring-pet-clinic-build-1-build-pod -c detect
kubectl logs spring-pet-clinic-build-1-build-pod -c restore
kubectl logs spring-pet-clinic-build-1-build-pod -c build
kubectl logs spring-pet-clinic-build-1-build-pod -c export
kubectl logs spring-pet-clinic-build-1-build-pod -c completion
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a Java
SDK!). Once the build completes, an image will be published. You can get the full image address with the following command:

```shell
kp image list
```

If you do not have the "kp" CLI installed, you can obtain the full image with the following command:

```shell
kubectl get cnbimage
```

For me, the image was `harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:d150b6b0b9a28d72795efb2f9e6c1a353eeec84691e965cb110d787215f8941c`.
Now lets deploy that image with Knative (you will need to change this command to use the image you built):

Windows Powershell:
```powershell
kn service create spring-pet-clinic `
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:d150b6b0b9a28d72795efb2f9e6c1a353eeec84691e965cb110d787215f8941c `
  --pull-secret tce-workshop-registry-secret
```

MacOS/Linux:
```shell
kn service create spring-pet-clinic \
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:d150b6b0b9a28d72795efb2f9e6c1a353eeec84691e965cb110d787215f8941c \
  --pull-secret tce-workshop-registry-secret
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://spring-pet-clinic.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete spring-pet-clinic
```

[Next (Explore Cartographer) -&gt;](../06-cartographer/README.md)
