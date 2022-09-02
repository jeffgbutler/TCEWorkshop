# Install Tanzu Community Edition Pre-Requisites

This workshop requires Docker and several other tools. Using package manager (Homebrew or Chocolatey) makes it
easier to install some components, but is not strictly required. We will only write instructions based on
package managers.

You will need to have Internet access to run the exercises in the workshop.

You will also need write access to a container registry you can access from your workstation. Dockerhub certainly works, but
you might experience rate limiting if you are on an unpaid Docker account.

To complete the pre-requisites, follow the platform specific initial setup instructions below for your platform:
   [MacOS](#install-prerequisites---macos), [Windows](#install-prerequisites---windows), or [Linux](#install-prerequisites---linux)

## Install Prerequisites - MacOS

Install required tools:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Homebrew: https://brew.sh/
- JQ (JSON Formatter):
   ```shell
   brew install jq
   ```
- Carvel Tools:
   ```shell
   brew tap vmware-tanzu/carvel
   brew install kapp kctrl ytt
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
- Kubectl Krew Plugin Manager (Optional, but helpful): https://krew.sigs.k8s.io/docs/user-guide/setup/install/
- Kubectl tree plugin (Optional, but helpful. Requires Krew Plugin Manager):
   ```shell
   kubectl krew install tree
   ```

[Next (Install TCE) -&gt;](LocalTCEInstall.md)

## Install Prerequisites - Windows

Install required tools:

- Git: https://git-scm.com/downloads
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Chocolatey: https://chocolatey.org/install
- JQ (JSON Formatter):
   ```shell
   choco install jq
   ```
- Carvel Tools:
   ```shell
   choco install kapp ytt
   ```
- Knative CLI: Go to https://github.com/knative/client/releases, download latest Windows binary, rename it to "kn", than add it to the path
- Kpack CLI: Go to https://github.com/vmware-tanzu/kpack-cli/releases, download latest Windows binary, rename it to "kp", add it to the path
- Kubectl Krew Plugin Manager (Optional, but helpful): https://krew.sigs.k8s.io/docs/user-guide/setup/install/
- Kubectl tree plugin (Optional, but helpful. Requires Krew Plugin Manager):
   ```shell
   kubectl krew install tree
   ```

[Next (Install TCE) -&gt;](LocalTCEInstall.md)

## Install Prerequisites - Linux

Install required tools:

- Git: https://git-scm.com/downloads
- Docker Engine: https://docs.docker.com/engine/install/
- Homebrew: https://brew.sh/
- JQ (JSON Formatter):
   ```shell
   brew install jq
   ```
- Carvel Tools:
   ```shell
   brew tap vmware-tanzu/carvel
   brew install kapp kctrl ytt
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
- Kubectl Krew Plugin Manager (Optional, but helpful): https://krew.sigs.k8s.io/docs/user-guide/setup/install/
- Kubectl tree plugin (Optional, but helpful. Requires Krew Plugin Manager):
   ```shell
   kubectl krew install tree
   ```

[Next (Install TCE) -&gt;](LocalTCEInstall.md)
