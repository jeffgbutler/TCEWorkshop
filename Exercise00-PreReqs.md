# Pre-Requisites

This workshop requires Docker. A package manager (Homebrew or Chocolatey) makes it easier to install some of the other
componants, but is not strictly required. We will only write instructions based on package managers.

You will also need write access to a container registry you can access from your workstation.

Details follow:

## MacOS
Install:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Homebrew: https://brew.sh/
- Knative CLI:
   ```shell
   brew install kn
   ```
- Kpack CLI:
   ```shell
   brew tap vmware-tanzu/kpack-cli
   brew install kp
   ```

## Windows
Install:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Chocolatey: https://chocolatey.org/install
- Knative CLI: https://github.com/knative/client/releases (download latest binary, rename to "kn", add to path)
- Kpack CLI: https://github.com/vmware-tanzu/kpack-cli/releases (download latest binary, rename to "kp", add to path)

## Linux
Install:

- Docker Engine: https://docs.docker.com/engine/install/
- Homebrew: https://brew.sh/
- Knative CLI:
   ```shell
   brew install kn
   ```
- Kpack CLI:
   ```shell
   brew tap vmware-tanzu/kpack-cli
   brew install kp
   ```

## Container Registry

You must also have write access to a container registry. There are several places in the workshop where you will need
to enter credentials and configuration information for your registry.

Examples of the various values are as follows:

| Registry                  | Example Server Name         | Example Tag                               |
|---------------------------|-----------------------------|-------------------------------------------|
| Azure Container Registry  | foo.azurecr.io              | foo.azurecr/kpack                         |
| Dockerhub                 | https://index.docker.io/v1/ | jeffgbutler/kpack                         |
| Harbor                    | harbor.tanzuathome.net      | harbor.tanzuathome.net/tce/kpack (1)      |
| Google Artifact Registry  | us-east1-docker.pkg.dev     | us-east1-docker.pkg.dev/foo/tce/kpack (2) |
| Google Container Registry | gcr.io                      | gcr.io/foo/kpack                          |

1. Harbor has the concept of projects in a repository. You must create the project in Harbor before pushing artifacts.
   In this case, we have a project named "tce"
2. Google Artifact repository has two levels of naming - project-id, and repository. In this case we have project-id
   of "foo" (typically your GCP user name), and a repository named "tce". The repository must be created before pushing
   artifacts.

[Next -&gt;](Exercise01-Install.md)
