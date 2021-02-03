# Arch Linux Installation
*A simple, quick, and easy Arch Installation guide.*

This is meant to help people who are new to Arch Linux and need a quick installation guide, I know the Arch wiki can be confusing sometimes so this is why I made this! This is also a little note for myself to use for Arch Linux installations. Congratulations on trying to install Arch, don't worry it won't be as hard as you think! **(Note: Please backup everything before you start anything on this guide, you might have a chance of messing up and wiping drives you don't intend to wipe!)**

## Prerequisites (Links are at the end of this section)
First things first, you obviously need a 4-8GB flash drive for your Arch install. Get this useful program called "balenaetcher", that's should help you burn your ISO to the USB drive. After you get that, get the Arch Linux ISO from the mirror list. Chose the one called "archlinux-XXXX.XX.XX-x86_64.iso" (the X's are for the Arch Linux version update date. Fire up balenaetcher and select where you downloaded your Arch Linux ISO file, click Next, and select the drive you want to burn the USB to **(Careful, this will delete everything on your drive so please back it up if you have anything on it).** Continue with the process and it shall burn it to the USB!

*Links:*  
Arch Linux ISO Download: https://archlinux.org/download/  
balenaetcher Donwload (Windows, MacOS, Linux download): https://www.balena.io/etcher/

## Boot from the USB Install
Depending on what motherboard (it is different for every motherboard, please Google your motherboard type and look for its boot menu button), you need to get to your boot menu and select the USB you just created. Select the USB and hit ENTER.

## Installation
###### Verify the boot mode
**For this walkthrough I will be using the Arch Wiki installation guide as a guide for me as well, you can check the installation guide that they have [here](https://wiki.archlinux.org/index.php/Installation_guide).**  
Once you boot into your disk you must wait until you see the command line or something that says "root@archiso ˜ #", that is when you are all set!
If you have a keymap that is not the default (US QWERTY) you will need to load it and apply it, but in my case, I am using QWERTY and do not need to change this, so I will be skipping over this. If you do need to load a keymap please refer to the Arch Wiki above.
Firstly, please verify your boot mode by entering this command.  
`ls /sys/firmware/efi/efivars`  
If you see some kind of output this is good, if you do not see an output this guide isn't for you as you will need some extra steps.  

###### Setting up a network connection
Now we will set up an internet connection. For this guide, I am using ethernet and if you are using WiFi please refer back to the Arch Wiki. Run the command,  
`ip link`

You should see all your internet devices. Next, you want to verify you have an internet connection by running the command,  
`ping archlinux.org`  
If you see stuff like, `64 bytes from archlinux.org etc.` then you are set and you have an internet connection. Press Control + C to quit the ping command.

###### Syncing the system clock
At this point, you want to update the system clock. To do this run the command,  
`timedatectl set-ntp true`  
If you don't see an output you are good, if you see an output wait until you see `[ OK ] Reached target Garphical Interface` and press Control + C to exit that.  

To check if the system clock is succesfully synced type:  
`timedatectl status`  
If you see `System clock synchronized: yes` you are good to go!

###### Partitioning the disks
It is time to partition the disk (you should've already backed up your stuff to prevent damage.) The tool I personally use is cfdisk, but you can use any other disk utility you want. Run the command,  
`cfdisk`

and a menu should pop up (navigate this with arrow keys.) Select the layout GPT. In the free space that you have, press ENTER on New and give it around 512 MB (to give this 512 MB type "512M" in the text field, this will be our EFI partition) and press ENTER. After that has been created press the right arrow key until you go to "Type". Press the up arrow key until you find "EFI System" and press enter. Now you have your EFI partition. All we have to do now is make your Swap partition and your root partition.

Highlight the free space, and press ENTER on "New". Give this partition 2 GB, so put 2G. This will be our swap (swap is for when your root filesystem becomes full and it uses the swap as a backup, remember swap has to correspond to your RAM, if you have less RAM put less storage for your swap, I currently have 16GB of RAM.) Highlight the swap you just created and change the type to Linux swap. Now you have your Swap partition.

Lastly, we need to do our filesystem/root filesystem. Highlight on the remaining free space and press ENTER twice if you want to use the remainder of the free space. Now highlight "Write" and type "yes" and press ENTER. This will confirm all your changes **(THIS CANNOT BE UNDONE.)** Afterward, you can press "Q" on your keyboard or highlight "Quit" and press ENTER.

###### Formatting the disks
Now you want to format your new partitions. To format your root filesystem partition type the command:  
`mkfs.ext4 /dev/root_partition`  
Where it says "root_partition" above it should be the device name. Go back to cfdisk and look under "Device". Under that, it should say /dev/sdaX (X is a number). Find your root filesystem and remember that. For me, the root filesystem is /dev/sda3 so I would enter the command: `mkfs.ext4 /dev/sda3` and press ENTER.

Now you want to make your swap and turn it on. To make your swap run the command,  
`mkswap /dev/swap_partition`  
The same principles apply here. My swap was on /dev/sda2 so instead of `mkswap /dev/swap_partition` it would be `mkswap /dev/sda2`. You can always refer back to cfdisk if you forget your partition names.  
To turn your swap partition on run the command,  
`swapon /dev/swap_partition`

You get it now right? Instead of "swap_partition" you enter your own partition.

Now we want to make or EFI partition. Run the command:  
`mkfs.fat -F32 /dev/efi_partition`  
Your EFI partition name should go where it says `efi_partition`.

Now mount your EFI partition so we can install your bootloader later. Run the command:  
`mkdir /mnt/efi` (to make your directory)  
`mount /dev/efi_partition /mnt/efi` (to mount your EFI to the newly-created directory.)  
Replace `efi_partition` with your actual EFI partition.

Now mount your root partition to the new filesystem. To do this enter this line into your console,  
`mount /dev/root_partition /mnt`  
Replace `root_partition` with your root partition. Mounting the root partition will allow us to access the partition and install packages.

###### Installing essential packages to make our system run
Now we will install the essential packages to actually make our system run! To install these packages run the command,  
`pacstrap /mnt base linux linux-firmware`  
Press ENTER and let it install!

###### Generating the fstab
Generate the fstab file with the following command,  
`genfstab -U /mnt >> /mnt/etc/fstab`  
Don't fright if you don't get an output!

###### Entering your new filesystem
Enter the newly created filesystem by running,  
`arch-chroot /mnt`  
This should now divert you to your new root directory!

###### Setting up your system clock
Time to set up your timezone for your system clock! To do this simply type:  
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`  
But replace "Region" and "City" with the options they provided. Press TAB on your keyboard when you get to `/usr/share/zoneinfo/` to see all the options.  

Now sync the hardware clock by doing  
`hwclock --systohc`

###### Installing a console text editor
Now it is time to install your favorite console text editor! In my case I will install nano, but you can obviously install another text editor like vi, vim, or even neovim. The world is your oyster I'm just living in it!  
To install nano, run:  
`pacman -Sy nano`  
This shall sync your repositories (repo for short) and will install nano text editor!
