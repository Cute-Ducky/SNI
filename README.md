# SNI
![SNI](https://user-images.githubusercontent.com/105547581/180649454-81f27742-a4cb-4320-8757-d7969c27de3a.png)

<h1 align="center">Simple NixOS Installation! (With ZFS!)</h1>
<p>
</p>

***Follow the guid completely and carefully for optimal results.***

Partitioning and ZFS
--------------------
To format your drive as ZFS first run `lsblk` and remember the name of the drive that you want to format as ZFS!

In my case It's: /dev/sda 
```
cDucky@snow:~/ > lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 476.9G  0 disk
├─sda1   8:1    0   800M  0 part /boot
└─sda2   8:2    0 476.2G  0 part
```
(Note: Replace `<your drive name>` with the name of the drive that you want to install NixOS on.)

To partition the drive using `gdisk` run these commands: 
```
sudo su
gdisk /dev/<your drive name>
Type o then press enter
And then answer y then press enter
And type q then enter to quit gdisk

cgdisk /dev/<your drive name>
Now delete all partitions
Create a new partition
Use default on the first sector (Press enter without any text)
For size in sectors or {KMGPT} part type +800M and press enter
For hex code or GUID type ef00 and press enter
As of the name use: NIXBOOT

Create a new partition in the bigger unallocated space
Use default when prompted on all options (Press enter without any text)
As of the name use: NIXROOT

Write and Quit
```

Now lets make a fat32 file system for the NIXBOOT partition
```
mkfs.vfat -F 32 /dev/<your drive name>
```
Now lets name the NIXBOOT partition
```
fatlabel /dev/<your drive name> NIXBOOT
```

Create a zpool and encrypt the drive
```
sudo zpool create -f \
-o altroot="/mnt" \
-o ashift=12 \
-o autotrim=on \
-O compression=lz4 \
-O acltype=posixacl \
-O xattr=sa \
-O relatime=on \
-O normalization=formD \
-O dnodesize=auto \
-O sync=disabled \
-O encryption=aes-256-gcm \
-O keylocation=prompt \
-O keyformat=passphrase \
-O mountpoint=none \
NIXROOT \
/dev/<your drive name>

enter your prefered password and DON'T forget it!
(Note: You can't see the password that you enter to the terminal don't worry!)
```
  
Create ZFS partitions with these commands
```
sudo zfs create -o mountpoint=legacy NIXROOT/root
sudo zfs create -o mountpoint=legacy NIXROOT/home
sudo zfs create -o refreservation=1G -o mountpoint=none NIXROOT/reserved
```
  
Mounting the partitions
```
sudo mount -t zfs NIXROOT/root /mnt
sudo mkdir /mnt/boot
sudo mkdir /mnt/home
sudo mount /dev/<your drive name> /mnt/boot
sudo mount -t zfs NIXROOT/home /mnt/home
```
  
Generating the configuration
----------------------------
```
sudo nixos-generate-config --root /mnt
```
Thanks to [@mcdonc](https://github.com/mcdonc) for the ZFS part!


Editing the config file
-----------------------

open the /mnt/etc/nixos/configuration.nix file with your preferred editor.
```
sudo nano /mnt/etc/nixos/configuration.nix
or
sudo vim /mnt/etc/nixos/configuration.nix
```
Remove everything from your config and copy and paste my hole config [linked here](https://github.com/Cute-Ducky/SNI/blob/main/configuration.nix)

Run ` head -c 8 /etc/machine-id ` and replace <your host id> in [line 14](https://github.com/Cute-Ducky/SNI/blob/main/configuration.nix#L14) with the number from the command's output.

Replace <your preferred host name> in [line 26](https://github.com/Cute-Ducky/SNI/blob/main/configuration.nix#L26) with the name that you want to set for your computer. (You can choose anything!)
  
In [line 35](https://github.com/Cute-Ducky/SNI/blob/main/configuration.nix#L35) replace Continent/Country or City with your time zone.
  
At [line 53](https://github.com/Cute-Ducky/SNI/blob/main/configuration.nix#L53) you can choose which Desktop Environment or Window Manager you want. By default I have set i3 and icewm but you can remove them or add other Desktop Environments or Window Managers!
To add a Desktop Environment or Window Manager go to search.nixos.org/options and search your preferred Desktop Environment or Window Manager then copy and paste the one with .enable at the end. After pasting add ` = true;` at the end.
An example for the KDE Desktop Environment:
```
services.xserver.desktopManager.plasma5.enable = true;
```
  
