# Arch Linux installation guide

So you want to install Arch Linux but got lost in the Wiki. This is a quick guide for installing a
basic system. It is far from being feature-complete, as it makes some basic assumptions.

## Assumptions
1. You want to use the entire disk for Arch Linux without dual-booting another operating system.
1. You use either a physical or a virtual x86 64 bit machine which is capable booting in either BIOS or EFI mode.
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
   1. **BIOS** - Just `mkfs.btrfs /dev/sda` the entire disk and then `mount /dev/sda /mnt`
   1. **EFI** - Use `cfdisk` for partitioning:
      1. Choose GPT partitioning
      1. Create a 512MiB partition. Set its type to `EFI System`
      1. Create a partition for the rest of the drive.
      1. `mkfs.vfat -F32 /dev/sda1`
      1. `mkfs.btrfs /dev/sda2`
      1. `mount /dev/sda2 /mnt`
      1. `mkdir /mnt/boot`
      1. `mount /dev/sda1 /mnt/boot`
1. `pacstrap /mnt base intel-ucode sudo btrfs-progs`
1. `genfstab -U /mnt >> /mnt/etc/fstab`
1. `arch-chroot /mnt`
1. `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
1. Uncomment `en_US.UTF-8`and other needed localizations in `/etc/locale.gen`
1. `locale-gen`
1. Edit `/etc/locale.conf` and write `LANG=en_US.UTF-8`
1. Networking
   1. **Laptop** - Use [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager)
      1. `pacman -S networkmanager`
      1. `systemctl enable NetworkManager`
      1. Once you have a GUI environment set up - configure the network using the GUI
   1. **PC/VM** - Use [systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd)
      1. `systemctl enable systemd-{network,resolve}d`
      1. Edit `/etc/systemd/network/dhcp.network`:
          ```
          [Match]
          Name=en*

          [Network]
          DHCP=ipv4
          Domains=extra.domains.that.you.need.example.com

          [DHCP]
          UseDomains=yes
          ```
      1. `rm /etc/resolv.conf ; ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf`
1. `echo <hostname>  > /etc/hostname`
1. `passwd` - Set the root password
1. `useradd -m <your_username>`
1. `usermod -G wheel -a <your_username>`
1. `passwd <your_username>` - Set the user password
1. `EDITOR=vi visudo` - Comment out the line containing the `wheel` group
1. Install the bootloader
    1. **BIOS** - [GRUB](https://wiki.archlinux.org/index.php/GRUB)
       1. `pacman -S grub`
       1. `grub-install --target=i386-pc /dev/sda`
       1. `grub-mkconfig -o /boot/grub/grub.cfg`
    1. **EFI** - [systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot)
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

## Extras
### GNOME
`pacman -S gnome && systemctl enable --now gdm`

### Setting up an SSH server
`pacman -S openssh && systemctl enable --now sshd.socket`

### Yay
Building packages from AUR isn't possible to do as root. In order to install Yay you have to
configure sudo and run these commands as a regular user.

1. `sudo pacman -S --needed base-devel git`
1. `cd /tmp`
1. `git clone https://aur.archlinux.org/yay-bin.git && cd yay-bin && makepkg -i && cd - && rm -rf yay-bin`
