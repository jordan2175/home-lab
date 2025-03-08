# Setup Ansible


## Configure

apt-get install ansible -y
setup /local/ansible/ansible.cfg
cd /local/ansible

ansible all -m ping
ansible all --list-hosts
ansible all -m gather_facts --limit node01
ansible all -m apt -a update_cache=true --become --ask-become-pass
ansible all -m reboot -b
ansible all -m apt -a "name=somepackage state=latest" --become --ask-become-pass


## Notes

become = means run with elevated privledges
  sudo is the default solutions for become
  so when it asks for a password, it is the sudo password it is asking for

```
 - hosts: all
   become: true
   tasks:
 
   - name: update repository index
     apt:
       update_cache: yes
     when: ansible_distribution == "Ubuntu"
 
   - name: install apache2 package
     apt:
       name: apache2
       state: latest
     when: ansible_distribution == "Ubuntu"

   - name: install apache2 package
     apt:
       name:
         - apache2
         - libapache2-mod-php
       state: latest
       update_cache: yes
     when: ansible_distribution == "Ubuntu"

   - name: install apache and php
     package:
       name:
         - "{{ Template:Apache package }}"
         - "{{ Template:Php package }}"
       state: latest
       update_cache: yes

   - name: install updates (Ubuntu)
     apt:
       upgrade: dist
       update_cache: yes
     when: ansible_distribution == "Ubuntu"

   - name: copy html file for site
     tags: apache,apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644
   

# inventory file
172.16.250.132 apache_package=apache2 php_package=libapache2-mod-php
```