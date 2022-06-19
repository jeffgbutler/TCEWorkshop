# Tanzu Community Edition Pre-Requisites

This workshop requires Docker and several other tools. Using package manager (Homebrew or Chocolatey) makes it
easier to install some of the componants, but is not strictly required. We will only write instructions based on
package managers.

You will need to have Internet access to run the exercises in the workshop.

You will also need write access to a container registry you can access from your workstation. Dockerhub certainly works, but
you might experience rate limiting if you are on an unpaid Docker account.

To complete the pre-requisites, do the following:

1. Follow the platform specific initial setup instructions below for your platform:
   [MacOS](#initial-setup---macos), [Windows](#initial-setup---windows), or [Linux](#initial-setup---linux)
2. Follow the instructions in [Container Registry Setup](#container-registry-setup)

## Initial Setup - MacOS
Install required tools:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Homebrew: https://brew.sh/
- YTT (YAML Templating Tool):
   ```shell
   brew install ytt
   ```
- Kapp (Carvel Kapp CLI):
   ```shell
   brew install kapp
   ```
- Knative CLI:
   ```shell
   brew install kn
   ```
- Kpack CLI - for Intel Macs Only!!!:
   ```shell
   brew tap vmware-tanzu/kpack-cli
   brew install kp
   ```

Proceed to [Container Registry Setup](#container-registry-setup)

## Initial Setup - Windows
Install required tools:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Chocolatey: https://chocolatey.org/install
- YTT (YAML Templating Tool):
   ```shell
   choco install ytt
   ```
- Kapp (Carvel Kapp CLI):
   ```shell
   choco install kapp
   ```
- Knative CLI: Go to https://github.com/knative/client/releases, download latest Windows binary, rename it to "kn", than add it to the path
- Kpack CLI: Go to https://github.com/vmware-tanzu/kpack-cli/releases, download latest Windows binary, rename it to "kp", add it to the path

Proceed to [Container Registry Setup](#container-registry-setup)

## Initial Setup - Linux
Install required tools:

- Git: https://git-scm.com/downloads
- Docker Engine: https://docs.docker.com/engine/install/
- Homebrew: https://brew.sh/
- YTT (YAML Templating Tool):
   ```shell
   brew install ytt
   ```
- Kapp (Carvel Kapp CLI):
   ```shell
   brew install kapp
   ```
- Knative CLI:
   ```shell
   brew install kn
   ```
- Kpack CLI:
   ```shell
   brew tap vmware-tanzu/kpack-cli
   brew install kp
   ```

Proceed to [Container Registry Setup](#container-registry-setup)

## Container Registry Setup

You must also have write access to a container registry. There are several places where registry credentials
are required.

Examples of the values required for various container registries are as follows:

| Registry                  | Cartographer Server Name | Kpack Default Repository                   | Cartographer Repository |
|---------------------------|--------------------------|--------------------------------------------|-------------------------|
| Azure Container Registry  | foo.azurecr.io           | foo.azurecr.io/tce/kpack                   | tce                     |
| Dockerhub                 | index.docker.io          | jeffgbutler/tce/kpack [^1]                 | jeffgbutler/tce         |
| Harbor                    | harbor.tanzuathome.net   | harbor.tanzuathome.net/tce/kpack [^2]      | tce                     |
| Google Artifact Registry  | us-east1-docker.pkg.dev  | us-east1-docker.pkg.dev/foo/tce/kpack [^3] | foo/tce                 |

[Next -&gt;](InstallTCE.md)

[^1]: We recommend you create a repository in Dockerhub specifically for TCE. In this case we are assuming the repository is named "tce"

[^2]: Harbor has the concept of projects in a repository. You must create the project in Harbor before pushing artifacts.
      In this case, we have a project named "tce"

[^3]: Google Artifact repository has two levels of naming - project-id, and repository. In this case we have project-id
      of "foo" (typically your GCP user name), and a repository named "tce". The repository must be created before pushing
      artifacts.
