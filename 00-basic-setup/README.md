# Prepare for the Workshop

The workshop requires you to install several tools on your workstation. There are detailed instructions
for installation on the following pages. Alternatively, our friends at [Guard Dog Solutions](https://guarddog.ai/)
have created a VM image (OVA) with everything pre-installed. You can run their image in vSphere, VMware Workstation,
or VMware Fusion.

After we create our first cluster, we will install several componants in the cluster that are required
for the later exercises. There are several configuration values you will need to have available for that
step. The following sections will help you determine proper values for your installation.

## Container Registry Access

This workshop requires that you have write access to a container registry. All users must plan their
access to a container registry.

## Domain and DNS Information

You must plan for a domain name and DNS. You do not need a public domain name for this workshop,
but you can use one in certain cases if you so desire. Read through the section below that relates to your
installation scenario and **make note of the domain name you plan to use**.

### Domain and DNS for Local Workstation Installation

In this case, you should use the domain name `127-0-0-1.nip.io`. This will direct all browser traffic
directly to your local workstation.

### Domain and DNS for the Guard Dog OVA

When you install the Guard Dog OVA, you will configure networking. This can be either DHCP or a static IP
configuration. In either case, the VM will have an IP address. You have two options for a domain name in this
case:

1. You can use an `nip.io` name like above prefixed with the IP address of the VM. For example,
   `192-168-127-39.nip.io` - notice that the IP address uses dashes between the octets.
1. If you have a domain name, you can add a wildcard DNS "A" record for a domain that answers with the
   IP address of the VM. For example:

   `*.tce.mydomain.com -> 192.168.127.39`

   In this case you should use the domain name `tce.mydomain.com` in the configuration file for TCE

### Domain and DNS for a TCE Management and Workload Cluster Installation

This is a more complex installation of TCE that is appropriate for long lived production clusters.
In this case you should configure a LoadBalancer for the cluster and have a domain name you control
where you can add a wildcard DNS entry to the IP address of the ingress controller in the cluster.
This is an advanced scenario and we will not cover it in this workshop.

## Next Steps

1. [Plan your Registry Access](ContainerRegistry.md)
1. Install TCE:
   - [Install TCE and Tools on a Local Workstation](LocalToolInstall.md)
   - [Install the Guard Dog OVA](InstallGuardDogOVA.md)
