# Bootstrap Phase 1: Flash and Initial Setup


NOTE: If the locale gets messed up and ansible will not run then try running the
following command: `sudo dpkg-reconfigure locales`

NOTE: For all manual tasks login over SSH as a normal user then copy and paste
the following bits of shell code into the shell, make sure to include the
parentheses. All commands assume you have already elevated privledges with
`sudo bash`.



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
ssh-keygen -t ed25519 -C "gate01 root"
)
```



## Create ansible user

NOTE: We need the public key from this process for the cloud-init process before
we can flash all of the nodes 

```
(
useradd ansible -m -s /bin/bash -U -G adm,sudo,ansible -c "Ansible User" 
usermod ansible -L
su ansible
ssh-keygen -t ed25519 -C "gate01 ansible"
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

Setup ansible.cfg and the inventory files



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



# Bootstrap Phase 2: Flash and Setup Users on Each Node



## Flash all nodes

NOTE: Make sure to add my public SSH key and the ansible SSH key to the
cloud-init process so that I can login in the first place and ansible can run
to finish configuration.

### Manual Tasks

NOTE: If you needed to manually create the ansible user on a node, this is how
you would do it.

```
(
useradd ansible -m -s /bin/bash -U -G adm,sudo,ansible -c "Ansible User" 
usermod ansible -L
su ansible
ssh-keygen -t ed25519 -C "gate01 ansible"
exit
)
```



## Setup my user, root user, and ansible user

NOTE: Login from my computer with my account

```
(
passwd
sudo bash
passwd
passwd ansible
)
```



# Bootstrap Phase 3: Finish Gate Setup



## Login to each node

NOTE: Login to each nodes with my account and the root account from the gate
server to accept the SSH key.



## Copy root's public SSH key to end nodes

NOTE: Need to first login to each node and set a password for the root user so
we can copy over the SSH keys that were just created.

```
(
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.20
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.21
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.22
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.23
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.24
)
```



## Update /etc/hosts on all nodes

```
- hosts: nodes
  gather_facts: false
  become: true
  tasks:
  - name: Add nodes to /etc/hosts
    ansible.builtin.blockinfile:
      path: /etc/hosts
      block: |
        10.128.64.19 gate01
        10.128.64.20 tpi01
        10.128.64.21 node01
        10.128.64.22 node02
        10.128.64.23 node03
        10.128.64.24 node04
      marker: "# {mark} ANSIBLE MANAGED BLOCK"
```



## Remove Cloud-Init from all nodes

```
- hosts: nodes
  gather_facts: false
  become: true
  tasks:
  - name: Remove and pruge cloud init packages
    ansible.builtin.apt:
      name: cloud-init
      state: absent
      purge: true

  - name: Remove /etc/cloud
    ansible.builtin.file:
      path: /etc/cloud
      state: absent

  - name: Remove /var/lib/cloud
    ansible.builtin.file:
      path: /var/lib/cloud
      state: absent
```



## Setup sudoers on all nodes

```
- hosts: nodes
  gather_facts: false
  become: true
  - name: Create a sudoers file for the username ansible
    ansible.builtin.copy:
      content: "ansible ALL = (ALL) NOPASSWD:ALL"
      dst: "/etc/sudoers.d/ansible"
      owner: root
      group: root
      mode: '0440'
  - name: Create a sudoers file for the username jordan
    ansible.builtin.copy:
      content: "jordan ALL = (ALL) NOPASSWD:ALL"
      dst: "/etc/sudoers.d/jordan"
      owner: root
      group: root
      mode: '0440'
  - name: Delete cloud-init sudo file used for initial bootstrap
    ansible.builtin.file:
      path: /etc/sudoers.d/90-cloud-init-users
      force: true
      state: absent
```



## Update system

```
- hosts: nodes
  gather_facts: false
  become: true
  tasks:
  - name: update and upgrade all nodes in cluster
    ansible.builtin.apt:
      update_cache: true
      upgrade: dist
    async: 600
    poll: 5

  - name: check reboot status
    ansible.builtin.stat:
      path: /var/run/reboot-required
      get_checksum: false
    register: reboot_required_file

  - name: reboot the server (if required).
    ansible.builtin.reboot:
    when: reboot_required_file.stat.exists == true

  - name: Remove dependencies that are no longer required.
    apt:
      autoremove: true
```








# OLD IDEAS - run ansible as root to create accounts, it is faster to just copy
  and paste commands on the shell, but here is a playbook that could do it.

### Ansible Tasks: Gate Only

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
      ssh_key_comment: "gate01 ansible"
      append: true
      state: present
```
