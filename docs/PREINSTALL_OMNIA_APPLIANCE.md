# Prerequisites

Ensure that the following prequisites are met before installing Omnia:
* On the management node, install Ansible and Git using the following commands:
	* `yum install epel-release -y`
	* `yum install ansible git -y`
__Note:__ Ansible must be installed using __yum__. If Ansible is installed using __pip3__, re-install it using the __yum__ command again.
* Ensure a stable Internet connection is available on management node and target nodes. 
* CentOS 7.9 2009 is installed on the management node.
* To provision the bare metal servers:
	* Go to http://isoredirect.centos.org/centos/7/isos/x86_64/ and download the **CentOS-7-x86_64-Minimal-2009** ISO file to the following directory on the management node: `omnia/appliance/roles/provision/files`.
	* Rename the downloaded ISO file to `CentOS-7-x86_64-Minimal-2009.iso`.
* For DHCP configuration, you can provide a mapping file named mapping_file.csv under __omnia/appliance/roles/provision/files__. The details provided in the CSV file must be in the format: MAC, Hostname, IP. For example, `xx:xx:4B:C4:xx:44,validation01,172.17.0.81` and  `xx:xx:4B:C5:xx:52,validation02,172.17.0.82` are valid entries.
__Note:__ Duplicate hostnames must not be provided in the mapping file and the hostname should not contain these characters: "_" and "."
* Connect one of the Ethernet cards on the management node to the HPC switch and one of the ethernet card connected to the global network.
* If SELinux is not disabled on the management node, disable it from `/etc/sysconfig/selinux` and restart the management node.
* The default mode of PXE is __UEFI__ and the BIOS Legacy Mode is not supported.
* The default boot order for the bare metal servers must be __PXE__.
* Configuration of __RAID__ is not part of Omnia. If bare metal servers have __RAID__ controller installed then it is mandatory to create **VIRTUAL DISK**.

## Assumptions

## Example system designs
Omnia can configure systems which use Ethernet- or Infiniband-based fabric to connect the compute servers.

![Example system configuration with Ethernet fabric](images/example-system-ethernet.png)

![Example system configuration with Infiniband fabric](images/example-system-infiniband.png)

## Network Setup
Omnia assumes that servers are already connected to the network and have access to the internet.
### Network Topology
Possible network configurations include:
* A flat topology where all nodes are connected to a switch which includes an uplink to the internet. This requires multiple externally-facing IP addresses
* A hierarchical topology where compute nodes are connected to a common switch, but the manager node contains a second network connection which is connected to the internet. All outbound/inbound traffic would be routed through the manager node. This requires setting up firewall rules for IP masquerade, see [here](https://www.server-world.info/en/note?os=CentOS_7&p=firewalld&f=2) for an example.
### IP and Hostname Assignment
The recommended setup is to assign IP addresses to individual servers. This can be done manually by logging onto each node, or via DHCP.
