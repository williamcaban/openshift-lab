help:
	@echo "Usage: \n\tmake [ping | pre | install | uninstall | clean | jumphost | <ACTIONS>]"
	@echo "\t>>>>NOTE<<<< This script assumes 'inventory_file' for openshift-ansible in local dir" 
	@echo "\n\tACTIONS: upgradenodes, upgradeocp, installocs, uninstallocs, rebootocs, reboootall, cleanimages, cleanleftovers\n" 

ping:
	ansible all -i inventory_file -m ping

pre:
	bash -c "time ansible-playbook -f 5 -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml"

install:
	bash -c "time ansible-playbook -v -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml"

uninstall:
	ansible-playbook -f 10 -i inventory_file -e "openshift_storage_glusterfs_wipe=true" /usr/share/ansible/openshift-ansible/playbooks/adhoc/uninstall.yml
	ansible -f 10 -i inventory_file glusterfs -a "wipefs -a -f /dev/sdb"
	ansible nodes -i inventory_file -m file -a "dest=/etc/origin state=absent"

upgradenodes:
	@echo "Upgrading OS packages in all nodes"
	bash -c "time ansible-playbook -v -f 5 -i inventory_file ./upgrade-cluster-os.yaml"

upgradeocp:
	bash -c "time ansible-playbook -v -f 5 -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/upgrades/v3_11/upgrade.yml"

installocs:
	ansible-playbook -f 5 -i inventory_file /usr/share/ansible/openshift-ansible/playbooks/openshift-glusterfs/config.yml

uninstallocs:
	ansible-playbook -f 5 -i inventory_file -e "openshift_storage_glusterfs_wipe=true" /usr/share/ansible/openshift-ansible/playbooks/openshift-glusterfs/uninstall.yml

rebootocs:
	@echo "Rebooting OCS nodes"
	ansible -f 5 -i inventory_file glusterfs -a "reboot"

rebootall:
	@echo "Rebooting ALL nodes"
	ansible -f 5 -i inventory_file all -a "reboot"

shutdownall:
	@echo "Shuttting down lab... the wrong way"
	ansible -i inventory_file all -a "shutdown -h now"

clean:
	@echo "Removing ansible log file from bastion"
	rm -f /var/log/ansible.log*

cleanimages:
	ansible -f 5 -i inventory_file nodes -a "docker system prune -a -f"

cleanleftovers:
	#ansible nodes -i inventory_file -a "rm -fr /etc/cni/"
	ansible nodes -i inventory_file -m file -a "dest=/etc/cni state=absent"
	ansible nodes -i inventory_file -m copy -a "src=/root/ocp-deployment/iptables.plain dest=/etc/sysconfig/iptables"

jumphost:
	ansible -i inventory_file masters[0] -b -m fetch -a "src=/root/.kube/config dest=/root/.kube/config flat=yes"
	oc adm policy add-cluster-role-to-user admin ocpadmin

