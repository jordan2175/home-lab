# Flashing Nodes

You can flash nodes through three primary ways

 1. Web interface
   - This is the easiest
   - Can be slow
   - Images are stored on local computer
   - Does not support automated re-flashing
 2. TPI interface
   - This can take 20 minutes per node to flash
   - Images can be stored on the BMC's micro SD card
     - The micro SD card is located on the bottom of the TuringPI board 
     - Access to the SD card can be challening
   - Could support automated re-flashing
     - BMC firmware does not support loop back interfaces
     - Meaning, no ability to mount the img file and make cloud-init changes
 3. USB 2.0 port
   - Mounts the node as a mass storage device
   - Requires rpi-boot software on local computer
   - Can use graphical tool RPI Imager

## Update img file from command line

Check partition structure of the image:

`fdisk -l ubuntu-24.04.img`

The following commands will first create a loop back interface and then mount that interface. Once it is mounted you can either edit the cloud-init user-data file directly or copy over a new version, which is what is done here. Once edits are done, one needs to unmount and detach the loop back interface.

Since this does not work on the BMC's firmware itself, I had to do this on another computer (the gate server) and then SCP the images to the BMC's micro SD card. This takes about 12 mintues to copy over.

Commands

`
(
losetup -P /dev/loop2 /local/images/ubuntu-24.04.1-preinstalled-server-arm64+raspi.img
mount /dev/loop2p1 /mnt/node-img
cp /local/images/user-data/user-data-node04 /mnt/node-img/user-data
umount /mnt/node-img
losetup -d /dev/loop2
scp ubuntu-24.04.1-preinstalled-server-arm64+raspi.img root@10.128.64.20:/mnt/sdcard/images/ubuntu-24.04-node04.img
)
`

## Flash Nodes (TPI)

`
tpi flash -n 4 -l -i /mnt/sdcard/images/ubuntu-24.04-node04.img
`

