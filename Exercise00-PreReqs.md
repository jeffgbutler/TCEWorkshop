# Pre-Requisites

Install:

- Docker Desktop: https://docs.docker.com/desktop/windows/install/
- Chocolatey: https://chocolatey.org/install
- Knative CLI: https://github.com/knative/client/releases (download latest binary, rename to "kn", add to path)
- Kpack CLI: https://github.com/vmware-tanzu/kpack-cli/releases (download latest binary, rename to "kp", add to path)

You must also have write access to a container registry. There are several places in the workshop where you will need
to enter credentials and configuration information for your registry. Examples of the various values are as follows:

- Dockerhub:
  - kpack.kp_default_repository in app toolkit configuration: `your-name/kpack`
  - docker_server in the Kpack secret: `https://index.docker.io/v1/`
  - spec.tag in the Kpack Builder configuration: `your-name/builder`
  - spec.tag in the Kpack Image configuration: `your-name/spring-pet-clinic`
- Harbor (create a Harbor project for tce - I created a project called "tce"):
  - kpack.kp_default_repository in app toolkit configuration: `harbor.tanzuathome.net/tce/kpack`
  - docker_server in the Kpack secret: `harbor.tanzuathome.net`
  - spec.tag in the Kpack Builder configuration: `harbor.tanzuathome.net/tce/builder`
  - spec.tag in the Kpack Image configuration: `harbor.tanzuathome.net/tce/spring-pet-clinic`
- GCR:
  - kpack.kp_default_repository in app toolkit configuration: `gcr.io/your-project/kpack`
  - docker_server in the Kpack secret: `gcr.io`
  - spec.tag in the Kpack Builder configuration: `gcr.io/your-project/builder`
  - spec.tag in the Kpack Image configuration: `gcr.io/your-project/spring-pet-clinic`
