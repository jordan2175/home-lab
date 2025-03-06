# Birth Certificate

I find it helpful to create a birth certificate for cloud nodes and cluster
nodes so that I can keep track of their genesis. Once created they can be found
at `/etc/birth_certificate` and will look like the following example.

```
------------------------------------------------------------
Birth Certificate
------------------------------------------------------------
Product:      Raspberry Pi Compute Module 4 Rev 1.1
Birth Date:   Thu Jan 30 16:16:05 MST 2025
Distro:       Ubuntu 24.04.1 LTS
Kernel:       6.8.0-1010-raspi
Eth0 Address: d8:3a:dd:a8:7a:d0
------------------------------------------------------------
```


## Cloud Init Version

```
#cloud-config

## --------------------------------------------
## Run Commands
## --------------------------------------------
# This command will only run the very first time the system is booted
runcmd:
- echo "------------------------------------------------------------" > /etc/birth_certificate
- echo "Birth Certificate" >> /etc/birth_certificate
- echo "------------------------------------------------------------" >> /etc/birth_certificate
- [sh, -c, 'echo -n ''Product:     '' >> /etc/birth_certificate']
- lshw -quiet -c system | grep product | cut -s -d ':' -f2 >> /etc/birth_certificate
- [sh, -c, 'echo -n ''Birth Date:   '' >> /etc/birth_certificate']
- date >> /etc/birth_certificate
- [sh, -c, 'echo -n ''Distro:       '' >> /etc/birth_certificate']
- lsb_release -d -s 2>/dev/null >> /etc/birth_certificate
- [sh, -c, 'echo -n ''Kernel:       '' >> /etc/birth_certificate']
- uname -r >> /etc/birth_certificate
- [sh, -c, 'echo -n ''Eth0 Address: '' >> /etc/birth_certificate']
- cat /sys/class/net/eth0/address >> /etc/birth_certificate
- echo "------------------------------------------------------------" >> /etc/birth_certificate
```


## Boot log: Cloud Init Version

Here is an example of what one could do if they wanted to add a boot log to
their birth certificate. 


```
- echo "Boot Log" >> /etc/birth_certificate
- echo "------------------------------------------------------------" >> /etc/birth_certificate
- date >> /etc/birth_certificate

# This command will run everytime the system is booted up
bootcmd:
- date >> /etc/birth_certificate
```
