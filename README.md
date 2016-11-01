
#Xiaomi Air 13: minimal Ubuntu installation  and power optimization steps


## Preparation

 * Make bootable flash drive. Personally I don't use flash drives and I have only one with some files on it. I used this wiki https://wiki.archlinux.org/index.php/Multiboot_USB_drive to prepare multi boot usb.
 * Download your favorite ubuntu distro. Without reason i chose xubuntu 16.04 (LTS).  
 * Power off laptop, power on nad press F2 to get in BIOS(?). If I do reboot F2 doesn't work.
 * Select usb to boot from. If you created usb without UEFI choose legacy mode in boot menu. Save - Reboot 
 * Boot to xubuntu-desktop and run gparted in terminal: `sudo gparted`. I created 4 partitions for root  - / , for /home, swap 8Gb and EFI one (200Mb). Efi partition I did with idea to change boot linux or  windows in bios only. Thus I can make grub transparent (without menu and s etc.) when laptop start from hibernate.     
 * You may check power consumption with live usb, in terminal  `upower -d` in my case with USB plugged it was about 7W
 * I downloaded lubuntu alternate to make minimal installation
 
## Installation
 * You may want use wifi, in terminal: `sudo modprobe -r acer_wmi; sudo service network-manager restart` 
 * Start Installation icon. Make some steps. Downloading updates and HD youtube music video will slow down installation.
 * At page with third-party software you must `Turn off secure boot` . Without this step we will not be able install efficient NVIDIA driver blob.
 * Answer no to unmount /dev/sda
 * In HDD partition selection I chose `Something else` to do manual selection.
 * Chose partitions for / and /home and uefi
 * Make obvious steps: your location, your language, your login details
 * reboot in ubuntu you probably will need select proper boot order in BIOS
 
 ## Configuration
 * Unlock secure boot: follow instructions on blue screen. 
 * WiFi:
 
```
sudo modprobe -r acer_wmi
sudo service network-manager restart
sudo -i
echo 'blacklist acer_wmi' >> /etc/modprobe.d/blacklist.conf
exit
```
* NVIDIA blob, I used version 370, but 367 is stable

> Warning: Don't do  it on Debian. Use How-to for NVIDIA driver from official wiki

``` 
sudo add-apt-repository ppa:graphics-drivers/ppa 
sudo apt update
sudo apt dist-upgrade
sudo apt install  nvidia-370
```
In result you will get two errors with  intel video firmware:

```
W: Possible missing firmware /lib/firmware/i915/kbl_dmc_ver1.bin for module i915_bpo
W: Possible missing firmware /lib/firmware/i915/skl_guc_ver6.bin for module i915_bpo
```
Download it from   http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git/tree/i915

```
cd /tmp/
wget http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git/tree/i915/skl_guc_ver6_1.bin
wget http://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git/tree/i915/kbl_dmc_ver1_01.bin
```
Here you can create links, but i just copied. Update initramfs.

```
sudo cp /tmp/kbl_dmc_ver1_01.bin /lib/firmware/i915/kbl_dmc_ver1.bin
sudo cp /tmp/skl_guc_ver6_1.bin /lib/firmware/i915/skl_guc_ver6.bin
sudo  update-initramfs -k $(uname -r) -u
```
* Once drivers installed, just activate laptopmode. You can add second line to /etc/rc.local in line before exit 0

```
sudo -i
echo 5 > /proc/sys/vm/laptop_mode
```
you also may want remove bluetooth and camera modules from kernel

```
modprobe -r btusb
modprobe -r uvcvideo
```
One think more, switch to intel video card with prime-select

```
sudo prime-select intel
sudo prime-select quiery

```

After this step I got about 4W power consumption with wifi on at screen brightness 5%  running awesome wm. 

```
xbacklight -set 5
sleep 180
upower -d
```
![alt text](https://github.com/Golovin-Andrey/xiaomi-mi-13-ubuntu/blob/master/power-drivers.png "Power consumption")

* Installation tlp and tlp radio with default settings can reduce power to bit less in my case 3.97W
* In my case, when I removed  btusb, uvcvideo, iwlwifi modules  i got only 3.8W . I can conclude that these drivers are  realy efficient.
* And finaly I removed XFCE and get no improvement in energy-rate 

### Conclusion: with proper drivers xiaomi air 13 power consumption in ubuntu is just equal to windows 10 one. Timelife is about 10 hours.

## Some tricks
* Swich off wifi

```
nmcli r wifi off
```

* For proper sleep and hybernate edit /etc/systemd/logind.conf , my noncomented lines are:

```
InhibitDelayMaxSec=5
HandlePowerKey=hibernate
HandleSuspendKey=suspend
HandleHibernateKey=hibernate
HandleLidSwitch=suspend
```

* I use autofs to mount flash and samba/nfs resourses. Configs are folder autofs

* For awesome wm brightness control was done with xbacklight
```
    awful.key({ }, "XF86MonBrightnessDown", function () awful.util.spawn("xbacklight -dec 5") end),
    awful.key({ }, "XF86MonBrightnessUp",   function () awful.util.spawn("xbacklight -inc 5") end),
```

* I use parcellite to manage clipboard, It's cool thing 
```
  parcellite -n &
```

* I added to /etc/rc.local , I don't want blacklist it.
```
# remove camera
modprobe -r uvcvideo
```

## PRIME technology with bumblebee

* Installation

> warning ! it's just unstable, probably you may want wait for bumblebee 4.0
 ```
 sudo add-apt-repository ppa:bumblebee/testing
 sudo apt update
 sudo apt upgrade
 ```

Edit `/etc/bumblebee/bumblebee.conf` 
  
 * adjust kernel driver to `KernelDriver=nvidia`
 * replace `nvidia-current` to `nvidia-370` to get this in bumblebee.conf:
 
 ```
 # colon-separated path to the nvidia libraries
LibraryPath=/usr/lib/nvidia-370:/usr/lib32/nvidia-370
# comma-separated path of the directory containing nvidia_drv.so and the
# default Xorg modules path
XorgModulePath=/usr/lib/nvidia-370/xorg,/usr/lib/xorg/modules
```

* Checking 
 * stop bumblebee service and run in debug mode

```
sudo service bumblebee stop
sudo bumblebeed --debug
```

* run glxgears in new terminal and check for ERROR lines in bumblebee output

```
primusrun glxgears
```

normal output is :

```
[  579.639172] [INFO]Unloading module nvidia_drm
[  579.648303] [INFO]Unloading module nvidia_modeset
[  579.668459] [INFO]Unloading module nvidia_uvm
[  579.688268] [INFO]Unloading module nvidia
[  579.713437] [INFO]Switching dedicated card OFF [bbswitch]
[  579.729761] [DEBUG][XORG] (II) NVIDIA(GPU-0): Deleting GPU-0

 ```
 
 bbswitch output should be OFF
 
 ```
~$ cat /proc/acpi/bbswitch; primusrun glxinfo|grep vendor ; cat /proc/acpi/bbswitch; sleep 5; cat /proc/acpi/bbswitch
0000:01:00.0 OFF
server glx vendor string: NVIDIA Corporation
client glx vendor string: primus
OpenGL vendor string: NVIDIA Corporation
0000:01:00.0 ON
0000:01:00.0 OFF
```

now you can run your games:)

```
primusrun steam
```
