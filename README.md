INSTALLATION DE BASE
--------------------

### Chargement du clavier en français :

```loadkeys fr```

&nbsp;

### Partitionnement :

```cfdisk```

On choisit un partitionnement en `GPT` puis on attribue les différentes partitions :

| Référence | Point de montage | Taille   | Type         |
|-----------|------------------|----------|--------------|
| /dev/sda1 | /boot            | 512 Mo   | EFI System   |
| /dev/sda2 | /                | Le reste | Linux System |

On quitte et on formate les partitions :

```
mkfs.vfat /dev/sda1
mkfs.ext4 /dev/sda2
```

>Dans le cas où on souhaiterait une partition Swap :
>
>| Référence | Point de montage | Taille                      | Type         |
>|-----------|------------------|-----------------------------|--------------|
>| /dev/sdaX | -                | Taille identique à la RAM   | Linux Swap   |
>
>Puis :
>
>```
>mkswap /dev/sdaX
>swapon /dev/sdaX
>```

On monte ensuite les partitions :

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

&nbsp;

### Installation de base :

#### Préparation :

On modifie le fichier `/etc/pacman.d/mirrorlist` pour ne garder que les serveurs français (mir.archlinux.fr et/ou archlinux.polymorf.fr par ex.) et on installe la base :

```pacstrap /mnt base base-devel pacman-contrib```

On génère le fichier fstab listant les partitions :

```genfstab -U -p /mnt >> /mnt/etc/fstab```

On entre dans l'environnement :

```arch-chroot /mnt```

#### Mise en place du bootloader et installation du microcode :

On installe le bootloader (ici le bootloader de systemd) et le microcode associé au processeur (ici intel) :

`bootctl install` pour l'installation du bootloader.

`pacman -S intel-ucode` pour l'installation du microcode.

On crée le fichier `/boot/loader/loader.conf` comme suit (le timeout est ici à 0 mais peut être maintenu à 3 ou 4 secondes dans un premier temps) :

```
default arch
timeout 0
```

Et le fichier `/boot/loader/entries/arch.conf` :

```
title Archlinux
linux /vmlinuz-linux
initrd /initramfs-linux.img
initrd /intel-ucode.img
options root=PARTUUID=[PARTUUID_sda2_sans_les_guillemets] rw (:r !blkid pour avoir l'id)
```

#### Passage du système en FR :

On modifie `/etc/vconsole.conf` :

```
KEYMAP=fr-latin9
FONT=eurlatgr
```

Puis `/etc/locale.conf` :

```
LANG=fr_FR.UTF-8
LC_COLLATE=C
```

Puis `/etc/locale.gen` :

```
fr_FR.UTF-8 UTF-8
```

Et on lance la commande :

```locale-gen```

Pour spécifier le fuseau horaire :

```
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc
```

#### Génération du noyau et prise en charge du réseau

On génère le noyau :

```mkinitcpio -p linux```

Et on installe NetworkManager (qu'on active au démarrage) :

```
pacman -Syy networkmanager
systemctl enable NetworkManager
```

### Reboot :)

On peut maintenant sortir de l'environnement, éjecter le support et rebooter :

```
exit
umount -R /mnt
reboot
```

Archlinux est installée ! On peut maintenant passer à l'installation de l'environnement Gnome, et des paquets nécessaires au quotidien.

&nbsp;

INSTALLATION DE L'ENVIRONNEMENT GRAPHIQUE
-----------------------------------------

pacman -Sy

pacman -S ntp cronie

vim /etc/systemd/journald.conf 
ForwardToSyslog=yes

pacman -S gst-plugins-{base,good,bad,ugly} gst-libav
pacman -S xorg-{server,xinit,apps} xf86-input-{mouse,keyboard} xdg-user-dirs
pacman -S cups

pacman -S tlp
pacman -S xf86-input-libinput

pacman -S ttf-{bitstream-vera,liberation,freefont,dejavu} freetype2
yay -S otf-fira-sans otf-fira-mono ttf-roboto ttf-ubuntu-font-family

useradd -m -g wheel -c 'Marc SISTO' -s /bin/bash msisto
passwd msisto

visudo
#Uncomment to allow members of group wheel to execute any command

pacman -S gnome gnome-extra

sudo localectl set-x11-keymap fr

systemctl enable syslog-ng@default
systemctl enable cronie
systemctl enable avahi-daemon
systemctl enable avahi-dnsconfd
systemctl enable org.cups.cupsd
systemctl enable ntpd

systemctl enable gdm

pacman -S git
git clone https://aur.archlinux.org/yay
cd yay
makepkg -sri

pacman -S vivaldi vivaldi-ffmpeg-codecs deluge pygtk

pacman -S xf86-video-intel 

pacman -S gtk-engine-murrine gtk-engines

mkdir .themes
cd .themes
git clone https://github.com/vinceliuice/vimix-gtk-themes.git
Install

mkdir .icons
cd .icons
git clone https://github.com/vinceliuice/vimix-icon-theme.git
Install

yay -S galculator --noconfirm
yay -S spotify --noconfirm
mkvtoolnix-gui
soundconverter
filezilla
smplayer
vlc
gimp gimp-help-fr libreoffice-fresh-fr hunspell-fr inkscape blender

/etc/pacman.conf
[multilib]
Include = /etc/pacman.d/mirrorlist

yay -S steam
retroarch retoarch-assets-xmb libretro-shaders libretro-snes9x libretro-mgba libretro-bsnes

yay -S otpclient

jdk
jdownloader2
imagemagick

vim /etc/mkinitcpio.cong
#HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block filesystems fsck)
mkinitcpio -p linux

yay -S fprintd
/etc/pam.d/system-local-login
auth      sufficient pam_fprintd.so
auth      include   system-login

fprintd-enroll

/usr/share/polkit-1/rules.d/50-net.reactivated.fprint.device.enroll.rules
polkit.addRule(function (action, subject) {
  if (action.id == "net.reactivated.fprint.device.enroll") {
    return subject.user == "root" ? polkit.Result.YES : polkit.result.NO
  }
})



