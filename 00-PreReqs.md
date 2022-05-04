# Pre-Requisites

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

## Initial Setup - Windows
Install required tools:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Chocolatey: https://chocolatey.org/install
- YTT (YAML Templating Tool):
   ```shell
   choco install ytt
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

You must also have write access to a container registry. There are several places in the workshop where registry credentials
are required. We will use the YTT tool to reduce the amount of typing of credentials.

First, clone the repo:

```shell
git clone https://github.com/jeffgbutler/TCEWorkshop.git
```

Now edit the `config/values.yaml` file and supply values for your registry. In that file are four
values that you must change:

1. `registry.server` - the server where your registry is installed
2. `registry.username` - user with write access to your registry
3. `registry.password` - password
4. `registry.tags.prefix` - an image tag prefix. The prefix will be combined with the other tags in the `values.yaml` file
    to compute full image tags for images pushed by the platform into your registry. If you complete all the parts of this
    workshop, there will be several images pushed into your registry.

Examples of the values for various container registries are as follows:

| Registry                  | Example Server Name         | Example Tag Prefix                   |
|---------------------------|-----------------------------|--------------------------------------|
| Azure Container Registry  | foo.azurecr.io              | foo.azurecr.io                       |
| Dockerhub                 | https://index.docker.io/v1/ | jeffgbutler                          |
| Harbor                    | harbor.tanzuathome.net      | harbor.tanzuathome.net/tce [^1]      |
| Google Artifact Registry  | us-east1-docker.pkg.dev     | us-east1-docker.pkg.dev/foo/tce [^2] |
| Google Container Registry | gcr.io                      | gcr.io/foo                           |

[^1]: Harbor has the concept of projects in a repository. You must create the project in Harbor before pushing artifacts.
      In this case, we have a project named "tce"

[^2]: Google Artifact repository has two levels of naming - project-id, and repository. In this case we have project-id
      of "foo" (typically your GCP user name), and a repository named "tce". The repository must be created before pushing
      artifacts.

Once you have updated the `config/values.yaml` file, you are ready to proceed with the workshop.

[Next -&gt;](01-Install.md)
