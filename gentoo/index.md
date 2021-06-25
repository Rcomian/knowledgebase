[Home](../index.md)

## Gentoo

![Gentoo Site Logo](res/gentoo-site-logo.png)

### Troubleshooting

[Murmur - Mumble server](mumbleserver.md)

[FileCloud Sync](filecloudsync.md)

[KVM Virtualisation passthrough](virt-passthrough.md)

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

### Mounting a directory over another directory

```bash
mkdir -p /mnt/target
mount --bind /mnt/source /mnt/target
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

### Installing 9999 packages (from git)

Installing keywordless packages, which are normally straight checkouts from git.

If you don't get this right you get an error like:

```bash
- media-video/v4l2loopback-9999::gentoo (masked by: missing keyword)
```

To unmask it properly, use `**` as the keyword, like:

``` bash
echo =media-video/v4l2loopback-9999 \*\* >/etc/portage/package.accept_keywords/v4l2loopback9999
```

You want two asterisks `**` in the kewords file, but to add them from the commandline you must escape them.

### Emoji fonts for gentoo

From: https://www.christitus.com/emoji

```bash
emerge -av noto-emoji
```

`/etc/fonts/conf.d/02-emoji.conf`:

```xml
<?xml version="1.0"?>
  <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
  <fontconfig>

   <alias>
     <family>sans-serif</family>
     <prefer>
       <family>DejaVu Sans</family>
       <family>Noto Color Emoji</family>
       <family>Noto Emoji</family>
     </prefer> 
   </alias>

   <alias>
     <family>serif</family>
     <prefer>
       <family>DejaVu Serif</family>
       <family>Noto Color Emoji</family>
       <family>Noto Emoji</family>
     </prefer>
   </alias>

   <alias>
    <family>monospace</family>
    <prefer>
      <family>Nimbus Mono L</family>
      <family>Noto Color Emoji</family>
      <family>Noto Emoji</family>
     </prefer>
   </alias>

  </fontconfig>

```

`~/.config/fontconfig/conf.d/01-emoji.conf`:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <!-- Use Google Emojis -->
  <match target="pattern">
    <test qual="any" name="family"><string>Segoe UI Emoji</string></test>
    <edit name="family" mode="assign" binding="same"><string>Noto Color Emoji</string></edit>
  </match>
</fontconfig>
```

