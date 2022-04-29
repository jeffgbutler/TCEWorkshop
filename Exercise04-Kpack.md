# Exercise 4: Configure Kpack

Kpack is a system that builds container images from source code and publishes them to a registry. Kpack uses
Cloud Native Buildpacks (https://buildpacks.io/) to build container images from source. Cloud Native Buildpacks
started as a collaboration between Heroku and Pivotal(now VMware) and are now the CNCF's recommended
method for building container images. You will never have to write another Dockerfile!

Buildpacks can inspect source code from many different languages and frameworks, determine if any
particular buildpack can make a contributions to the image, and then build the image. For Java projects,
buildpacks can recognize Gradle or Maven projects, build packs exist for many other languages including
.Net Core (C#, F#, etc.), NodeJS, Go, Ruby, etc.

Cloud NAtive Buildpacks define a standard API for creating and executing buildpacks. Another project - Paketo
Buildpacks (https://paketo.io/) - provides open source implementations of buildpacks for many different languages.

We installed Kpack when we installed the app toolkit above. In this exercise, we will configure Kpack so that
it knows how to build Java, .Net Core, and NodeJS applications.

Instructions adapted from here: https://github.com/pivotal/kpack/blob/main/docs/tutorial.md

Create a registry secret:

```powershell
kubectl create secret docker-registry kpack-registry-credentials `
    --docker-username=admin `
    --docker-password=Harbor12345 `
    --docker-server=harbor.tanzuathome.net `
    --namespace default
```

Define the Cluster stores, CLusterstacks, and builders for this cluster:

```shell
kubectl apply -f 03-kpack-resources.yaml
```

Kpack is now ready to begin building images! As with Knative, you can define image builds with a CLI, or with Kubectl.
We will show different options below - feel free to try one or all of the options!

## .Net Core Image Build with Kubectl

```shell
kubectl apply -f 04-kpack-test-image-dotnet.yaml
```

You can follow the build with this comand:

```shell
kp build logs dotnet-sample
```

Once the build completes, am image will be published. You can get the full image address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0`.
Now lets deploy that image with Knative (you will need to change this command to use the image you built):

```powershell
kn service create dotnet-sample `
  --image harbor.tanzuathome.net/tce/dotnet-sample@sha256:f2da339367a7410f6f397288d540465b1446887fc08c727efeb4ddfde8325ae0 `
  --pull-secret kpack-registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://dotnet-sample.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete dotnet-sample
```

## Java Image Build with Kubectl

Deploy a test build:

```shell
kubectl apply -f 05-kpack-test-image-java.yaml
```

You can follow the build with this comand:

```shell
kp build logs spring-pet-clinic
```

Once the build completes, am image will be published. You can get the full image address with the following command:

```shell
kp image list
```

For me, the image was `harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:57977aa16b7234080a7b9d3fdec2b663d247ee32cc2444add58e2dfd26c51b50`.
Now lets deploy that image with Knative (you will need to change this command to use the image you built):

```powershell
kn service create spring-pet-clinic `
  --image harbor.tanzuathome.net/tce/spring-pet-clinic@sha256:57977aa16b7234080a7b9d3fdec2b663d247ee32cc2444add58e2dfd26c51b50 `
  --pull-secret kpack-registry-credentials
```

Note that we have to specify the image pull secret this time as we are pulling from a private registry.

You should be able to access the application at http://spring-pet-clinic.default.127-0-0-1.nip.io/

Once you are finished experimenting, you can delete the service with the following command:

```shell
kn service delete spring-pet-clinic
```

