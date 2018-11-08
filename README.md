# ABOUT THIS REPO
This repository is a prescriptive OpenShift _inventory_ file I use for my lab environment and some notes on the installation process. The inventory file is verbose to allow for quick adjustment during tests. 

To customize the inventory file to your environment you may reference the official OpenShift documentation:

https://docs.openshift.com/container-platform/3.11/install/configuring_inventory_file.html


# LAB CONCEPTUAL DIAGRAM

![Diagram](images/openshift_ocs_converged_mode_small.png)

# HOST PREPARATION

***NOTE:*** This is only a summary. For detailed information visit:
https://docs.openshift.com/container-platform/latest/install/host_preparation.html

1. Ensure _bastion_ node has ssh access to all nodes
```
$ ssh-keygen

$ for host in master1.example.com \
master2.example.com \
master3.example.com \
infranode1.example.com \
infranode2.example.com \
node1.example.com \
node4.example.com \
node3.example.com \
ocs-node1.example.com \
ocs-node2.example.com \
ocs-node3.example.com; \
do ssh-copy-id -i ~/.ssh/id_rsa.pub $host; \
done
```

2. Register the host

```
# register each host with RHSM
subscription-manager register --username=<user_name> --password=<password>

# pull subscriptions
subscription-manager refresh

# identify the available OpenShift subscriptions
subscription-manager list --available --matches '*OpenShift*'

# assign a subscription to the node
subscription-manager attach --pool=<pool_id>
```

3. Assign subscription to the host

```
# Disable all RHSM repositories
subscription-manager repos --disable="*"

# Enable only repositories required by OpenShift
subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.11-rpms" \
    --enable="rhel-7-server-ansible-2.6-rpms"
```

# INSTALLING OPENSHIFT
1. Install required installation tools at _bastion_ node
```
$ yum -y install atomic-openshift-clients openshift-ansible
```

2. Copy the OpenShift Ansible _inventory_ file to the _bastion_ node. Either using the _bastion_ Ansible host file or maintaining a local _inventory_file_
```
/etc/ansible/hosts

_or_

~/inventory_file

```

- Inventory configuration details: https://docs.openshift.com/container-platform/3.11/install/configuring_inventory_file.html


3. Validate _bastion_ can reach all the nodes 

```
ansible all -i inventory_file -m ping
```

4. From _bastion_ node run the installation of prerequisites (base packages & Docker) on all OpenShift nodes

 ```
ansible-playbook -f 20 -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml
```

***NOTE***: In case of error indicating playbooks are not found, install the corresponding package in the _bastion_ node executing: ```yum install openshift-ansible```

***NOTE***: The ``-i <inventory_file>`` flag is not needed if using the default inventory path ``/etc/ansible/hosts``

5. Once the previous step is completed ***WITHOUT*** any error, you may proceed with the deployment.

    Deploy OpenShift using the ``ansilbe-playbook``

```
ansible-playbook -f 20 -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml
```

NOTE: If the initial installation fails you need to uninstall OpenShift and start the installation from the _prerequisites_ playbook.

# UNINSTALLING OPENSHIFT

```
# Uninstall OpenShift
ansible-playbook -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/adhoc/uninstall.yml

# Remove left over config and certs from master and nodes
ansible -i inventory_file nodes -a "rm -rf /etc/origin"
```

# TESTING THE OPENSHIFT INSTALLATION
## Setting up _bastion_ node for ``system:admin`` access
To use the _bastion_ node for cluster administration, copy the .kube directory from _master1_ to your _bastion_

```
ansible -i inventory_file masters[0] -b -m fetch -a "src=/root/.kube/config dest=/root/.kube/config flat=yes"
```

Once the .kube directory is in your _bastion_ node, validate you are ``system:admin``

```
oc whoami
```

## Verifying the environment
The following commands you be successful

```
oc get nodes --show-labels

oc get nodes -o wide

oc get pod --all-namespaces -o wide

oc login -u <ocp_user>
```

Login into the web console by accessing the name indicated in the ``openshift_master_cluster_hostname`` variable of the _inventory_file_ (i.e. https://openshift-console.example.com)

Try some demo applications to test the environment.

I have some simple demo activities here:
https://github.com/williamcaban/podcool-docs