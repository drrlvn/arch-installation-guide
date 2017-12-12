# Arch Linux installation guide

So you want to install Arch Linux but got lost in the Wiki. This is a quick guide for installing a
basic system. It is far from being feature-complete, as it makes some basic assumptions, such as
that you don't install Arch Linux side by side with another operating system, and that you wish to
use BTRFS as your root partition. This guide also assumes that you have an Intel CPU.

## The steps

1. Boot into Arch Linux live installation media. Make sure that you know which disk is used for your
   installation. We'll assume it's `/dev/sda`.
1. Partition your disk:
   1. BIOS - Just `mkfs.btrfs /dev/sda` the entire disk and then `mount /dev/sda /mnt`
   1. EFI - Use `cfdisk` for partitioning:
      1. Choose GPT partitioning
      1. Create a 512MiB partition. Set its type to `EFI System`
      1. Create a partition for the rest of the drive.
      1. `mkfs.vfat -F32 /dev/sda1`
      1. `mkfs.btrfs /dev/sda2`
      1. `mount /dev/sda2 /mnt`
      1. `mkdir /mnt/boot`
      1. `mount /dev/sda1 /mnt/boot`
1. `pacstrap /mnt base intel-ucode`
1. `genfstab -U /mnt >> /mnt/etc/fstab`
1. `arch-chroot /mnt`
1. `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
1. Uncomment `en_US.UTF-8`and other needed localizations in `/etc/locale.gen`
1. `locale-gen`
1. Edit `/etc/locale.conf` and write `LANG=en_US.UTF-8`
1. `echo hostname > /etc/hostname`
1. `passwd`
1. Install the bootloader
    1. BIOS - GRUB
       1. `pacman -S grub`
       1. `grub-install --target=i386-pc /dev/sda`
       1. `grub-mkconfig -o /boot/grub/grub.cfg`
    1. EFI - systemd-boot
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
1. Reboot

## Extras
### GNOME
`pacman -S gnome && exec systemctl enable --now gdm`
### Pacaur
TODO
