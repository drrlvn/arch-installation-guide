# Arch Linux installation guide

So you want to install Arch Linux but got lost in the Wiki. This is a quick guide for installing a
basic system. It is far from being feature-complete, as it makes some basic assumptions.

## Assumptions
1. You want to use the entire disk for Arch Linux without dual-booting another operating system.
1. You use either a physical or a virtual x86 64 bit machine which is capable booting in EFI
   mode. Note that if you use a virtual machine then you'll probably have to check the EFI option somewhere.
1. You use an Intel CPU.
1. Your desired root filesystem is BTRFS.

## The steps

1. Boot into Arch Linux live installation media. Make sure that you know which disk is used for your
   installation. We'll assume it's `/dev/sda`.  You can use `blkid` to list block devices.
1. If you're using Wifi, launch `wifi-menu`. You're network should be automatically configured if
   you're using wired network.
1. *Optional* - You can launch an SSH server and continue your installation remotely from another
computer. In order to do that:
    1. Set a root password using `passwd`
    1. Start the SSH server using `systemctl start sshd`
    1. Figure out your IP using `ip a`
    1. SSH to your installation disk from another computer and continue the installation as usual.
1. Partition your disk:
   1. Run Use `cfdisk` for partitioning:
   1. Choose GPT partitioning (if you don't get the option to choose ,please run `cfdisk -z`)
   1. Create a 512MiB partition. Set its type to `EFI System`
   1. Create a swap partition. 4GiB will probably do. Set its type to `Linux Swap`
   1. Create a partition for the rest of the drive.
   1. `mkswap /dev/sda2`
   1. `swapon /dev/sda2`
   1. `mkfs.vfat -F32 /dev/sda1`
   1. `mkfs.btrfs /dev/sda3`
   1. `mount /dev/sda3 /mnt`
   1. `mkdir /mnt/boot`
   1. `mount /dev/sda1 /mnt/boot`
1. `pacstrap /mnt base intel-ucode sudo btrfs-progs`
1. `genfstab -U /mnt >> /mnt/etc/fstab`
1. `arch-chroot /mnt`
1. `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` (you can see all the options in [wiki timezones list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones))
1. Uncomment `en_US.UTF-8`and other needed localizations in `/etc/locale.gen`
1. `locale-gen`
1. Edit `/etc/locale.conf` and write `LANG=en_US.UTF-8`
1. Networking - Use [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager)
   1. `pacman -S networkmanager`
   1. `systemctl enable NetworkManager`
   1. Once you have a GUI environment set up - configure the network using the GUI
1. `echo [YOUR HOSTNAME] > /etc/hostname`
1. `passwd` - Set the root password
1. `useradd -m <your_username>`
1. `usermod -G wheel -a <your_username>`
1. `passwd <your_username>` - Set the user password
1. `EDITOR=vi visudo` - Comment out the line containing the `wheel` group
1. Install the bootloader - [systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot)
    1. `bootctl --path=/boot install`
    1. Edit `/etc/pacman.d/hooks/systemd-boot.hook`:
       ```
       [Trigger]
       Type = Package
       Operation = Upgrade
       Target = systemd

       [Action]
       Description = Updating systemd-boot...
       When = PostTransaction
       Exec = /usr/bin/bootctl update
       ```
    1. Edit `/boot/loader/loader.conf`:
       ```
       default  arch
       timeout  4
       ```
    1. Figure out your root UUID By running `blkid`
    1. Create `/boot/loader/entries/arch.conf`
       ```
       title          Arch Linux
       linux          /vmlinuz-linux
       initrd         /intel-ucode.img
       initrd         /initramfs-linux.img
       options        root=PARTUUID=THE-UUID-YOU-FOUND-OUT rw
       ```
1. Leave chroot - `exit`
1. If this is a server installation you might want to enable SSH before rebooting. See the
   instructions at the bottom.
1. Reboot - `systemctl reboot`
1. If you use `systemd-networkd`:

    `rm /etc/resolv.conf ; ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf`

## Extras
### GNOME
```pacman -S gnome && systemctl enable --now gdm```

### KDE
```pacman -S sddm plasma kdebase kdeutils kdegraphics && systemctl enable --now sddm```

### Setting up an SSH server
```pacman -S openssh && systemctl enable --now sshd.socket```

### Yay
Building packages from AUR isn't possible to do as root. In order to install Yay you have to
configure sudo and run these commands as a regular user.

1. `sudo pacman -S --needed base-devel git`
1. `cd /tmp`
1. `git clone https://aur.archlinux.org/yay-bin.git && cd yay-bin && makepkg -i && cd - && rm -rf yay-bin`
