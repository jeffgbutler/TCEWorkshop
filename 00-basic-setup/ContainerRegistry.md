# Plan Your Container Registry

This workshop requires that you have write access to a container registry. There are several places where registry credentials
are required.

Examples of the values required for various container registries are as follows:

| Registry                 | Kpack Default Repository                   | Cartographer Catalog Server Name | Cartographer Catalog Repository |
|--------------------------|--------------------------------------------|----------------------------------|---------------------------------|
| Azure Container Registry | foo.azurecr.io/tce/kpack                   | foo.azurecr.io                   | tce                             |
| Dockerhub                | jeffgbutler/tce/kpack [^1]                 | index.docker.io                  | jeffgbutler/tce                 |
| Harbor                   | harbor.tanzuathome.net/tce/kpack [^2]      | harbor.tanzuathome.net           | tce                             |
| Google Artifact Registry | us-east1-docker.pkg.dev/foo/tce/kpack [^3] | us-east1-docker.pkg.dev          | foo/tce                         |

[Next (Install TCE Locally) -&gt;](LocalToolInstall.md)

- or -

[Next (Install Guard Dog OVA) -&gt;](InstallGuardDogOVA.md)

[^1]: We recommend you create a repository in Dockerhub specifically for TCE. In this case we are assuming the repository
      is named "tce"

[^2]: Harbor has the concept of projects in a repository. You must create the project in Harbor before pushing artifacts.
      In this case, we have a project named "tce"

[^3]: Google Artifact repository has two levels of naming - project-id, and repository. In this case we have project-id
      of "foo" (typically your GCP user name), and a repository named "tce". The repository must be created before pushing
      artifacts.
