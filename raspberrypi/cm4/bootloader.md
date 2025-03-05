# Raspberry Pi Compute Module 4

Each module has a boot loader that may be updated from time to time.

## Initial Bootloader

The modules I purchased were running the "Wed 11 Jan 17:40:52 UTC 2023 (1673458852)" bootloader. I was able to see this by flashing the module with Raspberrypi OS and running the rpi-eeprom-update tool. This produced the following.

```
root@raspberrypi:/home/foouser# rpi-eeprom-update 
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
   CURRENT: Wed 11 Jan 17:40:52 UTC 2023 (1673458852)
    LATEST: Mon 15 Apr 13:12:14 UTC 2024 (1713186734)
   RELEASE: default (/lib/firmware/raspberrypi/bootloader-2711/default)
            Use raspi-config to change the release.

  VL805_FW: Using bootloader EEPROM
     VL805: up to date
   CURRENT: 
    LATEST: 
```

## Bootloader Configuration

One can also see the current EEPROM configuration with the following:
```
root@raspberrypi:/home/foouser# rpi-eeprom-config 
[all]
BOOT_UART=0
WAKE_ON_GPIO=1
POWER_OFF_ON_HALT=0

# Try SD first (1), followed by, USB PCIe, NVMe PCIe, USB SoC XHCI then network
BOOT_ORDER=0xf25641

# Set to 0 to prevent bootloader self-updates (without recovery.bin)
# For remote units EEPROM hardware write protection should be used.
ENABLE_SELF_UPDATE=1

# Set to 1 to disable the bootloader HDMI diagnostics screen AND
# network install e.g. for kiosk devices.
DISABLE_HDMI=0
```

## Check Bootloader Version

 - Add the following lines to /etc/default/rpi-eeprom-update
   - RPI_EEPROM_USE_FLASHROM=1
   - CM4_ENABLE_RPI_EEPROM_UPDATE=1
 - From your computer ssh to the module
   - Run `rpi-udpate` only on raspberry pi OS to get latest firmware
   - Run `rpi-eeprom-update`

```
root@raspberrypi:/home/foouser# rpi-eeprom-update 
BOOTLOADER: up to date
   CURRENT: Mon 21 Oct 14:24:54 UTC 2024 (1729520694)
    LATEST: Mon 21 Oct 14:24:54 UTC 2024 (1729520694)
   RELEASE: latest (/lib/firmware/raspberrypi/bootloader-2711/latest)
            Use raspi-config to change the release.

  VL805_FW: Using bootloader EEPROM
     VL805: up to date
   CURRENT: 
    LATEST: 
```

## Upgrade Bootloader (CM4)

### Download Firmware

Firmware for CM4 can be found at the following URL
https://github.com/raspberrypi/rpi-eeprom/tree/master/firmware-2711/latest
 
 - BCM2711 = CM4
 - BCM2712 = CM5

### Install RPI Boot Tool

Install the Raspberry Pi `rpiboot` tool on your Mac laptop. Do all of the following on your Mac laptop.

 - Make a directory for this code
   - mkdir -p ~/raspberrypi
   - cd ~/raspberrypi/
 - Install USB libraries
   - brew install libusb
 - Download USBBoot Tool
   - git clone --depth=1 https://github.com/raspberrypi/usbboot
   - cd usbboot
   - git pull = to get current version if it has been a while since clone
 - Get EEPROM Firmware Files
   - From inside the usbboot directory do:
   - git clone --depth 1 https://github.com/raspberrypi/rpi-eeprom.git
   - cd rpi-eeprom
   - git pull = to get current version if it has been a while since clone
 - Compile USBBoot
   - cd ..
   - From inside the usbboot directory do:
   - make
 - Check to make sure firmware is actually there
   - cd recovery
   - In this directory you will have 
     - `pieeprom.original.bin -> ../firmware/2711/pieeprom.bin`
   - Check to see if the latest firmware is actually there
     - cd ../firmware/2711/
   - In here you will see 
     - `pieeprom.bin -> ../../rpi-eeprom/firmware-2711/latest/pieeprom-2025-02-11.bin`
   - Lets go check to see if the firmware is actually in this directory
   - cd ../../rpi-eeprom/firmware-2711/latest/
   - If it is not there then you can do the following
     - a git pull in the rpi-eeprom directory
     - Or download it from the URL right above and put in this directory
 - Prepare Bootloader Firmware
   - cd usbboot/recovery
   - ./update-pieeprom.sh 

### Install Bootloader on CM4

 - Set the CM4 to boot in Mass Storage Device mode (with disable eMMC boot jumper set)
 - From the recovery directory type `../rpiboot -d .`
 - Once this is finishd you can remove the "disable eMMC boot jumber"
 - Boot the CM4 in normal mode without the jumper
 - Now when you run `rpi-eeprom-udpate` from the CM4 shell you will see it running the latest bootloader
 - The reason the "latest" is older, is the package repositories and index is behind

```
root@raspberrypi:/home/foouser# rpi-eeprom-update
BOOTLOADER: up to date
   CURRENT: Tue 11 Feb 17:00:13 UTC 2025 (1739293213)
    LATEST: Mon 21 Oct 14:24:54 UTC 2024 (1729520694)
   RELEASE: latest (/lib/firmware/raspberrypi/bootloader-2711/latest)
            Use raspi-config to change the release.

  VL805_FW: Using bootloader EEPROM
     VL805: up to date
   CURRENT: 
    LATEST: 
```

### You can also flash the bootloader in the Turing Pi board

 - Shutdown the CM4 OS
 - Power the Node Off in the Turing Pi config
 - Set the USB to Flash mode
 - Connect the USB A to USB A+USBC adapter to your mac
 - Turn on the Node
 - Run the rpiboot command from above
 - From the recovery directory ../rpiboot -d .`
 - Make sure to allow the connection to your Mac, I got asked twice
 - Turn node off
 - Switch USB mode back
 - Turn node on 

### Update via RASPI Config

You can update via rsapi-config if you have it installed on the CM4

 - raspi-config -> Update
 - raspi-config -> Advanced Options -> Bootloader Version
   - Do not reset the boot config on exit
 - reboot CM4

```
root@raspberrypi:/home/foouser# rpi-eeprom-update 
BOOTLOADER: up to date
   CURRENT: Mon 21 Oct 14:24:54 UTC 2024 (1729520694)
    LATEST: Mon 21 Oct 14:24:54 UTC 2024 (1729520694)
   RELEASE: latest (/lib/firmware/raspberrypi/bootloader-2711/latest)
            Use raspi-config to change the release.

  VL805_FW: Using bootloader EEPROM
     VL805: up to date
   CURRENT: 
    LATEST: 
```

## Boot Order

The BOOT_ORDER property defines the sequence for the different boot modes. 
It is read **right to left**, and up to eight digits may be defined.

```
Value - Mode         - Description
0x0 - SD CARD DETECT - Try SD then wait for card-detect to indicate that the card 
                       has changed. Deprecated now that 0xf (RESTART) is available.
0x1 - SD CARD        - SD card (or eMMC on Compute Module 4).
0x2 - NETWORK        - Network boot - See Network boot server tutorial.
0x3 - RPIBOOT        - RPIBOOT - See usbboot.
0x4 - USB-MSD        - USB mass storage boot - See USB mass storage boot.
0x5 - BCM-USB-MSD    - USB 2.0 boot from USB Type C socket (CM4: USB type A 
                       socket on CM4IO board). Not available on Raspberry Pi 5.
0x6 - NVME           - CM4 and Pi 5 only: boot from an NVMe SSD connected to the 
                       PCIe interface. See NVMe boot for more details.
0x7 - HTTP           - HTTP boot over ethernet. See HTTP boot for more details.
0xe - STOP           - Stop and display error pattern. A power cycle is required 
                       to exit this state.
0xf - RESTART        - Restart from the first boot-mode in the BOOT_ORDER field 
                       i.e. loop.
```
