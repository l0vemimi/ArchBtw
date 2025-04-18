This guide assumes you're already familiar with arch or linux and should be used as a refresher for reinstallation.

There are **two different** guide modes: **regular** and **veteran**:

**Reg**: Good brief guide on installation with reference links and a bit of explanation.

**Vet**: Reminder and quick reference guide; Straight to the point and no screen bloat.

If you're familiar with linux / arch, skip to the [regular install guide](#regular).

If you're an arch vet, skip to the [vet install guide](#veteran).

**Linux noobs, [read below](#linux-noobs) before progressing.**


### Linux Noobs

I really do not recommend installing on dual boot with win***s, (do this at your own risk) nor do I recommend installing arch if you're a noob to linux. No hate against noobs but it get's annoying seeing the same stupid questions getting asked daily on [arch forums](https://bbs.archlinux.org/), [r/arch](https://www.reddit.com/r/arch/) and [r/archlinux](https://www.reddit.com/r/archlinux/).

If you want an introduction to linux, I recommend using [debian](https://www.debian.org/) or [ubuntu](https://ubuntu.com/download) to familiarise yourself, if you're hard set on arch try an arch distro such as [manjaro](https://manjaro.org/), [garuda](https://garudalinux.org/) or [endeavor](https://endeavouros.com/).

If you're still determined to download vanilla arch, I applaud and wish you luck. This guide will not go through [archinstall]() and I advise against it; archinstall is meant to be used for users who already know what they are doing and manually installing is the **best** way to learn about the system. You will learn about very critical commands that will need to be used when repairing your system such as [chroot](https://wiki.archlinux.org/title/Chroot) or [pacstrap](https://wiki.archlinux.org/title/Pacstrap).

For the love of god please [RTFW](https://wiki.archlinux.org/title/Main_page) and use it as your reference guide. The arch wiki will become your bible and it has EVERYTHING you need.

#### 

---

# Veteran

1. partition
2. mkfs
3. mount
4. pacstrap
5. fstab
6. chroot
7. grub-install & grub-mkconfig
8. useradd & usermod

---

# Regular

## 1. Preperation

### 1.1. Connect the USB

Connect the USB to your computer and boot into the iso. This shouldn't need much explanation...

### 1.2. Verify Boot Mode

    user@arch ~ # cat /sys/firmware/efi/fw_platform_size

It should return: **64** meaning *64 bit* or **32** meaning *32 bit*. 

If it returns: **No such file or directory**, you've got a **bios system**.

Remember this as this will be important when [partitioning](#4-partition) the disks.

## 2. Connect to the Internet

This will go through **two** ways to connect: [iwd](#21-connect-with-iwctl) and [ethernet](#22-connect-to-the-internet-via-ethernet); Then when connected, [ping](#23-confirm-internet-connection) to confirm the connection.

### 2.1. Connect with iwctl

Use [iwd](https://wiki.archlinux.org/title/Iwd#iwctl) to connect to the internet. Type in iwctl.

    user@arch ~ # 
    
    user@arch ~ # iwctl

The prompt will change to iwd.List the station then put it in scan mode.

    [iwd]# 
    
    [iwd]# station list
                            Devices in Station Mode

        Name            State           Scanning
                            
        wlan0           disconnected


    [iwd]# station wlan0 scan

Once you've got the station name; **wlan0** in this case, list the available networks.

    [iwd]# station wlan0 get-networks
                            Available networks
                            
        Network name       Security        Signal

        WIFI1              psk             ****
        WIFI2              psk              ***
        WIFI3              psk              ****      
    
After you get your network, connect to it and type in the password.

    [iwd]# station wlan0 connect WIFI1
    Type the network passphrase for ARCH5G psk.
    Passphrase...
If you have a hidden network, connect to it:

    [iwd]# station wlan0 connect-hidden [NETWORK_NAME]

### 2.2. Connect to the Internet via Ethernet

Connecting via ethernet should be automatic; you shouldn't need to configure anything on arch. Find the setting in your android phone:

**Settings > Connections > Mobile Hotspot and Teathering > USB Tethering**

***NOTE:*** *The USB cable needs to be connected for this option to be toggled. If you can't find the setting, look up ethernet connection for your specific device model*.

### 2.3. Confirm Internet Connection

Ping the network to make sure you're connected. You can use any address you want, e.g *google's public DNS: 8.8.8.8; cloudfare's public DNS: 1.1.1.1*. I usually use archlinux, so we'll use it in the example below.



    user@arch ~ # ping archlinux.org


## 4. Partition

List the disks on your system:

    user@arch ~ # lsblk
    NAME        MAJ:MIN     RO      SIZE    TYPE    MOUNTPOINTS
    nvme0n1     259:0       0       1.8T    disk

**Your disks can also be named sda**; follow what is relevant. Anything that ends in *rom, loop, airootfs* can be ignored. See [device file](https://wiki.archlinux.org/title/Device_file#Block_device_names) for more information.

The following will be going through GPT partitions, for more information on partitioning, view: [Partition Types](https://en.wikipedia.org/wiki/Partition_type) and GNU's [bios installation](https://www.gnu.org/software/grub/manual/grub/html_node/BIOS-installation.html#BIOS-installation)

### 4.1. Layouts

**EFI:**

|    Partition  | Partition Type    | Recommended Size |
| ------------- | ----------------- | ---------------- |
| /boot         | EFI System        | 1 GiB            |
| [SWAP]        | Linux swap        | 4 GiB            |
| /             | Linux x86_64 root | 23-32 GiB        |

**BIOS:**

| Partition | Partition Type    | Recommended Size |
| --------- | ----------------- | ---------------- |
|     -     | BIOS Boot         | 1 MiB            |
| [SWAP]    | Linux swap        | 4 GiB            |
| /         | Linux x86_64 root | 23-32 GiB        |

**NOTE:** 

*Home partition isnt required but it is usually created; especially if you'll be using your pc as a daily and:*

- *So you're not sharing the same userspace as root.* 
- *Easier to distro hop or nuke your install to start over without wiping user configs.*

***Boot*** *of 1 GiB is enough for multiple kernels; if you don't plan on using more kernels on /boot, then 400 MiB should be enough. I usually just go for 1 GiB* ***just in case***

### 4.2. Partition Disk

    user@arch ~ # fdisk /dev/sda

Create a new GPT partition.

    Command (m for help): g

Create boot, swap, root and home. 

    Command (m for help): n

    Partition number (1-4, default 1): [ENTER]

    First sector (2048-3907026943, default 2048): [ENTER]

    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-3907026943, default 3907026943): +1G

Do not type +1G for all partitions... make sure the size is relevant to the partition type. Review [4.1. Layouts](#41-layouts).

**BIOS does not need /boot**, *do not create it.You'll need an extra partition called **bios boot**, with no file system and +1M ~ it can be in any position but needs to be on the first 2 TiB of the disk. This partition also needs to be created **BEFORE** grub is installed.*

Change the partition types.

    Command (m for help): t

Quick partition type reference guide for GPT:

| Alias | Type                 | [GUID](https://wiki.archlinux.org/title/GUID_Partition_Table) |
| ----- | -------------------- | ------------------------------------------------------------- |
| 1     | EFI System           | C12A7328-F81F-11D2-BA4B-00A0C93EC93B                          |
| 4     | BIOS Boot            | 21686148-6449-6E6F-744E-656564454649                          |
| 19    | Linux Swap           | 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F                          |
| 20    | Linux Filesystem     | 0FC63DAF-8483-4772-8E79-3D69D8477DE4                          |
| 23    | Linux Root x86_64    | 4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709                          |
| 42    | Linux Home           | 933AC7E1-2EB4-4F13-B844-0E14E2AEF915                          |

When finished, write the disk and exit.

    Command (m for help): w

### 4.3. Format Partition

For boot partition, create the efi file system. 

    user@arch ~ # mkfs.fat -F 32 /dev/[EFI PARTITION]

Create the swap file system.

    user@arch ~ # mkswap /dev/[SWAP_PARTITION]

Create a filesystem for root and home if you've created one. I always use ext4, view [file systems](https://wiki.archlinux.org/title/File_systems) for more information on different types.

    user@arch ~ # mkfs.ext4 /dev/[ROOT_PARTITION]

    user@arch ~ # mkfs.ext4 /dev/[HOME_PARTITION]

## 5. Mount

Mount the partitions you've created.

    user@arch ~ # mount /dev/[ROOT_PARTITION] /mnt

You'll need to run mkdir with other mount points.

    user@arch ~ # mount --mkdir /dev/[BOOT_PARTITION] /mnt/boot

    user@arch ~ # mount --mkdir /dev/[HOME_PARTITION] /mnt/home

**NOTE: Do not mount BIOS boot** partition.

Turn on swap.

    user@arch ~ # swapon /dev/[SWAP_PARTITION]

## 6. Pacstrap

Install the base packages, kernel and firmware. See [kernel](https://wiki.archlinux.org/title/Kernel) for more information on different kernel types and options.

    user@arch ~ # pacstrap -K /mnt base linux linux-firmware

It's best to download the rest of your packages now, you could also do this when you've chrooted.

***NOTE:*** *Nothing except for /etc/pacman.d/mirrorlist will be carried over from the live install to the installed system.*

### 6.1. Things to Consider

**[Boot Loader](https://wiki.archlinux.org/title/Arch_boot_process#Feature_comparison)**

- [efibootmgr](https://archlinux.org/packages/core/x86_64/efibootmgr/)
- [grub](https://archlinux.org/packages/core/x86_64/grub/)
- [grub-legacy](https://aur.archlinux.org/packages/grub-legacy/)

**CPU [Microdes](https://wiki.archlinux.org/title/Microcode):** 

- [amd-ucode](https://archlinux.org/packages/?name=amd-ucode)
- [intel-ucode](https://archlinux.org/packages/?name=intel-ucode)

**[Desktop Environments](https://wiki.archlinux.org/title/Desktop_environment):**

- [KDE Plasma](https://archlinux.org/packages/extra/x86_64/plasma-desktop/)
- [gnome](https://archlinux.org/packages/extra/x86_64/gnome-desktop/)

**[Display Manager](https://wiki.archlinux.org/title/Display_manager):**

-[gdm](https://archlinux.org/packages/?name=gdm)
- [lightdm](https://archlinux.org/packages/?name=lightdm)
- [lxdm](https://archlinux.org/packages/?name=lxdm)
- [SDDM](https://archlinux.org/packages/extra/x86_64/sddm/)

**Firmware:**

- [broadcom wireless](https://wiki.archlinux.org/title/Broadcom_wireless)

**[Networking](https://wiki.archlinux.org/title/Network_configuration#Network_managers):**

- [dhcpcd](https://archlinux.org/packages/extra/x86_64/dhcpcd/)
- [iwd](https://archlinux.org/packages/extra/x86_64/iwd/)
- [ModemManager](https://archlinux.org/packages/extra/x86_64/modemmanager/)
- [NetworkManager](https://archlinux.org/packages/extra/x86_64/networkmanager/)
- [wpa_supplicant](https://archlinux.org/packages/core/x86_64/wpa_supplicant/)

**Text Editors:**

- [nano](https://archlinux.org/packages/core/x86_64/nano/)
- [vim](https://archlinux.org/packages/extra/x86_64/vim/)

**[Terminal Emulators](https://wiki.archlinux.org/title/List_of_applications/Utilities#Terminal_emulators):**

- [alacritty](https://archlinux.org/packages/?name=alacritty)
- [kitty](https://archlinux.org/packages/?name=kitty)
- [konsole](https://archlinux.org/packages/?name=konsole)
- [wezterm](https://archlinux.org/packages/?name=wezterm)

## 7. Change Root

### 7.1. Pre-Chroot

Before you chroot you must generate [fstab](https://wiki.archlinux.org/title/Fstab).

    user@arch ~ # genfstab -U /mnt >> /mnt/etc/fstab

### 7.2. Chroot

[Chroot](https://wiki.archlinux.org/title/Change_root) into the system at your chosen mountpoint; it should be /mnt for a general new install if you followed this guide or [archwiki](https://wiki.archlinux.org/title/Installation_guide).

    user@arch ~ # arch-chroot /mnt    

### 7.3. Post-Chroot

### 7.3.1. Bootloader

This will run through installing [grub](https://wiki.archlinux.org/title/GRUB#Generate_the_main_configuration_file), for other bootloader options see [boot loader comparison](https://wiki.archlinux.org/title/Arch_boot_process#Feature_comparison).

EFI:

    chroot@arch ~ # grub-install --target=x86_64-efi --efi-directory=boot --boot-loader-id=GRUB

BIOS:

    grub-install --target=i386-pc /dev/sdx

/dev/sdx is the **disk not the partition** for grub to be installed e.g. /dev/sda /dev/nvme1n1 are **disks**; /dev/sda1 /dev/nvme1n1p1 are **partitions**.

Generate grub config. The file for both EFI and BIOS will be generated in /boot/grub/grub.cfg.

    chroot@arch ~ # grub-mkconfig -o /boot/grub/grub.cfg

### 7.3.2. Initramfs

[Mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio) was run during the pacstrap; To create a new initramfs for any reason:

    chroot@arch ~ # mkinitcpio -P

### 7.3.3. Network

Create a [hostname](https://wiki.archlinux.org/title/Hostname) file in /etc; I use nano.

    chroot@arch ~ # nano /etc/hostname

    yourhostname

### 7.3.4. Time & Localisation

Set the time zone:

    chroot@arch ~ # ln -sf /usr/share/zoneinfo/Region/City /etc/localtime

    chroot@arch ~ # hwclock --systohc

Set the locale:

    chroot@arch ~ # locale-gen

Set the LANG in /etc/locale.conf:

    chroot@arch ~ # LANG=en_US.UTF-8

### 7.3.5. Users & Login

Create the root password, you will need this when you reboot and you will not be able to log in without it. If you skip this step, you'll need to chroot back in and add the password.

    chroot@arch ~ # passwd

Create a user; You will need to create a user if you want to use a desktop environment; You will not be able to login with root.

    chroot@arch ~ # useradd -m -s /usr/bin/bash [USERNAME]

**-m / --create-home**: User home dir is created as /home/[USERNAME].

**-s / --shell**: Path to the login shell. Make sure you've got the shell you want to use installed...

Add a password for the user account you've just created.

    chroot@arch ~ # [USERNAME] passwd

## 8. Reboot

**CTRL + D** to exit chroot, then type reboot.

    user@arch ~ # reboot










