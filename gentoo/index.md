[Home](/index.md)

## Gentoo

![Gentoo Site Logo](res/gentoo-site-logo.png)

### Chrooting

Source: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev

mount -o remount,rw /sys/firmware/efi/efivars

test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
chmod 1777 /dev/shm

chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

### Unchrooting

Source: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader

```bash
exit

umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo

```

### Booting from UEFI

```bash
mount -o remount,rw /sys/firmware/efi/efivars
grub-install --target=x86_64-efi --efi-directory=/boot --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

### Enabling AMD Virtualisation VT-x on Gigabyte AORUS 

```
BIOS Setup
--> Tweaker
  --> Advanced Processor Options
    --> SVM Mode (Enabled)
```
