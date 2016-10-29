
#Minimal Ubuntu Installation  and power optimization steps

## Preparation

 * Make bootable flash drive. Personally I don't use flash drives and I have only one with some files on it. I used this wiki https://wiki.archlinux.org/index.php/Multiboot_USB_drive to prepare multi boot usb.
 * Download your favorite ubuntu distro. Without reason i chose xubuntu 16.04 (LTS).  
 * Power off laptop, power on nad press F2 to get in BIOS(?). If I do reboot F2 doesn't work.
 * Select usb to boot from. If you created usb without UEFI choose legacy mode in boot menu. Save - Reboot 
 * Boot to xubuntu-desktop and run gparted in terminal: `sudo gparted`. I created 3 partitions for root  - / , for /home and EFI one (200Mb). Efi partition I did with idea to change boot linux or  windows in bios only. Thus I can make grub transparent (without menu and s etc.) when laptop start from hibernate.     
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
sudo  update-initramfs -k `uname -r` -u
```
Once drivers installed, just activate laptopmode. You can add second line to /etc/rc.local in line before exit 0
```
sudo -i
echo 5 > /proc/sys/vm/laptop_mode
```

After this step I got about 4W power consumption with wifi on at screen brightness 5%  running awesome wm. 
```
xbacklight -set 5
sleep 180
upower -d
```
![alt text](https://github.com/Golovin-Andrey/xiaomi-mi-13-ubuntu/blob/master/power-drivers.png "Power consumption")

