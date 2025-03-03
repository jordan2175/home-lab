# Bootstrap Gate Server

NOTE: If the locate gets messed up and ansible will not run then try running the following command: `sudo dpkg-reconfigure locales`

## Setup root user

### Manual Tasks

Login over SSH as a normal user, then copy and paste the following bits of shell code into the shell, make sure to include the parentheses.

```
sudo bash
ssh-keygen -t ed25519 -C "gate01 root"
passwd
```



## Copy root's public SSH key to end nodes

### Manual Tasks

```
(
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.20
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.21
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.22
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.23
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.24
)
```

NOTE: Use the ansible playbook to copy the root key over to the nodes



## Update /etc/hosts

### Manual Tasks

```
(
echo "10.128.64.20 tpi01" >> /etc/hosts
echo "10.128.64.21 node01" >> /etc/hosts
echo "10.128.64.22 node02" >> /etc/hosts
echo "10.128.64.23 node03" >> /etc/hosts
echo "10.128.64.24 node04" >> /etc/hosts
)
```

### Ansible Tasks: Nodes Only

This task is ONLY for end nodes, not the gateway server given the chicken-and-egg problem with the entries in the inventory file.

```
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



## Create ansible user

### Manual Tasks

```
(
useradd ansible -m -s /bin/bash -U -G adm,sudo,ansible -c "Ansible User" 
usermod ansible -L
su ansible
ssh-keygen -t ed25519 -C "gate01 ansible"
exit
)
```

### Cloud-Init Tasks

For all nodes, this will be done via Cloud-Init. However, for the gate01 server we will need to manully create the SSH keypair and then add that public key to the OS image that gets deployed to all of the nodes.

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

### Ansible Tasks: Nodes Only

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
      append: true
      state: present
```


## Copy ansible's public SSH key to end nodes

### Manual Tasks

```
(
su ansible
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.20
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.21
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.22
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.23
ssh-copy-id -i .ssh/id_ed25519.pub 10.128.64.24
exit
)
```

### Cloud-Init Tasks

For the end nodes, this will be done via Cloud-Init



## Setup sudoers

### Manual Tasks

```
(
echo "ansible ALL = (ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible
chmod 0440 /etc/sudoers.d/ansible
echo "jordan ALL = (ALL) NOPASSWD:ALL" > /etc/sudoers.d/jordan
chmod 0440 /etc/sudoers.d/jordan
rm -f /etc/sudoers.d/90-cloud-init-users
)
```

### Ansible Tasks

```
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



## Install Ansible: Gate Only

### Manual Tasks

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



## Remove Cloud-Init

### Manual Tasks

```
(
apt-get purge cloud-init -y
rm -rf /etc/cloud
rm -rf /var/lib/cloud
)
```

### Ansible Tasks

```
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



## Update system

### Manual Tasks

```
(
apt-get dist-upgrade -y
reboot
)
```

### Ansible Tasks

```
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
