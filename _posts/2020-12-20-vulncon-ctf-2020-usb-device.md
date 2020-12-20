---
title: "VULNCON CTF 2020 - USB Device writeup"
excerpt: "Writeup for the USB Device chall from [VULNCON CTF 2020](https://ctftime.org/event/1149)."
date: 2020-12-20T15:41:30-03:00
categories:
  - Cybersecurity
tags:
  - Memory Forensic
  - VULNCON CTF 2020
  - CTF
---

## Description:

```
He told me that some usb device is connected with his PC. I want to figure out ContainerID of connected USB device. Can you help me with this?

Flag Format: vulncon{containerID}

Author - r3curs1v3_pr0xy
```

## Solution

The Volatility plugin used this time was [USBSTOR](https://github.com/kevthehermit/volatility_plugins):

```
$ python2 ~/tools/volatility/vol.py --plugins=volatility_plugins/usbstor/ --profile=Win7SP1x64 -f dump.raw usbstor
Volatility Foundation Volatility Framework 2.6.1
Reading the USBSTOR Please Wait
Found USB Drive: CCYYMMDDHHmmSSX1TIOR&0
        Serial Number:  CCYYMMDDHHmmSSX1TIOR&0
        Vendor: SMI
        Product:        USB_DISK
        Revision:       1100
        ClassGUID:      USB_DISK

        ContainerID:    {68b70eb8-f3fd-5099-907d-4e542601b2c7}
        Mounted Volume: \??\Volume{f7d58027-3b76-11eb-a2d8-d0abd5a4ad75}
        Drive Letter:   \DosDevices\E:
        Friendly Name:  SMI USB DISK USB Device
        USB Name:       Unknown
        Device Last Connected:  2020-12-11 06:19:46 UTC+0000

        Class:  DiskDrive
        Service:        disk
        DeviceDesc:     @disk.inf,%disk_devdesc%;Disk drive
        Capabilities:   16
        Mfg:    @disk.inf,%genmanufacturer%;(Standard disk drives)
        ConfigFlags:    0
        Driver: {4d36e967-e325-11ce-bfc1-08002be10318}\0001
        Compatible IDs:
                USBSTOR\Disk
                USBSTOR\RAW


        HardwareID:
                USBSTOR\DiskSMI_____USB_DISK________1100
                USBSTOR\DiskSMI_____USB_DISK________
                USBSTOR\DiskSMI_____
                USBSTOR\SMI_____USB_DISK________1
                SMI_____USB_DISK________1
                USBSTOR\GenDisk
                GenDisk


Windows Portable Devices
        --
        FriendlyName:   E:\
        Serial Number:  CCYYMMDDHHMMSSX1TIOR&0
        Last Write Time:        2020-12-11 06:19:59 UTC+0000
```

The containerID is showed, so the flag is:

`vulncon{68b70eb8-f3fd-5099-907d-4e542601b2c7}`
