# Quick Start

If you already have the prerequisites installed and want to get straight into Knative, Kpack, or Cartographer without
going through the background information about Tanzu Community Edition, you can create and prepare an unmanaged cluster
on Kind with three simple commands:

Create a cluster:

```shell
tanzu unmanaged-cluster create tceworkshop --port-map '80:80,443:443'
```

Create a secret for your registry. On an unmanaged cluster, all you need is a simple secret named `registry-credentials`.
The following is an example for my private Harbor instance. Change the secret to match your image repository:

<details><summary>Powershell</summary>
<p>

```powershell
kubectl create secret docker-registry registry-credentials `
  --docker-server=harbor.tanzuathome.net `
  --docker-username=admin `
  --docker-password=Harbor12345
```

</p>
</details>

```shell
kubectl create secret docker-registry registry-credentials \
  --docker-server=harbor.tanzuathome.net \
  --docker-username=admin \
  --docker-password=Harbor12345
```

Install the app-toolkit. First modify the values in
[03-app-toolkit/config/app-toolkit-values.yaml](03-app-toolkit/config/app-toolkit-values.yaml) to match your image
repository, then install the app toolkit with the following command:

```powershell
tanzu package install app-toolkit `
  --package-name app-toolkit.community.tanzu.vmware.com `
  --version 0.2.0 `
  --values-file 03-app-toolkit/config/app-toolkit-values.yaml
```

Once the app-toolkit reconciles, you can proceed with any subsequent exercise.

[Explore Knative -&gt;](04-knative/README.md)

[Explore Kpack -&gt;](05-kpack/README.md)

[Explore Cartographer -&gt;](06-cartographer/README.md)

[Create a Custom Supply Chain with Cartographer -&gt;](07-CustomSupplyChain/README.md)
