# Install and Configure the Guard Dog OVA

Our friends at Guard Dog solutions have created an OVA with the prerequisite tools installed. This makes it very easy
to start the workshop. I'll cover the steps I took to install and use it.

## Install the OVA

1. Download the OVA from hare: https://github.com/guarddog-dev/VMware_Photon_OVA/tree/main/output-vmware-iso
2. Import the OVA into vSphere and assign a static IP address. Also, enable SSH and take the option to learn TCE
   on your own.

Remember the following items from the template parameters:

1. The IP address - this will be important in the TCE configuration
2. The SSH root password

Once the OVA is installed and started, SSH into the VM.

## Clone the Exercise Repository

Once you've installed the OVA and started the VM, log in to the VM with either SSH or a VM console.
Then clone the exercise repository so you will have local access to the configuration and exercise files:

```shell
git clone https://github.com/jeffgbutler/TCEWorkshop.git
```


[Next (Create a Cluster) -&gt;](../01-creating-clusters/README.md)
