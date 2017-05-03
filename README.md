# Lenovo ideapad 710S-13IKB linux

Linux is working well on this device, at least with Ubuntu 16.04 and Linux mint 18.  ~~Audio need the installation of the latest ALSA driver to work correctly (See instruction bellow).~~ Audio seems to work out of the box since kernel 4.4.0-62 as with Ubuntu 16.04.2 (kernel 4.8.0-39).

I quickly tried Ununtu 16.10 which boot successfully from the live disk but once installed the machine refused to boot from; didn't digg at all into this issue.

## Key features
Here are the elements which push the buy for this device:
- Full HD matte screen (don't need touch screen and neither to see my face in a mirror)
- No unremovable Adaptive Brightness Control (CABC) (wink to XPS 13)
- light weight: 1.2Kg
- Low price: (XPS 13 is more than 25% expensiver)

The only counterpart is the lack of a USB-C port but I can live with that...

## Requirement
There's a dedicated bios update which allows to set the nvme device mode to ahci, thus allows to install linux; download and follow instructions here: https://forums.lenovo.com/t5/Lenovo-Yoga-Series-Notebooks/Yoga-900-and-Ideapad-710S-Linux-Only-BIOS/ta-p/3466850

## Hardware
- Disk 256 GB: Samsung PM951 NVME MZVLV256 - M.2 2281
- Wifi: Intel® Dual Band Wireless-AC 3165 - M.2 2230

inxi return:
```
$ inxi -Abd
System:    Host: 710s Kernel: 4.4.0-51-generic x86_64 (64 bit) Desktop: Unity 7.4.0  Distro: Ubuntu 16.04 xenial
Machine:   System: LENOVO (portable) product: 80VQ v: Lenovo ideapad 710S-13IKB
           Mobo: LENOVO model: Lenovo ideapad 710S-13IKB v: SDK0J40709 WIN
           Bios: LENOVO v: 3HCN12S2 date: 10/07/2016
CPU:       Dual core Intel Core i7-7500U (-HT-MCP-) speed/max: 400/2701 MHz
Graphics:  Card: Intel Device 5916
           Display Server: X.Org 1.18.4 drivers: intel (unloaded: fbdev,vesa) Resolution: 1920x1080@60.02hz
           GLX Renderer: Mesa DRI Intel Kabylake GT2 GLX Version: 3.0 Mesa 11.2.0
Audio:     Card Intel Device 9d71 driver: snd_hda_intel Sound: ALSA v: k4.4.0-51-generic
Network:   Card: Intel Intel Dual Band Wireless-AC 3165 Plus Bluetooth driver: iwlwifi
Drives:    HDD Total Size: NA (-) ID-1: /dev/nvme0n1 model: N/A size: 256.1GB
           Optical: No optical drives detected.
Info:      Processes: 219 Uptime: 25 min Memory: 1022.8/7876.4MB Client: Shell (bash) inxi: 2.2.35
```

## Audio
The audio work unstable on stock install with kernel 4.4.0-51. That is, the audio sometime work and sometime doesn’t on reboot and never after suspend.

Solving this issue requires to install the latest version of ALSA driver and disable secure boot

Update: it looks like this issue isn't present since kernel 4.4.0-62 and installing the latest ALSA driver isn't necessary anymore. Disabling secure boot might be still required.

### Install latest ALSA driver
```
$ sudo apt-add-repository ppa:ubuntu-audio-dev/alsa-daily
$ sudo apt-get update
$ sudo apt-get install oem-audio-hda-daily-dkms
```
Shutdown, wait at least 10 secondes and then power on.

### Disable secure boot
Disabling secure boot in UEFI bios successfully allows to load the snd-hda-intel module.
(source: http://askubuntu.com/a/762255)

## Fn keys
Function keys all works out of the box but the microphone muting switch Fn+F4.

### Microphone Fn key
To switch the microphone muting one can submit the following instruction:
```
$ /usr/bin/xdotool key XF86AudioMicMute
```

Unfortunately, I couldn't add this command in the system settings keyboard shortcuts as the Fn+F4 isn't detected there... 

#### ACPI events
Instead, using acpi avents I could trigger the above commande successfully; here is how:

A. Find the acpi event related to the Fn+F4

Run `acpi_listen` and hit Fn+F4:
```
$ acpi_listen 
button/micmute MICMUTE 00000080 00000000 K
```

B. Read the above event and define which script to trigger

Create a new file as /etc/acpi/events/lenovo-mutemic:
```
event=button/micmute MICMUTE 00000080 00000000 K
action=/etc/acpi/lenovo-mutemic.sh
```

C. Add the triggered script

Create a new file as /etc/acpi/lenovo-mutemic.sh:
```
#!/bin/sh
USER=marcusgarvey
export XAUTHORITY=/home/$USER/.Xauthority
export DISPLAY=:0.0
/usr/bin/xdotool key XF86AudioMicMute
```

## Customization

### NVME optimization
I was very disapointed at first when I quickly compare the write performance with the device I was switching from. Indeed, for a given mysql dump inport I found that the new device was taking twice the time.
Digging into this issue I found that the nobarrier mount option of ext4 (https://www.kernel.org/doc/Documentation/filesystems/ext4.txt) reduce that particular import from 6 minutes to 17 secondes. Following this find I setted that option to the old device and could optimize that import time from 3 minutes to 45 secondes.

Add nobarrier to /etc/fstab example:
```
UUID=8d502364-56dc-45de-xd3c-956ad2941e65 /               ext4    nobarrier,errors=remount-ro 0       1
```

Now, is that option safe to use is still a question that I should sort out because I couldn't deternine if the disk is battery-backed. Either as a feature of the disk because or because a labtop is battery backed up by nature.

### DRI3

It appears that DRI3 Xorg feature isn't enabled by default; to enable it add the following into /etc/X11/xorg.conf:
```
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option      "DRI" "3"
EndSection
```

#### Chromium slow down on multiple workspaces
Enabling DRI3 aslo solve the slow down issue with Chromium when multiple of its windows are used on different workspaces: https://bugs.chromium.org/p/chromium/issues/detail?id=683486#c31

Alternatively, this isssue can also be resolved by setting the LIBGL_DRI3_DISABLE environment variable.

### Touchpads
All features of the touchpad works out of the box but I found that the touchpad move speed was a bit slow. So digging with xinput I could change the 'AlpsPS/2 ALPS DualPoint TouchPad' device 'Move Speed' parameter which output initialy the following output:
Synaptics Move Speed (277): 1.000000, 1.750000, 0.043687, 0.000000

The third value correpond to the 'AccelFactor' option which I set to 0.100000 which appear better to my taste. To set that permanently edit the /usr/share/X11/xorg.conf.d/50-synaptics.conf file so that it look like the following:
```
Section "InputClass"
        Identifier "touchpad catchall"
        Driver "synaptics"
        MatchIsTouchpad "on"
# This option is recommend on all Linux systems using evdev, but cannot be
# enabled by default. See the following link for details:
# http://who-t.blogspot.com/2010/11/how-to-ignore-configuration-errors.html
      MatchDevicePath "/dev/input/event*"

      # Custom 710s (# was 0.043687)
      Option "AccelFactor" "0.100000"
EndSection
```

Reference: ftp://www.x.org/pub/X11R7.5/doc/man/man4/synaptics.4.html

### Swappiness
Swappiness is set to 60 by default:
```
$ cat /proc/sys/vm/swappiness
```

We can reduce this as we are using an SSD type disk with the following then reboot:
```
echo vm.swappiness=20 | sudo tee -a /etc/sysctl.conf
```
