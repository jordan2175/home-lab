# cloud-config examples #

This directory contains some of my cloud-config configuration elements.

## Cloud Init files left on the system

/etc/cloud/
/run/cloud-init/
/usr/
/var/lib/cloud/
/var/log/cloud*
  /var/log/cloud-init.log
  /var/log/cloud-init-output.log
/var/lib/dpkg/info/cloud*
/var/lib/systemd/deb-systemd-helper-enabled/cloud*
/boot/firmware/user-data

/etc/sudoers.d/90-cloud-init-users
/etc/ssh/sshd_config.d/50-cloud-init.conf
/etc/netplan/50-cloud-init.yaml c

## Disable cloud-init

Create an empty file

`touch /etc/cloud/cloud-init.disabled`

## Check to see if cloud-init is already installed

When making a new image, you need to make sure Cloud Init is already installed.
You can check this with the following command

dpkg --get-selections | grep cloud-init

## Install cloud-init

apt-get install cloud-init

## Remove cloud-init

apt-get purge cloud-init -y 
rm -rf /etc/cloud
rm -rf /var/lib/cloud

cd /etc/cloud

## Need to create a password by using the mkpasswd command

which mkpasswd
mkpasswd -m sha-512



## The whois package has this command

See if whois is installed:
apt search whois

/etc/cloud/cloud.cfg.d
99-fake_cloud.cfg

```
# configure cloud-init for NoCloud
datasource_list: [ NoCloud, None ]
datasource:
  NoCloud:
    fs_label: system-boot
```

## Clean existing cloud-init

cloud-init clean

If the following file exists we need to remove it
/etc/systemd/network/99-default.link

## Run cloud-init without rebooting to check to see if everything works

cloud-init init

