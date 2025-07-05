# Arch Linux Manual Installation Guide Me Version

Sometimes i forgot some step, this is just for a reminder or maybe someone also looking for this guide (which is unlikely since there is a lot of guide on internet).

## 1. Connect to the internet

Since most of the time im installing arch linux on a laptop with wifi connection, this is how to connect to available network (you dont need this if you have ethernet)

```bash
#entering wifi setup
iwctl

#list available network interface device (wifi usually wlan0)
device list

#scan nearby network
station interface-name scan

#list available network
station interface-name get-networks

#connect to the desired network
station interface-name connect "network-name"

then exit
```

### 1.1 Remote Installation

This is step by step to setup ssh for remote Installation

```bash
#setup root password
passwd

#start ssh service
systemctl start sshd

#check the ip under wifi interface
ip a
```

## 2. Partitioning

For partitioning im using cfdisk, althought arch wiki recommended to use fdisk, im personally more comfortable with cfdisk

```bash
#enter cfdisk menu
cfdisk
```
then make this 3 partition
- EFI (100MB)
- Swap (optional)
- Root (The rest of the partition)

### 2.1 Partition Formatting

Check if partition is correct
```bash
#list all partition
lsblk
```
Note:
- for SATA SSD it usually /dev/sda
- for NVME SSD it usually /dev/np0s01

```bash
#format root partition
mkfs.ext4 /your-root-partition

#format efi partition
mkfs.fat -F 32 /your-efi-partition

#format swap
mkswap /your-swap-partition
```

### 2.2 Partition Mounting

```bash
#mount root partition
mount /your-root-partition /mnt

#mount efi partition
mkdir -p /mnt/boot/efi
mount /your-efi-partition /mnt/boot/efi

#turn on swap partition
swapon
```

## 3. Installing Important Packages

Arch wiki recommended to intall base, linux and linux-firmware. But for me thats not enough for post installation, so i want to add several packages:
- git
- base-devel
- networkmanager
- grub
- efibootmgr
- nano
- sof-firmware (for newer soundcard)
- openssh

```bash
pacstrap /mnt base linux linux-firmware git base-devel networkmanager grub efibootmgr nano sof-firmware openssh
```

### 4. Generating Filesystem Tab

```bash
genfstab /mnt > /mnt/etc/fstab
```

Note:
This is for single boot linux, idk about dual booting


### 5. Configuring The System
After finished installing all the packages, now we enter the system to configure some setting

```bash
arch-chroot /mnt

#setting timezone
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime

#sync time with device
hwclock --systohc
```

### 5.1 Configuring localtime

```bash
nano /etc/locale.gen
```

then uncomment en_US.UTF-8

```bash
locale-gen

#declaring default locale
nano /etc/locale.conf
```
then add LANG=en_US.UTF-8

## 6. Finale Step

```bash
#set hostname
nano /etc/hostname

#set root password
passwd

#add user
useradd -m -G wheel -s /bin/bash username

#set new user password
passwd username
```

### 6.1 Setup Sudo

```bash
EDITOR=nano visudo
```
uncomment %wheel ALL=(ALL:ALL) ALL

### 6.2 Enabling Core Services

```bash
#enable networkmanager
systemctl enable NetworkManager

#enable ssh
systemctl enable sshd
```

### 6.3 Configuring Bootloader

```bash
grub-install /storage-type *(/dev/sda or /dev/np0s0)
grub-mkconfig -o /boot/grub/grub.cfg

exit
```
```bash
#unmount all partition
umont -a

reboot
```
