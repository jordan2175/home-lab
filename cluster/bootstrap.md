# Bootstrap Gate Server


NOTE: If the locale gets messed up and ansible will not run then try running the
following command: `sudo dpkg-reconfigure locales`

NOTE: For all manual tasks login over SSH as a normal user then copy and paste
the following bits of shell code into the shell, make sure to include the
parentheses. All commands assume you have already elevated privledges with
`sudo bash`.

```
#cloud-config
users:
  - name: root
    lock_passwd: false
    hashed_passwd: <output from mkpasswd --method=SHA-512 --rounds=4096>
    ssh_authorized_keys:
      - ssh-rsa <key>
```

## Flash gate server

NOTE: Make sure to add my public SSH key to the cloud-init process so that I can
login in the first place



## Setup my user and the root user

NOTE: Login from my computer with my account

```
(
passwd
sudo bash
passwd
ssh-keygen -t ed25519 -C "Gate01 Root Default"
)
```



## Create ansible system user

NOTE: We need the public key from this process for the cloud-init process before
we can flash all of the nodes 

```
(
groupadd ansible -r
useradd ansible -r -m -s /bin/bash -g ansible -G adm,sudo,ansible -c "Ansible User" 
usermod ansible -L
su ansible
ssh-keygen -t ed25519 -C "Gate01 Ansible Default"
exit
)
```



## Setup sudoers

```
(
echo "ansible ALL = (ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
chmod 0440 /etc/sudoers.d/ansible
echo "jordan ALL = (ALL) NOPASSWD:ALL" > /etc/sudoers.d/jordan
chmod 0440 /etc/sudoers.d/jordan
rm -f /etc/sudoers.d/90-cloud-init-users
)
```



## Remove Cloud-Init

```
(
apt-get purge cloud-init -y
rm -rf /etc/cloud
rm -rf /var/lib/cloud
rm -f /boot/firmware/user-data
rm -f /boot/firmware/meta-data
rm -f /boot/firmware/network-config
)
```



## Install ansible

```
(
apt-get update
mkdir -p /local/ansible/playbooks
touch /local/ansible/ansible.cfg
touch /local/ansible/inventory 
chown -R ansible:ansible /local/ansible
chmod -R 0770 /local/ansible
apt-get install ansible
)
```

## Setup Ansible

Download ansible configuartion and playbooks from Github


## Update /etc/hosts

```
(
echo "10.128.64.20 tpi01" >> /etc/hosts
echo "10.128.64.21 node01" >> /etc/hosts
echo "10.128.64.22 node02" >> /etc/hosts
echo "10.128.64.23 node03" >> /etc/hosts
echo "10.128.64.24 node04" >> /etc/hosts
)
```



## Update system

```
(
apt-get update && apt-get dist-upgrade -y
reboot
)
```



# Bootstrap Nodes Phase 1

 - Flash all nodes
   - NOTE: Make sure to add the root public SSH key to the cloud-init process
 - Assign IP addresses in DHCP
   - Do this based on the MAC Address
   - May need to login and reboot the node to get new address
   - Or run the ansible script against the temporary IP address and once it reboots it will get the correct one
 - Login to each node with root to accept SSH key
 - Run Ansible playbooks
   - ansible-playbook playbooks/bootstrap-nodes.yml
   - ansible-playbook playbooks/update.yml







# OLD IDEAS - run ansible as root to create accounts, it is faster to just copy
  and paste commands on the shell, but here is a playbook that could do it.





## Ansible Tasks: Gate Only

```
  - name: Create an ansible group
    ansible.builtin.group:
      name: "ansible"
      system: false
      state: present

  - name: Create an ansible user
    ansible.builtin.user:
      name: "ansible"
      password: '!'
      system: false
      comment: "Ansible User"
      group: "ansible"
      groups: adm,sudo,ansible 
      create_home: true
      shell: /bin/bash 
      generate_ssh_key: true 
      ssh_key_type: ed25519
      ssh_key_comment: "Gate01 Ansible Default"
      append: true
      state: present
```

Copyright 2025 Bret Jordan, All rights reserved.