# 710s 13ikb linux

Linux is working well on this device. At least with Ubuntu 16.04 and Linux mint 18.

I quickly tried Ununtu 16.10 which boot successfully from the live disk but once installed the machine refused to boot from; didn't digg at all into this issue.

## Key features
Here are the elements which push the buy for this device:
- Full HD matte screen (don't need touch screen and neither to see my face in a mirror)
- No unremovable cabs (wink to XPS 13)
- light weight: 1.2Kg
- Low price: (XPS 13 is more than 25% expensiver)

The only counterpart is the lack of a USB-C port but I can live with that...

## Requirment
There's a dedicated bios update which allows to set the nvme device mode to ahci.

## Hardware
- Disk 256 GB: Samsung PM951 NVME MZVLV256  - M.2 2281

## Customisation

### NVME optimization
I was very disapointed at first when I quickly compare the write performance with the device I was switching from. Indeed, for a given mysql dump inport I found that the new device was taking twice the time.
Digging into this issue I found that the nobarrier mount option of ext4 (https://www.kernel.org/doc/Documentation/filesystems/ext4.txt) reduce that particular import from 6 minutes to 17 secondes. Following this find I setted that option to the old device and could optimize that import time from 3 minutes to 45 secondes.
Now, is that option safe to use is still a question that I should sort out because I couldn't deternine if the disk is battery-backed. Either as a feature of the disk because or because a labtop is battery backed up by nature.

### Touchpads
All features of the touchpad works out of the box but I found that the touchpad move speed was a bit slow. So digging with xinput I could change the 'AlpsPS/2 ALPS DualPoint TouchPad' device 'Move Speed' parameter which output initialy the following output:
Synaptics Move Speed (277): 1.000000, 1.750000, 0.043687, 0.000000

The third value correpond to the 'AccelFactor' option which I set to 0.100000 which appear better to my taste. To set that permanently edit the /usr/share/X11/xorg.conf.d/50-synaptics.conf file so that it look like the followiing:
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

Reference: ftp://www.x.org/pub/X11R7.5/doc/man/man4/synaptics.4.html
