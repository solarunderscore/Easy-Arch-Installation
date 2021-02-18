# Easy Arch Installation
*A simple, quick, and easy Arch Installation guide.*

This is meant to help people who are new to Arch Linux and need a quick installation guide, I know the Arch Wiki can be confusing sometimes so this is why I made this! This is also a little note for myself to use for Arch Linux installations. Congratulations on trying to install Arch, don't worry it won't be as hard as you think! **(Note: Please backup everything before you start anything on this guide, you might have a chance of messing up and wiping drives you don't intend to wipe!)**

## Prerequisites (Links are at the end of this section)
First things first, you obviously need a 4-8GB flash drive for your Arch install. Get this useful program called "balenaetcher", that's should help you burn your ISO to the USB drive. After you get that, get the Arch Linux ISO from the mirror list. Chose the one called "archlinux-XXXX.XX.XX-x86_64.iso" (the X's are for the Arch Linux version update date). Fire up balenaetcher and select where you downloaded your Arch Linux file, click Next, and select the drive you want to burn the USB to **(Careful, this will delete everything on your drive so please back it up if you have anything on it).** Continue with the process and it shall burn it to the USB!

*Links:*  
Arch Linux ISO Download: https://archlinux.org/download/  
balenaetcher Donwload (Windows, MacOS, Linux download): https://www.balena.io/etcher/

## Boot from the USB Install
Depending on what motherboard (it is different for every motherboard, please Google your motherboard type and look for its boot menu button), you need to get to your boot menu and select the USB you just created. Select the USB and hit ENTER.

## Installation
### Verify the boot mode
**For this walkthrough I will be using the Arch Wiki installation guide as a guide for me as well, you can check the installation guide that they have [here](https://wiki.archlinux.org/index.php/Installation_guide).**  
Once you boot into your disk you must wait until you see the command line or something that says "root@archiso ˜ #", that is when you are all set!
If you have a keymap that is not the default (US QWERTY) you will need to load it and apply it, but in my case, I am using QWERTY and do not need to change this, so I will be skipping over this. If you do need to load a keymap please refer to the Arch Wiki above.
Firstly, please verify your boot mode by entering this command.  
`ls /sys/firmware/efi/efivars`  
If you see some kind of output this is good, if you do not see an output this guide isn't for you as you will need some extra steps.  

### Setting up a network connection
Now we will set up an internet connection. For this guide, I am using ethernet and if you are using WiFi please refer back to the Arch Wiki. Run the command,  
`ip link`

You should see all your internet devices. Next, you want to verify you have an internet connection by running the command,  
`ping archlinux.org`  
If you see stuff like, `64 bytes from archlinux.org etc.` then you are set and you have an internet connection. Press Control + C to quit the ping command.

### Syncing the system clock
At this point, you want to update the system clock. To do this run the command,  
`timedatectl set-ntp true`  
If you don't see an output you are good, if you see an output wait until you see `[ OK ] Reached target Garphical Interface` and press Control + C to exit that.  

To check if the system clock is succesfully synced type:  
`timedatectl status`  
If you see `System clock synchronized: yes` you are good to go!

### Partitioning the disks
It is time to partition the disk (you should've already backed up your stuff to prevent damage.) The tool I personally use is cfdisk, but you can use any other disk utility you want. Run the command,  
`cfdisk`

and a menu should pop up (navigate this with arrow keys.) Select the layout GPT. In the free space that you have, press ENTER on New and give it around 512 MB (to give this 512 MB type "512M" in the text field, this will be our EFI partition) and press ENTER. After that has been created press the right arrow key until you go to "Type". Press the up arrow key until you find "EFI System" and press enter. Now you have your EFI partition. All we have to do now is make your Swap partition and your root partition.

Highlight the free space, and press ENTER on "New". Give this partition 2 GB, so put 2G. This will be our swap (swap is for when your root filesystem becomes full and it uses the swap as a backup, remember swap has to correspond to your RAM, if you have less RAM put less storage for your swap, I currently have 16GB of RAM.) Highlight the swap you just created and change the type to Linux swap. Now you have your Swap partition.

Lastly, we need to do our filesystem/root filesystem. Highlight on the remaining free space and press ENTER twice if you want to use the remainder of the free space. Now highlight "Write" and type "yes" and press ENTER. This will confirm all your changes **(THIS CANNOT BE UNDONE.)** Afterward, you can press "Q" on your keyboard or highlight "Quit" and press ENTER.

### Formatting the disks
Now you want to format your new partitions. To format your root filesystem partition type the command:  
`mkfs.ext4 /dev/root_partition`  
Where it says "root_partition" above it should be the device name. Go back to cfdisk and look under "Device". Under that, it should say /dev/sdaX (X is a number). Find your root filesystem and remember that. For me, the root filesystem is /dev/sda3 so I would enter the command: `mkfs.ext4 /dev/sda3` and press ENTER.

Now you want to make your swap and turn it on. To make your swap run the command,  
`mkswap /dev/swap_partition`  
The same principles apply here. My swap was on /dev/sda2 so instead of `mkswap /dev/swap_partition` it would be `mkswap /dev/sda2`. You can always refer back to cfdisk if you forget your partition names.  
To turn your swap partition on run the command,  
`swapon /dev/swap_partition`

You get it now right? Instead of "swap_partition" you enter your own partition.

Now we want to make our EFI partition. Run the command:  
`mkfs.fat -F32 /dev/efi_partition`  
Your EFI partition name should go where it says `efi_partition`.

Now mount your root partition to the new filesystem. To do this enter this line into your console,  
`mount /dev/root_partition /mnt`  
Replace `root_partition` with your root partition. Mounting the root partition will allow us to access the partition and install packages.

### Installing essential packages to make our system run
Now we will install the essential packages to actually make our system run! To install these packages run the command,  
`pacstrap /mnt base linux linux-firmware`  
Press ENTER then press Y, ENTER, and let it install!

### Generating the fstab
Generate the fstab file with the following command,  
`genfstab -U /mnt >> /mnt/etc/fstab`  
Don't fright if you don't get an output!

### Entering your new filesystem
Enter the newly created filesystem by running,  
`arch-chroot /mnt`  
This should now divert you to your new root directory!

# Seting up your new operating system
### Setting up your system clock
Time to set up your timezone for your system clock! To do this simply type:  
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`  
But replace "Region" and "City" with the options they provided. Press TAB on your keyboard when you get to `/usr/share/zoneinfo/` to see all the options. *Example:`ln -sf /usr/share/zoneinfo/US/Eastern /etc/localtime`*

Now sync the hardware clock by doing  
`hwclock --systohc`

### Installing a console text editor
Now it is time to install your favorite console text editor! In my case I will install nano, but you can obviously install another text editor like, vi, vim, or even neovim. The world is your oyster I'm just living in it!  
To install nano, run:  
`pacman -Sy nano`  
This shall sync your repositories (repo for short) and will install nano the text editor!

### Configuring your locale
We are going to configure our locale. The locale is to define your reigon and language. For my usage I am an English speakers so I will be uncommenting the `en_US.UTF-8 UTF-8`.  
To edit your config file, enter this:  
`nano /etc/locale.gen`  
This will open up the locale.gen file in nano (the text editor you just installed).

Scroll down with your arrow key until you go pass the first instance of `en_US.UTF-8 UTF-8`. Next press Control + W at the same time to open up a search tool. Enter in `en_US UTF-8 UTF-8`, in should scroll all the way down to that line. In front of that line there will be a "`#`" symbol. Now you want to uncomment that by removing the "`#`" symbol.  
*Before:*  
`#en_US.UTF-8 UTF-8`   
*After:*  
`en_US.UTF-8 UTF-8`  
Now write the file and quit. To do this press Control + X then press Y and then ENTER.

Now it is time to generate the new locale. Type the command into the terminal,  
`locale-gen`  
and it should regenerate the locale, if there are any errors it should say so, but if it doesn't you should see the output `Generation complete.`

Now make a new locale configuration file. To do that type:  
`nano /etc/locale.conf` (this will be a new file.)  
In this new file type the locale you just uncommented. When you type this remember to add `LANG=` before you type your locale. The file should be blank because it is a new file.  
*Example:*  
`LANG=en_US.UTF-8`  
Write the file out and quit/exit.

### Configure your keyboard layout
For non-QWERTY(US) users out there you will need to refer to the Arch Wiki and change your keyboard layout. I am using QWERTY right now so this does not apply to me. However if this does apply to you here is the Arch Wiki going over this [here](https://wiki.archlinux.org/index.php/Installation_guide#Localization).

### Create your host name and host domain
Creating your host name and host domain is fairly easy. Keep in mind that you can make your host name anything! In my case I will call my system "arch", but you can name it anything you want! Your host name will appear on your local network!

To edit this file, open up the hostname file with your text editor.  
`nano /etc/hostname`  
Once you have that opened up it is time to create the host name. The file should be blank so simply type in your name of choice into the first line of the document. Now you can write out the file and quit nano.

Now we want to configure the host domain.  
To do this enter the file hosts with your text editor.  
`nano /etc/hosts`  
You should see some stuff already in here. Just go to the end and press ENTER. Now you can setup your host domain like so...  
*Before:*  
```
# Static table lookup for hostnames.  
# See hosts(5) for details.  

```
*After:*  
```
# Static table lookup for hostnames.
# See hosts(5) for details.  

127.0.0.1   localhost
::1         localhost
127.0.1.1   your-hostname.localdomain   your-hostname
```
Where it says `your-hostname` put the host name you chose in `/etc/hosts`. For `127.0.0.1` you can leave it as is or you can change it if you want a specific IP address linked to your domain. Now you can write the file and quit.

### Setting up an internet connection
*Disclaimer: I am using an wired internet connection, if you are using a wireless one please refer back to the Arch Wiki [here](https://www.wiki.archlinux.org/).*

For this we will be getting a few packages to make our network actually run in our new system. The network manager that we will use in this guide is netctl. You can of course change this to any network manager you want to use, but for this instance I will be using netctl.

Install the netctl package by running:  
`pacman -Sy netctl`  
Press 1 for default, and press Y to install. This should install the first package netctl.  
There are optional depencies for this network manager and some mandatory once. Will walk over the ones you really need.

To install the depencies enter in,  
`pacman -Sy dialog dhcpcd wpa_supplicant ifplugd`  
Press ENTER and install these packages.

We are not quite done yet. We will continue this after we reboot back into the system, but before we do that we have some more things to do.

### Adding a user
To add a user run the command,  
`useradd -G wheel,audio,video -m username`  
Replace `username` with the name you want for your new user. This will create the new user and its home directory.

To setup a password for the new user, type:  
`passwd username`  
then enter a password of your choice and press ENTER, then confirm your password. This should update your password.

Now make sure your home directory for the user exists. To check if you have the directory run,  
`ls /home/`  
then press TAB and if your username appears then it is there! Now you can delete the line.

Now set a password for your root user by doing,  
`passwd`  
and enter in your password for your root.

### Install a bootloader
There are many bootloaders out there. There are things like Clover, Syslinux, EFISTUB, but in this case I will use GRUB. Some people say GRUB is slow, but it's really not that slow compared to the others, and on the plus side it support practically any BIOS and UEFI firmware out there.

To install GRUB get these packages by running the command:  
`pacman -S grub efibootmgr`  
and this should install everything you need for GRUB to run!

You may also want to install the OS prober package if you are dual booting, but I will always get this package just incase. Run the command,  
`pacman -S os-prober`  
This should install without any issues.

Now mount your EFI partition so we can install your bootloader. Run the command:  
`mkdir /mnt/efi` (to make your directory)  
`mount /dev/efi_partition /mnt/efi` (to mount your EFI to the newly-created directory.)  
Replace `efi_partition` with your actual EFI partition.

Time to install GRUB to your /efi/ directory. Run this in your terminal,  
`grub-install --target=x86_64-efi --efi-directory=/mnt/efi/ --bootloader-id=GRUB`
 
Now we will create the GRUB configuration file. To do this run,  
`grub-mkconfig -o /boot/grub/grub.cfg`  
and it should output somethings and the config should be made!

### Enter your new system!
After that you should be able to reboot and enter your new computer. To do this type the commands,  
`exit`(this will exit the USB installation)  
`reboot` (this will reboot the system)  
and the computer should reboot into the new operating system!

## Final Touches
Once you boot into your new system type in your username you set and the password you provided it. Right now you will not have an internet connection because you have not yet configured your network manager. To configure this login as root. To login as root type this command in the console.  
`su root`  
Type in your password for root and now you should be logged in as root!

To actually configure the network manager *(for this guide I will be using ethernet)* you need to run this command to change your command directory.  
`cd /etc/netctl`  
Then you can type `ls` to list all the contents of the folder.  
To configure the network manager type:  
`cp examples/ethernet-dhcp ./custom-dhcp-profile`  
You can chose any other file in `cp examples/` by pressing TAB, you may want to choose the one with wireless connection if you are using WiFi, but in this case I am using ethernet so I will be using the ethernet-dhcp. You can change `custom-dhcp-profile` to anything you want!  
Now run:  
`ls`  
And you should see your new custom-dhcp-profile (or what ever you named your profile).

Now you want to open the file up with nano. So do:
`nano custom-dhcp-profile`(replace `custom-dhcp-profile` with your profile)  
And the file should now open up. Now you want to find where it says `Interface=eth0` this is not correct and you will need to change this. To find out the correct interface exit out of nano and do:  
`ip link`  
You should see a list with 2 entries. The second entry name is the one you want.  
*Example:*  
```
1: XX
2: XXXXXX
```  
You want to get the 2nd one and remember that name. Enter back into your `custom-dhcp-profile` file and change `Interface=eth0` to the correct one.  
*Before:*  
`Interface=eth0`  
*After:*  
`Interface=XXXXXX`  
Then you want to uncomment the line that says:  
`DHCPClient=dhcpcd`  
Here is an example:  
*Before:*  
`#DHCPClient=dhcpcd`  
*After:*  
`DHCPClient=dhcpcd`  
Now write and quit the file.

Now you want to enable the profile by doing:  
`netctl enable custom-dhcp-profile`  
Press ENTER and it should generate a new profile and start using that custom-dhcp-profile you just made.

Now to enable this on system startup type:  
`systemctl enable dhcpcd.service`  
And it should start on boot everytime!

To confirm these changes reboot the system one more time by typing in:  
`reboot`  
Then your computer will reboot!

Now ping a server again to check if you have internet by doing:  
`ping archlinux.org`  
If you get returns back from the server that means that you have succesfully done it and there are no problems!

### Finale
Congratulations, you have just installed Arch Linux! I hope this gudie was simple to follow, if you have any issues please leave it on the issues tab on the GitHub. Right now you will not have a DE (Desktop Environment) to work in and I will show you how to install each one right now! There are also WM (Window Managers), but I will not begin into that yet.

## Installing a Desktop Environment
There are many desktop environments out there! There are ones like Cinnamon, Enlightenment, GNOME, KDE, LXDE, MATE, Xfce. I will teach you how to install each and every one of them. KDE is the most like Windows so if you are coming from Windows this is a great option! KDE is also now the lightest desktop environment! GNOME is the most Mac like desktop environment so if you are coming from Mac GNOME is a awesome choice. GNOME is heavier than KDE so keep mind of that.

### Installing KDE
To install KDE it is really simple! Run the following commands in your terminal!  
`pacman -Syu` (this updates the system)  
`pacman -S xorg` (this installs vital programs that you will need for your DE)  
`pacman -S plasma-meta kde-applications` (for KDE applications you are able to select all the applications or only some, I recommend getting all of it but if you don't want that get the packages you need such as a terminal emulator and a file manager. If you need that install Konsole which is KDE's terminal emulator and Dolphin which is the file manager. If you are new to Arch or even KDE for that matter get all the applications!)  
`systemctl enable sddm`  
`systemctl enable NetworkManager`  
That's it! You've installed a DE, now reboot your PC and the new DE will appear!

### Installing Cinnamon
Installing Cinnamon is as easy as installing KDE!  
`pacman -Syu`  
`pamcan -S xorg`  
`pacman -S cinnamon nemo-filereoller`  
`pacman -S gdm`  
`systemctl enable gdm`  
`systemctl start gdm`  
`systemctl enable NetworkManager`  
Now reboot your computer and you will get into the Cinnamon desktop environment!

### Installing Enlightenment
`curl https://download.enlightenment.org/distors/arch/archlinux/arch/repo.txt -o - | sudo tee -a /etc/pacman.conf`  
`pacman -Sy && sudo pacman -S efl-git enlightenment-git terminology-git rage-git`  
`pacman -S lightdm`  
`pacman -S lightdm-gtk-greeter`  
`systemctl enable lightdm`  
Now you can reboot into your new desktop environment!

### Installing GNOME
`pacman -Syu`  
`pacman -S xorg`  
`pacman -S gnome`  
`systemctl start gdm`  
`systemctl enable gdm`  
Now reboot into your new system!

### Installing LXDE
`pacman -Syu`  
`pacman -S xorg`  
`pacman -S lxdm`  
`systemctl start lxdm`  
`systemctl enable lxdm`  
Now you have installed LXDE! Reboot and you will enter the desktop environment.

### Installing MATE
`pacman -Syu`  
`pacman -S xorg`  
`pacman -S mate mate-extra`  
`pacman -S lightdm`  
`pacman -S lightdm-gtk-greeter`  
`systemctl enable lightdm`  
Now reboot into your new environment!

### Installing Xfce
`pacman -Syu`  
`pacman -S xfce4 xfce4-goodies`  
Now reboot into Xfce and you should be good to go!

# Conclusion
Thank you for reading and follow this guide! If this helped you I am happy to help! If anything is outdated in the guide please let me know in our issue tab on this GitHub repo! Again here are all the links for Arch Wiki and balenaetcher.  
Arch Wiki: https://wiki.archlinux.org/  
balenaetcher: https://www.balena.io/etcher/
