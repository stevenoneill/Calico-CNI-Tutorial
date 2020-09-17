# Introduction
The inspiration for this repo came from [this Medium post from Vikram Fugro](https://medium.com/@vikram.fugro/project-calico-the-cni-way-659d057566ce) ([@cotigao](https://github.com/cotigao) on Github) on the subject of investigating Project Calico using Ubuntu servers.  It is highly recommended that this be read first before proceeding.  This repo merely provides a means using Vagrant and Ansible to quickly get to the point where one can learn about Calico, CNI and the interaction between them.  It is not meant to be a deep dive into Vagrant or Ansible but some pointers may be highlighted if they're applicable to the flow of the project.

# Required Tools 
The tools in the below table are what are required to be installed on a laptop/computer in order to be able to use this repo.  Note, future editions of these tools may work just fine, these are the versions that were tested.

| Tool      | Tested Version | Install Instructions  |
| --------- |:-------:|:---------------------:|
| Vagrant   | 2.2.10  | [link](https://www.vagrantup.com/docs/installation) |
| Ansible   | 2.9     | [link](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) |
| Virtualbox| 6.1.12  | [link](https://www.virtualbox.org/manual/ch02.html) |
| Sufficient resource to run three VMs | N/A | N/A |
| An Internet connetion | N/A | N/A |

TODO:Test with configurations to limit the amount of RAM Vagrant provides to each system to be 256MB.

For those unfamiliar with Vagrant, it is a product from Hashhicorp which allows for VMs or containers to be spun up and subsequently provisioned for the purposes of creating a lightweight and portable development environment.  The reason for listing Virtualbox above is that Vagrant by default uses Vitualbox as its hypervisor for spinning up resources.  

# Topology
TODO: Create a diagram of the VMs and docker containers that get launched

# Setup
Once all of the above tools are installed on the target test system, clone the repo to the target test system:

```bash
$ git clone git@github.com:TheFutonEng/Calico-CNI-Tutorial.git
```

After that, run the following to deploy the three VMs:

```bash
$ cd Calico-CNI-Tutorial
$ vagrant up
```

This command may take a few minutes to run but it does the following:

* Builds the three VMs defined in the Vagrantfile and detailed in the previous topology section  
* Stands up etcd docker container on the etcd node 
* Stands up calico docker containers on the calico nodes which point to the etcd node for their datastore
* Installs calicoctl on all three VMs  
* Downloads and installs the calico and calico-ipam CNI reference plugins on the calico nodes
* Downloads and installs Golang on the calico nodes (role to perform this copied from @jlund [here](https://github.com/jlund/ansible-go))
* Downloads and installs the CNI network plugins on the calico nodes
* Downloads and install the cnitool on the calico nodes

# Setup Verification

Run the this section to confirm that the setup has worked.

```
$ vagrant ssh etcd-node
$ sudo calicoctl get nodes
```

The first command uses an SSH key generated by vagrant to log into the etcd-node.  The second command checks what nodes have registered with etcd, the output should be the two calico nodes:

```
NAME      
calico1   
calico2 
```

Now access one of the calico nodes and confirm that it has discovered the other node:

```
$ vagrant ssh calico1
$ sudo calicoctl node status
```

The output from the command should be the data regarding the established BGP peer with calico2:

```
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+----------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+--------------+-------------------+-------+----------+-------------+
| 192.168.4.6  | node-to-node mesh | up    | 23:26:50 | Established |
+--------------+-------------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```

# Calico and the CNI - Exploration

From this point, feel free to explore however you wish.  This repo will continue to step through some changes that can be done once the setup is up and running successfully.  Automation has been written to perform each of the steps below and the instructions on using it will be supplied first.  Manual instructions for each step will be supplied thereafter if you would prefer to step through each change on your own for learning purposes.

## IPPool

An IP Pool is a Calico resource type which represents an IP range from which Calico can assign individual addresses to endpoints.  In order to create a pool, run the following command from your localhost machine:

```bash
$ ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory ippool.yml 
```

The above command will deploy the ippool defined in ```Calico-CNI-Tutorial/roles/ippool/templates/ippool.yml.j2```.  