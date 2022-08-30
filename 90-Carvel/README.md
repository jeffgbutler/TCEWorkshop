# Carvel Tools

Understanding the Carvel tools is fundamental to success with Tanzu. Most of the Carvel tools are command line tools that do
relatively simple things. Used together, they form a very capable toolchain for working with Kubernetes.

Some of the Carvel tools are installed into Kubernetes clusters as controllers. In fact, the definition of a "Tanzu" cluster
is just a plain old Kubernetes cluster with two specific Carvel tools installed: the secretgen controller and the Kapp controller.

For some historical context, the Carvel tools were previously called simple "Kubernetes Tools" which was shortened to "k14s" in
true Kubernetes fashion. "k14s" was rebranded as "Carvel" when the project was sponsored by VMware.

Carvel is open source, sponsored by VMware, and you can read all about it here: https://carvel.dev/

The following pages have overviews and examples of several Carvel tools. You can view them in any order, but you should
have a good understanding of YTT, Kbld, and Kapp before reading about the Kapp controller.

- [Secretgen Controller](secretgen-controller/README.md)
- [YTT](ytt/README.md)
- [Kbld](kbld/README.md)
- [Kapp](kapp/README.md)
- [Kapp Controller](kapp-controller/README.md)
