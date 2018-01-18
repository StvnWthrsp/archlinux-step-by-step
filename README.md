# archlinux-step-by-step
A step-by-step guide to a fresh install of Arch Linux.
Note: This guide was created from my personal experience installing Arch Linux and is not intended to serve as official documentation. I found myself constantly searching and browsing various Arch Wiki pages in order to correctly install the OS, so I decided to compile the steps into this step-by-step document as I executed the commands. This made subsequent installations MUCH easier and streamlined for me, so I thought I’d share it for others. Your mileage may vary, but hopefully this makes the process easier even if it isn’t exactly the same process on your system.
This was done on a system using an Intel CPU and Intel GPU. The biggest difference comes in the display drivers, though other parts of this guide may be incorrect if the hardware is different. 
How to use this guide: The majority of this guide is simply the exact commands to be executed in the command line, the purely informational lines are preceded by a #. Whenever there is a word in italics, that indicates the specific information will change based on your system. For example, boot under “Format Partitions” refers to whichever partition you have created to be your boot partition after using cfdisk. Ex. You would execute the command, “mkfs.ext /dev/sdb1”, replacing boot with the partition.
Again, this was done for personal use, so specific options are added (such as making a sudoer NOPASSWD). You may change or remove such options as you wish.
ARCH LINUX STEP-BY-STEP INSTALLATION AND CONFIGURATION
Set up partitions using cfdisk
	#Boot partition, 10+ GB
	#Swap partition, 2x RAM (though equal to RAM should be acceptable)
	#Extended partition, variable size
Format partitions
	mkfs.ext4 /dev/boot
	mkfs.ext4 /dev/extended
	mkswap /dev/swap
	swapon /dev/swap
Mount partitions
	mount /dev/boot /mnt
	mkdir /mnt/home
	mount /dev/extended /mnt/home
Network Configuration for USB Live Boot
#If using wired network, it should already be working.
#If using wi-fi, it must be configured at this stage.
	iw dev
		#provides the name of the wireless interface. Should start with a “w”.
	ip link set interface up
	iw dev interface scan | less
		#provides information on AP’s in range, including SSID and security (RNS = WPA2)
	No Encryption
	iw dev interface connect SSID
	WEP
	iw dev interface connect “SSID” key 0:your_key
	WPA/WPA2
	wpa_supplicant -D n180211,wext -i interface -c <(wpa_passphrase “SSID” “your_key”)
dhcpd interface
Install Base Packages
pacstrap /mnt base
Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab
Change Root
arch-chroot /mnt
Time Zone
ln -s /usr/share/zoneinfo/US/Eastern /etc/localtime
hwclock –systohc
Locale
nano /etc/locale.gen
	#Uncomment en_US.UTF-8 UTF-8
locale-gen
nano /etc/locale.conf
	#Add line: LANG=en_US.UTF-8
Create & Change Hostname
nano /etc/hostname
	#Add line: myhostname
nano /etc/hosts
	#Add line: 127.0.1.1 myhostname.localdomain myhostname
Network Configuration for Environment
Install iw, wpa_supplicant, dialog packages
Install appropriate firmware packages
	linux-firmware 
#(should be installed already)
nano /etc/wpa_supplicant/control.conf
	ctrl_interface=/run/wpa_supplicant
	update_config=1
wpa_supplicant -B -i interface -c /etc/wpa_supplicant/control.conf
wpa_passphrase SSID passphrase > /etc/wpa_supplicant/wpa_supplicant-interface.conf
systemctl enable wpa_supplicant@interface
Install GRUB
pacman -S grub os-prober
grub-install /dev/drive
grub-mkconfig -o /boot/grub/grub.cfg
Unmount, Reboot
exit
umount /mnt/home
umount /mnt
reboot
Add New User
useradd -m -g users -s /bin/bash username
passwd username
Install sudo, Make New User a sudoer
pacman -S sudo
EDITOR=nano visudo
	#Add line: username ALL=(ALL) NOPASSWD: ALL
Install Some Packages
pacman -S net-tools pkgfile base-devel
Configure Sound (assuming Intel chip)
pacman -S alsa-utils
pacman -S alsa-firmware
nano /etc/modprobe.d/alsa-base.conf
	#Add line: options snd_hda_intel index=1
modprobe -rv snd_hda_intel
mobprobe -v snd_hda_intel
Install Video Drivers
#In most cases, “vesa” will work. But you should install native drivers for better functionality.
pacman -S xf86-video-vesa
pacman -S xf86-video-intel
Install Synaptics (For Laptop Touchpad)
pacman -S xf86-input-synaptics
Install GUI
pacman -S gnome
systemctl enable gdm.service
reboot
