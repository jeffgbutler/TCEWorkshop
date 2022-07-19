# Test Kpack

**Important** the commands below assume you have a terminal window open in the same directory as this file.

Kpack is a system that builds container images from source code and publishes them to a registry. Kpack uses
Cloud Native Buildpacks (https://buildpacks.io/) to build container images from source. Cloud Native Buildpacks
started as a collaboration between Heroku and Pivotal(now VMware) and are now the CNCF's recommended
method for building container images.

Buildpacks can inspect source code from many languages and frameworks, determine if any
particular buildpack can make a contributions to the image, and then build the image. For Java projects,
buildpacks can recognize Gradle or Maven projects. Build packs exist for many other languages including
.Net Core (C#, F#, etc.), NodeJS, Go, Ruby, etc.

Cloud Native Buildpacks define a standard API for creating and executing buildpacks. Another project - Paketo
Buildpacks (https://paketo.io/) - provides open source buildpack implementations for many languages.

We installed Kpack when we installed the app toolkit previously. The default app toolkit also includes a
component called "kpack dependencies" that sets up many of the requirements to run Kpack in our cluster.

As with Knative, you can define image builds with a CLI, or with Kubectl.
We're going to use the Kubectl version because of the way we've created the service account - the Kpack CLI can only work with the
default service account. Feel free to try one or all of the options below!

## Exploring Kpack

Display cluster stores installed and their versions:

```shell
kp clusterstore list
```

Display detailed information about a cluster store:

```shell
kp clusterstore status default -v
```

Display cluster stacks installed and their versions:

```shell
kp clusterstack list
```

Display detailed information about a cluster stack:

```shell
kp clusterstack status default -v
```

Display cluster builders installed and their versions...

```shell
kp clusterbuilder list
```

Display detailed information about a cluster builder...

```shell
kp clusterbuilder status default
```


## .Net Core Image Build with Kpack CLI

If you have the Kpack CLI installed, you can create images from the command line:

Powershell...

```powershell
kp image create dotnet-sample `
 --tag harbor.tanzuathome.net/tce/kpack-dotnet-sample `
 --git https://github.com/paketo-buildpacks/samples `
 --git-revision main `
 --sub-path ./dotnet-core/aspnet
```

MacOS/Linux Shell...

```shell
kp image create dotnet-sample \
 --tag harbor.tanzuathome.net/tce/kpack-dotnet-sample \
 --git https://github.com/paketo-buildpacks/samples \
 --git-revision main \
 --sub-path ./dotnet-core/aspnet
```

You can follow the build with this command:

```shell
kp build logs dotnet-sample
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a .Net
SDK!). Subsequent builds would be faster. Once the build completes, an image will be published. You can get the full image
address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/kpack-dotnet-sample@sha256:e64109703a5293b55882d524d98690ebb73d6d775fd60c286149b1f360de5eba`.

Now we can deploy that image with Knative (you will need to change this command to use the image you built):

Powershell:

```powershell
kn service create dotnet-sample `
  --image harbor.tanzuathome.net/tce/kpack-dotnet-sample@sha256:e64109703a5293b55882d524d98690ebb73d6d775fd60c286149b1f360de5eba `
  --pull-secret registry-credentials
```

MacOS/Linux Shell:

```shell
kn service create dotnet-sample \
  --image harbor.tanzuathome.net/tce/kpack-dotnet-sample@sha256:e64109703a5293b55882d524d98690ebb73d6d775fd60c286149b1f360de5eba \
  --pull-secret registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://dotnet-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete dotnet-sample
```

## .Net Core Image Build with Kubectl

Look at the file [kpack-test-image-dotnet.yaml](kpack-test-image-dotnet.yaml). This file
contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag for where the
image should be published.

**Important:** You must change the image tag in this file before proceeding!

Create the image with the following command:

```shell
kubectl apply -f kpack-test-image-dotnet.yaml
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
  --pull-secret registry-credentials
```

MacOS/Linux:
```shell
kn service create dotnet-sample \
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 \
  --pull-secret registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://dotnet-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete dotnet-sample
```

## Java Image Build with Kpack CLI

If you have the Kpack CLI installed, you can create images from the command line:

Powershell...

```powershell
kp image create spring-pet-clinic `
 --tag harbor.tanzuathome.net/tce/kpack-java-sample `
 --git https://github.com/spring-projects/spring-petclinic `
 --git-revision 82cb521d636b282340378d80a6307a08e3d4a4c4
```

MacOS/Linux Shell...

```shell
kp image create spring-pet-clinic \
 --tag harbor.tanzuathome.net/tce/kpack-java-sample \
 --git https://github.com/spring-projects/spring-petclinic \
 --git-revision 82cb521d636b282340378d80a6307a08e3d4a4c4
```

You can follow the build with this command:

```shell
kp build logs spring-pet-clinic
```

The build will be slow the first time it runs because the buildpack will need to download all the dependencies (like a .Net
SDK!). Subsequent builds would be faster. Once the build completes, an image will be published. You can get the full image
address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/kpack-java-sample@sha256:7eb23df634e494ecab677434c506b1ad63f42225dd29f86b8848f45cace942b1`.

Now we can deploy that image with Knative (you will need to change this command to use the image you built):

Powershell:

```powershell
kn service create java-sample `
  --image harbor.tanzuathome.net/tce/kpack-java-sample@sha256:7eb23df634e494ecab677434c506b1ad63f42225dd29f86b8848f45cace942b1 `
  --pull-secret registry-credentials
```

MacOS/Linux Shell:

```shell
kn service create java-sample \
  --image harbor.tanzuathome.net/tce/kpack-java-sample@sha256:7eb23df634e494ecab677434c506b1ad63f42225dd29f86b8848f45cace942b1 \
  --pull-secret registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://java-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete java-sample
```

## Java Image Build with Kubectl

Look at the file [kpack-test-image-java.yaml](kpack-test-image-java.yaml) in this directory.
This file contains a definition for a Kpack "Image" - importantly it contains a path to the source code in Git, and a tag
for where the image should be published.

**Important:** You must change the image tag in this file before proceeding!

Create the image with the following command:

```shell
kubectl apply -f kpack-test-image-java.yaml
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
  --pull-secret registry-credentials
```

MacOS/Linux:
```shell
kn service create spring-pet-clinic \
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:d150b6b0b9a28d72795efb2f9e6c1a353eeec84691e965cb110d787215f8941c \
  --pull-secret registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://spring-pet-clinic.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete spring-pet-clinic
```

[Next (Explore Cartographer) -&gt;](../06-cartographer/README.md)
