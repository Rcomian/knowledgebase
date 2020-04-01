## DOOM Eternal on gentoo

Sources:

* [CTT - Doom Eternal on Linux | How to Setup and Game Performance](https://www.youtube.com/watch?v=g3UPxd8iUsU&t=1s)
* [TLG - Here are 3 EASY STEPS to play DOOM ETERNAL on Linux!
](https://www.youtube.com/watch?v=u2_Eoqekr9o)
* [krypalkora - Proton 5.4-GE-2 Steam Linux | Doom Eternal | GTX 1650 | Ubuntu 19.10 | Playable | 60 fps | GamePlay](https://www.youtube.com/watch?v=cwhoIZcFXLs)

Doom had problems on NVIDIA graphics cards.

Need [Proton 5.4-GE-3](https://github.com/GloriousEggroll/proton-ge-custom/releases) or higher.

Need [NVidia Drivers 440.66.04](https://developer.nvidia.com/vulkan-driver) or higher.

Need vulkan icd v1.2.135 or higher - get by using [steam beta](https://support.steampowered.com/kb_article.php?ref=7021-eiah-8669).

### Install NVIDIA drivers

```bash
emerge --unmerge nvidia-drivers 
rc-update del xdm
reboot
/home/dragon/Downloads/NVIDIA-Linux-x86_64-440.66.04.run
rc-update add xdm default
rc-config start xdm
```

