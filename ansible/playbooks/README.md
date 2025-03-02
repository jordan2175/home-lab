# Playbooks

## Running Playbooks

First run the bootstrap playbook
 - ansible-playbook --ask-become-pass playbooks/bootstrap.yml 


ansible-playbook playbooks/update.yml 
ansible-playbook playbooks/remove-cloud-init.yml 
ansible-playbook playbooks/poweroff-cluster.yml 
