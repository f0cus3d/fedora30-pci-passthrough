# fedora-pci-passthrough
Guide for configuring VFIO/IOMMU GPU Passthrough on Fedora 30

# Hardware Configuration
Before you configure anything in the OS you will need to head over to your systems BIOS configuration and ensure that the required virtualization options are enabled. For my current setup I will be using the ASRock x470 Taichi. The options I have enabled are AMD-V and SR-IOV Support.

For Intel setups Intel-VT will need to be enabled.

The GPU's I will be using in this example are the Vega64 and the RX480.

# Fedora 30 VFIO/IOMMU Configuration
1. `sudo dnf update`

It's generally a great idea to update your system before proceeding to ensure you are running the latest version of software installed on your Fedora installation.

2. Run the following script to display your IOMMU groups (Assuming you enabled Hardware Virtualization settings in your systems BIOS; no really go enable them.)
```
#!/bin/bash

shopt -s nullglob

for d in /sys/kernel/iommu_groups/*/devices/*; do

    n=${d#*/iommu_groups/*}; n=${n%%/*}

    printf 'IOMMU Group %s ' "$n"

    lspci -nns "${d##*/}"

done;
```
(I'm not going to cover saving and running the script as it is not in the scope of this guide to cover that.)

When the script is ran you should have an output similar to the following:

##### System IOMMU Groups
```
$ ./iommu.sh 
IOMMU Group 0 00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 10 00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 11 00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
IOMMU Group 12 00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 59)
IOMMU Group 12 00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 13 00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 0 [1022:1460]
IOMMU Group 13 00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 1 [1022:1461]
IOMMU Group 13 00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 2 [1022:1462]
IOMMU Group 13 00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 3 [1022:1463]
IOMMU Group 13 00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 4 [1022:1464]
IOMMU Group 13 00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 5 [1022:1465]
IOMMU Group 13 00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 6 [1022:1466]
IOMMU Group 13 00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 7 [1022:1467]
IOMMU Group 14 01:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981 [144d:a808]
IOMMU Group 15 03:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:43d0] (rev 01)
IOMMU Group 15 03:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset SATA Controller [1022:43c8] (rev 01)
IOMMU Group 15 03:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Bridge [1022:43c6] (rev 01)
IOMMU Group 15 1d:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
IOMMU Group 15 1d:02.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
IOMMU Group 15 1d:03.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
IOMMU Group 15 1d:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
IOMMU Group 15 1d:09.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
IOMMU Group 15 1e:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142 USB 3.1 Host Controller [1b21:2142]
IOMMU Group 15 20:00.0 SATA controller [0106]: ASMedia Technology Inc. ASM1062 Serial ATA Controller [1b21:0612] (rev 02)
IOMMU Group 15 21:00.0 PCI bridge [0604]: ASMedia Technology Inc. ASM1184e PCIe Switch Port [1b21:1184]
IOMMU Group 15 26:01.0 PCI bridge [0604]: ASMedia Technology Inc. ASM1184e PCIe Switch Port [1b21:1184]
IOMMU Group 15 26:03.0 PCI bridge [0604]: ASMedia Technology Inc. ASM1184e PCIe Switch Port [1b21:1184]
IOMMU Group 15 26:05.0 PCI bridge [0604]: ASMedia Technology Inc. ASM1184e PCIe Switch Port [1b21:1184]
IOMMU Group 15 26:07.0 PCI bridge [0604]: ASMedia Technology Inc. ASM1184e PCIe Switch Port [1b21:1184]
IOMMU Group 15 27:00.0 Network controller [0280]: Intel Corporation Dual Band Wireless-AC 3168NGW [Stone Peak] [8086:24fb] (rev 10)
IOMMU Group 15 2a:00.0 Ethernet controller [0200]: Intel Corporation I211 Gigabit Network Connection [8086:1539] (rev 03)
IOMMU Group 16 2e:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev c7)
IOMMU Group 16 2e:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [1002:aaf0]
IOMMU Group 17 2f:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1470] (rev c1)
IOMMU Group 18 30:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:1471]
IOMMU Group 19 31:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 XL/XT [Radeon RX Vega 56/64] [1002:687f] (rev c1)
IOMMU Group 1 00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 20 31:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 HDMI Audio [Radeon Vega 56/64] [1002:aaf8]
IOMMU Group 21 32:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Raven/Raven2 PCIe Dummy Function [1022:145a]
IOMMU Group 22 32:00.2 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Platform Security Processor [1022:1456]
IOMMU Group 23 32:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Zeppelin USB 3.0 Host controller [1022:145f]
IOMMU Group 24 33:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Renoir PCIe Dummy Function [1022:1455]
IOMMU Group 25 33:00.2 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU Group 26 33:00.3 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) HD Audio Controller [1022:1457]
IOMMU Group 2 00:01.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 3 00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 4 00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 5 00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 6 00:03.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 7 00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 8 00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 9 00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
```
Pay special attention to the devices that you will ultimately want to passthrough to a VM, in the case of this guide you want the video card that GPU that you wish to passthrough. For me this is the Vega64 Graphics Card. Isolate the information and make note of the specific device id's because you will need them later.

```
$ ./iommu.sh | grep 'Vega'
IOMMU Group 19 31:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 XL/XT [Radeon RX Vega 56/64] [1002:687f] (rev c1)
IOMMU Group 20 31:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 HDMI Audio [Radeon Vega 56/64] [1002:aaf8]
```
##### Device ID's for Passthrough
```
1002:687f
1002:aaf8
```
You passthrough the GPU and its associated Audio Device if it has one.

3. `sudo dnf install @virtualization`

This will install the needed virtualization software needed to setup and configure a virtual machine on your system.

4. Edit `/etc/sysconfig/grub` and add `iommu=1 amd_iommu=on rd.driver.pre=vfio-pci` to the `GRUB_CMDLINE_LINUX` variable.

Example:
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet iommu=1 amd_iommu=on rd.driver.pre=vfio-pci"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

5. Create `vfio.conf` in the `/etc/modprobe.d/` directory. Add the following to that file.

```
options vfio-pci ids=1002:687f,1002:aaf8
```

This will instruct the vfio module to assume control of the device id's you have set, so that when the system boots the os will not assume control of the devices.

6. Create `vfio.conf` in the `/etc/dracut.conf.d/` directory (Note: This folder was not present when I first installed Fedora30). Add the following to the file.

```
add_drivers+="vfio vfio_iommu_type1 vfio_pci vfio_virqfd"
```
This will allow `dracut` update the initial ram disk on Fedora 30 to include the vfio drivers.

7. Execute: 
```
dracut –f –kver `uname –r`
```

8. Update your GRUB2 configuration. (I'm assuming you have a EFI system.)

```
grub2-mkconfig > /etc/grub2-efi.cfg 
```

9. Reboot your system. You should now check to see if the the vfio kernel drivers have control of the devices you specificed in the above configurations.

```
$ lspci -nnk -d 1002:687f
31:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 XL/XT [Radeon RX Vega 56/64] [1002:687f] (rev c1)
	Subsystem: ASUSTeK Computer Inc. Device [1043:04c4]
	Kernel driver in use: vfio-pci
	Kernel modules: amdgpu


$ lspci -nnk -d 1002:aaf8
31:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 HDMI Audio [Radeon Vega 56/64] [1002:aaf8]
	Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Vega 10 HDMI Audio [Radeon Vega 56/64] [1002:aaf8]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```

Alternatively you can check `dmesg` as well:

```
$ dmesg | grep vfio
[    0.000000] Command line: BOOT_IMAGE=(hd3,gpt2)/vmlinuz-5.0.16-300.fc30.x86_64 root=/dev/mapper/fedora_localhost--live-root ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet iommu=1 amd_iommu=on rd.driver.pre=vfio-pci
[    0.000000] Kernel command line: BOOT_IMAGE=(hd3,gpt2)/vmlinuz-5.0.16-300.fc30.x86_64 root=/dev/mapper/fedora_localhost--live-root ro resume=/dev/mapper/fedora_localhost--live-swap rd.lvm.lv=fedora_localhost-live/root rd.lvm.lv=fedora_localhost-live/swap rhgb quiet iommu=1 amd_iommu=on rd.driver.pre=vfio-pci
[    4.649636] vfio-pci 0000:31:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=none
[    4.661023] vfio_pci: add [1002:687f[ffffffff:ffffffff]] class 0x000000/00000000
[    4.673024] vfio_pci: add [1002:aaf8[ffffffff:ffffffff]] class 0x000000/00000000
[   47.440749] vfio-pci 0000:31:00.0: enabling device (0000 -> 0003)
[   47.441017] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[   47.441026] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0
[   47.454745] vfio-pci 0000:31:00.1: enabling device (0000 -> 0002)
[ 1008.232168] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[ 1008.232178] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0
[ 1204.451195] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[ 1204.451205] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0
[ 1375.582869] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[ 1375.582880] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0
[ 3659.618888] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[ 3659.618898] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0
[ 9297.927847] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x19@0x270
[ 9297.927857] vfio_ecap_init: 0000:31:00.0 hiding ecap 0x1b@0x2d0

```

You should now beable to pass your GPU to a Virtual Machine.

TODO: Guide on adding GPU to VM using Virt Manager
