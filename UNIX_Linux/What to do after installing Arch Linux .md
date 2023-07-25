# What to do after installing Arch Linux 

```UNIX/Linux```

A vanilla Arch Linux installation gives you your base operating system with no utilities, allowing you to choose what you want your Operating System to behave like. This allows the user to have complete access over their operating system.


A vanilla installation leaves you with nothing more than just a black screen which is for you to customize. In this module, we’ll be walking through the essential things to do after installing Arch Linux.


# 1. Update The System


First things first, update the system with the pacman command:


```
$ sudo pacman -Syyu

```


Now we can go ahead and install packages and other application on our system!


# 2. Install A Display Server


To get a GUI environment, first we need to install a Display Server. The go-to option is to install xorg, which is one of the oldest and the most popular display servers out there.


```
$ sudo pacman -S xorg

```


# 3. Install A Desktop Environment


Next up, we would need a Desktop Environment for our distro. Popular choices include:


- Xfce4
- KDE Plasma
- Gnome
- Cinnamon
- MATE

To install Xfce4:


```
$ sudo pacman -S xfce4 xfce4-goodies

```


To install KDE Plasma:


```
$ sudo pacman -S plasma

```


To install Gnome:


```
$ sudo pacman -S gnome gnome-extra

```


To install Cinnamon:


```
$ sudo pacman -S cinnamon nemo-fileroller

```


To install MATE:


```
$ sudo pacman -S mate mate-extra

```


# 4. Install A Display Manager


Next up, we would need a Display Manager which would enable us to login to our Desktop Environments. The popular choices are :


- LightDM
- LXDM
- SDDM

To install LightDM:


```
$ sudo pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings

```


Enable lightdm with:


```
$ sudo systemctl enable lightdm

```


To install LXDM:


```
$ pacman -S lxdm

```


Enable LXDM with:


```
$ sudo systemctl enable lxdm.service

```


To install SDDM:


```
$ sudo pacman -S sddm

```


Enable SDDM with:


```
$ sudo systemctl enable sddm

```


# 5. Install An AUR Helper


One of the main reasons is the Arch User Repository (AUR) which has a vast array of packages and application. However, we cannot fetch these packages directly using pacman. To fetch packages from AUR we need special programs called AUR Helpers. There are many such helpers available but the one we recommend is paru.


To install Paru:


```
$ sudo pacman -S base-devel git --needed 
$ cd paru
$ makepkg -si

```


Now we can fetch packages from AUR with:


```
$ paru -S <PACAKGE-NAME>

```


# 6. Install Additional Kernels


It is considered good practice to have multiple kernels at your disposal, just in case the main kernel runs into any issues.


The popular kernels apart from the mainline Linux Kernel are :


- Linux LTS Kernel
- Linux Hardened Kernel
- Linux Zen Kernel

To install the LTS kernel:


```
$ sudo pacman -S linux-lts linux-lts-headers

```


To install the Hardened kernel:


```
$ sudo pacman -S linux-hardened linux-hardened-headers

```


To install the Zen kernel:


```
$ sudo pacman -S linux-zen linux-zen-headers

```


# 7. Install Microcode


Processor manufacturers release stability and security updates to the processor microcode. These updates provide bug fixes that can be critical to the stability of your system. Without them, you may experience spurious crashes or unexpected system halts that can be difficult to track down. It is recommended to install it after Arch install just for the sake of stability.


For Intel Processors:


```
$ sudo pacman -S intel-ucode
$ sudo grub-mkconfig -o /boot/grub/grub.cfg

```


For AMD Processors:


```
$ sudo pacman -S linux-firmware
$ sudo grub-mkconfig -o /boot/grub/grub.cfg

```


# 8. Rank Mirrorlists


In order to have faster updates, you can rank your mirrors according to their speed. To do so, first backup your current mirrorlist.


```
# mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak

```


Next up, to rank all mirrors based on their speed with:


```
#  rankmirrors /etc/pacman.d/mirrorlist.bak > /etc/pacman.d/mirrorlist

```


# Conclusion


Thus in this module we covered the essential things to be done after an Arch install. There’s still a lot to do, especially regarding the installation of essentials but we would leave that to the reader as to what applications they want to work with !


