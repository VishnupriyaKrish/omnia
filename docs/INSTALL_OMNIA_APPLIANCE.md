# Install the Omnia appliance

## Prerequisties
Ensure that all the prerequisites listed in the [PREINSTALL_OMNIA_APPLIANCE](PREINSTALL_OMNIA_APPLIANCE.md) file are met before installing Omnia appliance

__Note:__ Changing the manager node after the installation of Omnia is not supported by Omnia. If you want to change the manager node, you must redeploy the entire cluster.  
__Note:__ The user should have root privileges to perform installations and configurations.

## Steps to install the Omnia appliance
__Note:__ If there are errors when any of the following Ansible playbook commands are run, re-run the commands again.
1. On the management node, change the working directory to the directory where you want to clone the Omnia Git repository.
2. Clone the Omnia repository.
``` 
$ git clone https://github.com/dellhpc/omnia.git 
```
__Note:__ After the Omnia repository is cloned, a folder named __omnia__ is created. It is recommended that you do not rename this folder.

3. Change the directory to `omnia/appliance`
4. To provide passwords for Cobbler and AWX, edit the `appliance_config.yml` file.
* To provide a mapping file for DHCP configuration, go to **appliance_config.yml** file and set the variable named **mapping_file_exits** as __true__, else set it to __false__.

Omnia considers the following usernames as default:  
* `cobbler` for Cobbler Server
* `admin` for AWX
* `slurm` for MariaDB

**Note**: 
* Minimum length of the password must be at least eight characters and a maximum of 30 characters.
* Do not use these characters while entering a password: -, \\, "", and \'

5. Using the `appliance_config.yml` file, you can change the NIC for the DHCP server under **hpc_nic** and the NIC used to connect to the Internet under **public_nic**. Default values of **hpc_nic** and **public_nic** are set to em1 and em2 respectively.
6. The valid DHCP range for HPC cluster is set in two variables named __Dhcp_start_ip_range__ and __Dhcp_end_ip_range__ present in the `appliance_config.yml` file.
7. To provide passwords for mariaDB Database for Slurm accounting and Kubernetes CNI, edit the `omnia_config.yml` file.

__Note:__ Supported Kubernetes CNI : calico and flannel. The default CNI is calico.

To view the set passwords of `appliance_config.yml`, run the following command under omnia->appliance:
```
ansible-vault view appliance_config.yml --vault-password-file .vault_key
```

To view the set passwords of `omnia_config.yml`, run the following command:
```
ansible-vault view omnia_config.yml --vault-password-file .omnia_vault_key
```

8. To install Omnia, run the following command:
```
ansible-playbook appliance.yml -e "ansible_python_interpreter=/usr/bin/python2"
```
   
Omnia creates a log file which is available at: `/var/log/omnia.log`.

**Provision operating system on the target nodes**  
Omnia role used: *provision*  
Ports used by __Cobbler__:  
* __TCP__ ports: 80,443,69
* __UDP__ ports: 69,4011

To create the Cobbler image, Omnia configures the following:
* Firewall settings.
* The kickstart file of Cobbler will enable the UEFI PXE boot.

To access the Cobbler dashboard, enter `https://<IP>/cobbler_web` where `<IP>` is the Global IP address of the management node. For example, enter
`https://100.98.24.225/cobbler_web` to access the Cobbler dashboard.

__Note__: After the Cobbler Server provisions the operating system on the nodes, IP addresses and host names are assigned by the DHCP service.  
* If a mapping file is not provided, the hostname to the server is provided based on the following format: **computexxx-xxx** where "xxx-xxx" is the last two octets of Host IP address. For example, if the Host IP address is 172.17.0.11 then he assigned hostname by Omnia is compute0-11.  
* If a mapping file is provided, the hostnames follow the format provided in the mapping file.

**Install and configure Ansible AWX**  
Omnia role used: *web_ui*  
Port used by __AWX__ is __8081__.  
AWX repository is cloned from the GitHub path: https://github.com/ansible/awx.git 

Omnia performs the following configuration on AWX:
* The default organization name is set to **Dell EMC**.
* The default project name is set to **omnia**.
* Credential: omnia_credential
* Inventory: omnia_inventory with compute and manager groups
* Template: DeployOmnia and Dynamic Inventory
* Schedules: DynamicInventorySchedule which is scheduled for every 10 mins

To access the AWX dashboard, enter `http://<IP>:8081` where **\<IP>** is the Global IP address of the management node. For example, enter `http://100.98.24.225:8081` to access the AWX dashboard.

**Note**: The AWX configurations are automatically performed Omnia and Dell Technologies recommends that you do not change the default configurations provided by Omnia as the functionality may be impacted.

__Note__: Although AWX UI is accessible, hosts will be shown only after few nodes have been provisioned by Cobbler. It takes approximately 10 to 15 minutes to display the host details after the provisioning by Cobbler. If a server is provisioned but you are unable to view the host details on the AWX UI, then you can run **provision_report.yml** playbook from __omnia__ -> __appliance__ ->__tools__ folder to view the hosts which are reachable.

## Install Kubernetes and Slurm using AWX UI
Kubernetes and Slurm are installed by deploying the **DeployOmnia** template on the AWX dashboard.

1. On the AWX dashboard, under __RESOURCES__ __->__ __Inventories__, select __Groups__.
2. Select either __compute__ or __manager__ group.
3. Select the __Hosts__ tab.
4. To add the hosts provisioned by Cobbler, select __Add__ __->__ __Add__ __existing__ __host__, and then select the hosts from the list and click __Save__.
5. To deploy Omnia, under __RESOURCES__ -> __Templates__, select __DeployOmnia__ and click __LAUNCH__.
6. By default, no skip tags are selected and both Kubernetes and Slurm will be deployed. To install only Kubernetes, enter `slurm` and select **Create "slurm"**. Similarly, to install only Slurm, select and add `kubernetes` skip tag. 

__Note:__
*	If you would like to skip the NFS client setup, enter `nfs_client` in the skip tag section to skip the **k8s_nfs_client_setup** role of Kubernetes.

7. Click **Next**.
8. Review the details in the **Preview** window, and click **Launch** to run the DeployOmnia template. 

To establish the passwordless communication between compute nodes and manager node:
1. In AWX UI, under __RESOURCES__ -> __Templates__, select __DeployOmnia__ template.
2. From __Playbook dropdown__ menu, select __appliance/tools/passwordless_ssh.yml__ and launch the template.

__Note:__ If you want to install __JupyterHub__ and __Kubeflow__ playbooks, you have to first install the __JupyterHub__ playbook and then install the __Kubeflow__ playbook.

__Note:__ To install __JupyterHub__ and __Kubeflow__ playbooks:
*	From __AWX UI__, under __RESOURCES__ -> __Templates__, select __DeployOmnia__ template.
*	From __Playbook dropdown__ menu, select __platforms/jupyterhub.yml__ option and launch the template to install JupyterHub playbook.
*	From __Playbook dropdown__ menu, select __platforms/kubeflow.yml__ option and launch the template to install Kubeflow playbook.


The DeployOmnia template may not run successfully if:
- The Manager group contains more than one host.
- The Compute group does not contain a host. Ensure that the Compute group must be assigned with a minimum of one host node.
- Under Skip Tags, when both kubernetes and slurm tags are selected.

After **DeployOmnia** template is run from the AWX UI, the **omnia.yml** file installs Kubernetes and Slurm, or either Kubernetes or slurm, as per the selection in the template on the management node. Additionally, appropriate roles are assigned to the compute and manager groups.

The following __kubernetes__ roles are provided by Omnia when __omnia.yml__ file is run:
- __common__ role:
	- Install common packages on manager and compute nodes
	- Docker is installed
	- Deploy time ntp/chrony
	- Install Nvidia drivers and software components
- **k8s_common** role: 
	- Required Kubernetes packages are installed
	- Starts the docker and Kubernetes services.
- **k8s_manager** role: 
	- __helm__ package for Kubernetes is installed.
- **k8s_firewalld** role: This role is used to enable the required ports to be used by Kubernetes. 
	- For __head-node-ports__: 6443, 2379-2380,10251,10252
	- For __compute-node-ports__: 10250,30000-32767
	- For __calico-udp-ports__: 4789
	- For __calico-tcp-ports__: 5473,179
	- For __flanel-udp-ports__: 8285,8472
- **k8s_nfs_server_setup** role: 
	- A __nfs-share__ directory, `/home/k8snfs`, is created. Using this directory, compute nodes share the common files.
- **k8s_nfs_client_setup** role
- **k8s_start_manager** role: 
	- Runs the __/bin/kubeadm init__ command to initialize the Kubernetes services on manager node.
	- Initialize the Kubernetes services in the manager node and create service account for Kubernetes Dashboard
- **k8s_start_workers** role: 
	- The compute nodes are initialized and joined to the Kubernetes cluster with the manager node. 
- **k8s_start_services** role
	- Kubernetes services are deployed such as Kubernetes Dashboard, Prometheus, MetalLB and NFS client provisioner

__Note:__ After Kubernetes is installed and configured, few Kubernetes and calico/flannel related ports are opened in the manager and compute nodes. This is required for Kubernetes Pod-to-Pod and Pod-to-Service communications. Calico/flannel provides a full networking stack for Kubernetes pods.

The following __Slurm__ roles are provided by Omnia when __omnia.yml__ file is run:
- **slurm_common** role:
	- Installs the common packages on manager node and compute node.
- **slurm_manager** role:
	- Installs the packages only related to manager node
	- This role also enables the required ports to be used by Slurm.  
	    **tcp_ports**: 6817,6818,6819  
		**udp_ports**: 6817,6818,6819
	- Creating and updating the Slurm configuration files based on the manager node requirements.
- **slurm_workers** role:
	- Installs the Slurm packages into all compute nodes as per the compute node requirements.
- **slurm_start_services** role: 
	- Starting the Slurm services so that compute node communicates with manager node.
- **slurm_exporter** role: 
	- Slurm exporter is a package for exporting metrics collected from Slurm resource scheduling system to prometheus.
	- Slurm exporter is installed on the host like Slurm, and Slurm exporter will be successfully installed only if Slurm is installed.

## Adding a new compute node to the Cluster

If a new node is provisioned through Cobbler, the node address is automatically displayed on the AWX dashboard. The node is not assigned to any group. You can add the node to the compute group and run `omnia.yml` to add the new node to the cluster and update the configurations in the manager node.
