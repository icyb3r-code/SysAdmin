
## 101.1 Determine and configure hardware settings

### lspci 

lspci (peripheral component interconnect) bus

```bash
# this tool reads information from these files
/proc/cpuinfo
# Lists detailed information about the CPU(s) found by the operating system.
/proc/interrupts
# A list of numbers of the interrupts per IO device for each CPU.
/proc/ioports
# Lists currently registered Input/Output port regions in use.
/proc/dma
# Lists the registered DMA (direct memory access) channels in use.
# to show more details about device
lspci -s 04:00.0 -v  

# to show kernel modules related to this device 
lspci -s 04:00.0 -k 

```


### lsusb

```bash

$ lsusb -d 1d6b:0003 -v

Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Couldnt open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               3.00
  bDeviceClass            9 Hub
  bDeviceSubClass         0
  bDeviceProtocol         3
  bMaxPacketSize0         9
  idVendor           0x1d6b Linux Foundation
  idProduct          0x0003 3.0 root hub
  bcdDevice            6.00

$ lsusb -t

/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/8p, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/16p, 480M
    |__ Port 2: Dev 7, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 2: Dev 7, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 3: Dev 6, If 2, Class=Audio, Driver=snd-usb-audio, 12M
    |__ Port 3: Dev 6, If 0, Class=Audio, Driver=snd-usb-audio, 12M
    |__ Port 3: Dev 6, If 3, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 3: Dev 6, If 1, Class=Audio, Driver=snd-usb-audio, 12M
    |__ Port 6: Dev 3, If 0, Class=Wireless, Driver=btusb, 12M
    |__ Port 6: Dev 3, If 1, Class=Wireless, Driver=btusb, 12M
    |__ Port 7: Dev 4, If 0, Class=Video, Driver=uvcvideo, 480M
    |__ Port 7: Dev 4, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 8: Dev 5, If 0, Class=Human Interface Device, Driver=usbhid, 12M

$ lsusb -s 01:04

Bus 001 Device 004: ID 13d3:5666 IMC Networks USB2.0 HD UVC WebCam


```

### Kernel Modules

#### lsmod

```bash
# Used by is the Depending modules.
# Size Amount of RAM occupied by the module, in bytes.
$ lsmod | head
Module                  Size  Used by
tls                   122880  7
vmnet                  61440  13
vmw_vsock_vmci_transport    32768  0
vsock                  49152  1 vmw_vsock_vmci_transport
vmw_vmci               90112  1 vmw_vsock_vmci_transport
vmmon                 135168  0
ccm                    20480  6
rfcomm                 86016  4


~ » lsmod | fgrep -i snd_hda_intel
snd_hda_intel          53248  1
snd_intel_dspcfg       36864  1 snd_hda_intel
snd_hda_codec         172032  4 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec_realtek
snd_hda_core          114688  5 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek
snd_pcm               155648  5 snd_hda_codec_hdmi,snd_hda_intel,snd_usb_audio,snd_hda_codec,snd_hda_core
snd                   114688  19 snd_hda_codec_generic,snd_seq,snd_seq_device,snd_hda_codec_hdmi,snd_hwdep,snd_hda_intel,snd_usb_audio,snd_usbmidi_lib,snd_hda_codec,snd_hda_codec_realtek,snd_timer,snd_pcm,snd_rawmidi
~ »

```



#### modprobe 

```bash
# need root, and below will remove the module from kernel
$ modprobe -r snd-hda-intel
```

#### modinfo 

```bash
# show all nouveau module parameters 
$ modinfo -p nouveau
vram_pushbuf:Create DMA push buffers in VRAM (int)
kms_vram_pushbuf:Place EVO/NVD push buffers in VRAM (default: auto) (int)
tv_norm:Default TV norm.
                Supported: PAL, PAL-M, PAL-N, PAL-Nc, NTSC-M, NTSC-J,
                        hd480i, hd480p, hd576i, hd576p, hd720p, hd1080i.
                Default: PAL
                *NOTE* Ignored for cards with external TV encoders. (charp)
nofbaccel:Disable fbcon acceleration (int)
fbcon_bpp:fbcon bits-per-pixel (default: auto) (int)
mst:Enable DisplayPort multi-stream (default: enabled) (int)
tv_disable:Disable TV-out detection (int)
ignorelid:Ignore ACPI lid status (int)
duallink:Allow dual-link TMDS (default: enabled) (int)
hdmimhz:Force a maximum HDMI pixel clock (in MHz) (int)
config:option string to pass to driver core (charp)
debug:debug string to pass to driver core (charp)
noaccel:disable kernel/abi16 acceleration (int)
modeset:enable driver (default: auto, 0 = disabled, 1 = enabled, 2 = headless) (int)
atomic:Expose atomic ioctl (default: disabled) (int)
runpm:disable (0), force enable (1), optimus only default (-1) (int)


# to add parameters to for nouveau by just create and add 
vim /etc/modprobe.d/nouveau.conf
options nouveau modeset=0
# or add it to this file 
vim /etc/modprobe.conf
options nouveau modeset=0


#### Blacklist the module by just add it to this file or create new one 

vim /etc/modprobe.d/blacklist.conf
blacklist nouveau
# or create
vim /etc/modprobe.d/nouveau.conf
blacklist nouveau

```

### Storage Devices 

In linux storage devices generically referred as block devices, and each block identified by a file in the `/dev` directory.

IDE: /dev/hda,/dev/hdb .. etc.
Floppy: /dev/fd0,/dev/fd1 .. etc.

In linux kerenel 2.4 onwards, most storage devices are now identified as if they were SCSI devices regardless of their hardware IDE, SSD, and USB block will be prefixed by sd. 

## 101.2 Boot the System

### BIOS or UEFI


### The Bootloader


From the GRUB menu it is possible to choose which one of the installed kernels should be loaded  
and to pass new parameters to it. Most kernel parameters follow the pattern option=value. Some of  
the most useful kernel parameters are:  
* acpi  
Enables/disables ACPI support. acpi=off will disable support for ACPI.  
* init  
Sets an alternative system initiator. For example, init=/bin/bash will set the Bash shell as the  
initiator. This means that a shell session will start just after the kernel boot process.  
* systemd.unit  
Sets the systemd target to be activated. For example, systemd.unit=graphical.target. Systemd  
also accepts the numerical runlevels as defined for SysV. To activate the runlevel 1, for example,  
it is only necessary to include the number 1 or the letter S (short for “single”) as a kernel  
parameter.
* **mem**
Sets the amount of available RAM for the system. This parameter is useful for virtual machines  
so as to limit how much RAM will be available to each guest. Using mem=512M will limit to 512  
megabytes the amount of available RAM to a particular guest system.  
* **maxcpus**
Limits the number of processors (or processor cores) visible to the system in symmetric multiprocessor machines. It is also useful for virtual machines. A value of 0 turns off the support for  
multi-processor machines and has the same effect as the kernel parameter nosmp. The parameter  
maxcpus=2 will limit the number of processors available to the operating system to two.  
* quiet  
Hides most boot messages.  
* vga  
Selects a video mode. The parameter vga=ask will show a list of the available modes to choose  
from.  
* root  
Sets the root partition, distinct from the one pre-configured in the bootloader. For example,  
root=/dev/sda3.  
rootflags  
Mount options for the root filesystem.  
ro  
Makes the initial mount of the root filesystem read-only.  
rw  
Allows writing in the root filesystem during initial mount.


### Initialization Inspection 

```bash
# list Kernel ring buffer messages  
dmesg 

# to clear messages 
dmesg --clear 

# Read log human mode 
dmesg -H --human 

# will show initialization messages 
journalctl -b / --boot 
journalctl -k / --dmesg
journalctl --list-boots 
journalctl -b 0 / --boot=0

# read logs of another machine from active machine 
journalctl -D /mnt/hd/var/log/journal
```


### systemd


```bash

# the most common systemctl actions will be:  
systemctl start unit.service  
#Starts unit.  

systemctl stop unit.service  
# Stops unit.  

systemctl restart unit.service  
# Restarts unit.  

systemctl status unit.service  
# Shows the state of unit, including if it is running or not.  

systemctl is-active unit.service  
# Shows active if unit is running or inactive otherwise.  

systemctl enable unit.service  
# Enables unit, that is, unit will load during system initialization.  

systemctl disable unit.service  
# unit will not start with the system.  

systemctl is-enabled unit.service  
# Verifies if unit starts with the system. The answer is stored in the variable $?. The value 0  indicates that unit starts with the system and the value 1 indicates that unit does not starts with the system.


systemctl isolate multi-user.target


# alternate between levels 
systemctl isolate multi-user.target

# set default run level 
systemctl set-default multi-user.target

systemctl list-unit-files
systemctl list-unit-files --type=service/target/mount

# list active units 
systemctl list-units


# systemd responding to 


```


```bash

# to check the run level 
runlevel 

# to switch the runlevel from 0 to 6 without reboot 
telinit 0 - 6 


```


### GRUB2

```bash

# Default configuration files for grub

# below not recommanded to edit it manually 
/boot/grub/grub.cfg

# below you can edit it and apply the changes using below command 
/etc/default/grub
update-grub # its shortcut for below command 
grub-mkconfig -o /boot/grub/grub.cfg 



```





