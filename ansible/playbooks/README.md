# Playbooks

## Running Playbooks

NOTES:

 1. If the system has been reimaged you need to clear out the local known_hosts entry
 2. You need to run ssha to start the SSH agent
 3. You need to login to the machine to accept SSH key
   - As the IP address and the short name of node01
 4. Old way to create a password
   - mkpasswd --method=SHA-512 --rounds=500000
 5. Run bootstrap playbooks


 - Bootstrap playbooks
   - ansible-playbook playbooks/bootstrap-nodes.yml
   - ansible-playbook playbooks/update-nodes.yml
