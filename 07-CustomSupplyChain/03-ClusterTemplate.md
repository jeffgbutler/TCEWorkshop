# Deploy the Image with a ClusterTemplate

Currently, our supply chain can retrieve source code and build an image from it. The last step is to deploy the image
to our cluster. To make this happen, we will add a `ClusterTemplate` to the supply chain.

TODO

```powershell
tanzu apps workload create source-to-deployment-template `
  --git-repo https://github.com/jeffgbutler/java-payment-calculator `
  --git-branch main `
  --type source-to-deployment-template `
  --yes `
  -n default
```


kapp deploy -a nginx-ingress -f ..\03-app-toolkit\contour-test-ingress.yaml
