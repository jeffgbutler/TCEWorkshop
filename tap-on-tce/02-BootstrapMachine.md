# Bootstrap Machine

Create a VM to bootstrap TAP.

## Create and Configure a Bootstrap VM

Create a VM for working with TCE. The VM I created has the following characteristics:

- Ubuntu Desktop 20.04 (LTS)
- 8 vCPU
- 16 GB RAM
- 160 GB Storage

I did a minimal install initially. We will add a few items...

## Install OpenSSH Server

SSH can be usefull in many cases, so let's install it:

```shell
sudo apt-get update

sudo apt-get install openssh-server

sudo ufw allow ssh
```

## Install Docker

https://docs.docker.com/engine/install/ubuntu/

## Setup Kubectl

ssh jeff@192.168.128.38

export KUBECONFIG=/home/jeff/tap-install/tap-cluster-kubeconfig.yaml

## Steps

Install brew
brew install kubectl

brew tap vmware-tanzu/carvel
brew install imgpkg kapp ytt kbld vendir

download tanzu framwork package and follow the steps to install


## Relocate Images

docker login harbor.tanzuathome.net

docker login registry.tanzu.vmware.com  jgbutler@pivotal.io/U.N24ui]!iw5dNG[(a

export INSTALL_REGISTRY_USERNAME=admin
export INSTALL_REGISTRY_PASSWORD=Harbor12345
export INSTALL_REGISTRY_HOSTNAME=harbor.tanzuathome.net
export TAP_VERSION=1.2.0

imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:1.2.0 --to-repo harbor.tanzuathome.net/tap12/tap-packages

kubectl create ns tap-install

tanzu secret registry add tap-registry \
  --username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
  --server ${INSTALL_REGISTRY_HOSTNAME} \
  --export-to-all-namespaces --yes --namespace tap-install

tanzu package repository add tanzu-tap-repository \
  --url harbor.tanzuathome.net/tap12/tap-packages:1.2.0 \
  --namespace tap-install
