INSTALLATION DE BASE
--------------------

### Prérequis :

On charge le clavier en français :

```
loadkeys fr
```

Et on vérifie qu'Internet soit bien accessible (je conseille une connection par Ethernet pour plus de facilité) :

```
ping archlinux.org
```

&nbsp;

### Partitionnement :

## Pour un PC sous UEFI :

```
cfdisk
```

On choisit un partitionnement en `GPT` puis on attribue les différentes partitions :

| Référence | Point de montage | Taille                    | Type         |
|-----------|------------------|---------------------------|--------------|
| /dev/sda1 | /boot            | 512 Mo                    | EFI System   |
| /dev/sda2 | -                | Taille identique à la RAM | Linux Swap   |
| /dev/sda3 | /                | Le reste                  | Linux System |

On quitte et on formate les partitions :

```
mkfs.vfat /dev/sda1
mkfs.ext4 /dev/sda3
mkswap /dev/sda2
swapon /dev/sda2
```

>Il est déconseillé d'utiliser une partition Swap sur un disque SSD.


On monte ensuite les partitions :

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## Pour un ancien PC sous BIOS :


```
cfdisk
```

On choisit un partitionnement en `DOS` puis on attribue les différentes partitions :

| Référence | Point de montage | Taille                    | Type         |
|-----------|------------------|---------------------------|--------------|
| /dev/sda1 | /boot            | 512 Mo                    | Linux System |
| /dev/sda2 | -                | Taille identique à la RAM | Linux Swap   |
| /dev/sda3 | /                | Le reste                  | Linux System |

On quitte et on formate les partitions :

```
mkfs.ext2 /dev/sda1
mkfs.ext4 /dev/sda3
mkswap /dev/sda2
swapon /dev/sda2
```

>Il est déconseillé d'utiliser une partition Swap sur un disque SSD.


On monte ensuite les partitions :

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

&nbsp;

### Installation de base :

#### Préparation :

On modifie le fichier `/etc/pacman.d/mirrorlist` (avec nano ou vi) pour ne garder qu'un ou deux serveurs français (mir.archlinux.fr et/ou archlinux.polymorf.fr par ex.) et on installe la base (et vim pour l'édition des fichiers) :

```
pacstrap /mnt base base-devel pacman-contrib
pacstrap /mnt vim
```

On génère le fichier fstab listant les partitions :

```
genfstab -U -p /mnt >> /mnt/etc/fstab
```

On entre dans l'environnement :

```
arch-chroot /mnt
```

&nbsp;

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
options root=PARTUUID=[PARTUUID_sda2] rw
```

**NB :** Pour obtenir le PARTUUID dans vim, il faut taper `ECHAP` puis `:r !blkid`

>Sur d'anciens PCs en BIOS, on utilise le bootloader grub :
>
>```
>pacman -S grub os-prober intel-ucode
>mkdir /boot/grub
>grub-install --no-floppy --recheck /dev/sda
>grub-mkconfig -o /boot/grub/grub.cfg
>```


&nbsp;

#### Passage du système en FR :

On modifie/crée `/etc/vconsole.conf` :

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

```
locale-gen
```

Pour spécifier le fuseau horaire :

```
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc --utc
```

&nbsp;

#### Génération du noyau et prise en charge du réseau

On génère le noyau :

```
mkinitcpio -p linux
```

Et on installe NetworkManager (qu'on active au démarrage) :

```
pacman -Syy networkmanager
systemctl enable NetworkManager
```

&nbsp;

### Reboot :)

On peut maintenant sortir de l'environnement, éjecter le support et rebooter :

```
exit
umount -R /mnt
reboot
```

Archlinux est installée ! On peut maintenant passer à l'installation de l'environnement Gnome, et des paquets utiles au quotidien.

&nbsp;

INSTALLATION DE L'ENVIRONNEMENT GRAPHIQUE
-----------------------------------------

### Préparation

On active le support "multilib" pour pouvoir installer des librairies 32 bits, nécessaires à certains programmes (Steam par ex.), en décochant les lignes suivantes dans le fichier `/etc/pacman.conf` :

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

On en profite pour activer la coloration en décochant ```Color``` dans ce même fichier `/etc/pacman.conf`

On actualise les dépôts :

```
pacman -Sy
```

On crée un utilisateur avec les bons droits :

```
useradd -m -g wheel -c 'Prénom NOM' -s /bin/bash [user]
passwd [user]
```

Puis on tape `visudo` et on décommente la ligne juste en dessous de `#Uncomment to allow members of group wheel to execute any command`

On installe ensuite yay pour pouvoir utiliser le Arch User Repository :

```
pacman -S git
sudo -s -u [user]
cd
git clone https://aur.archlinux.org/yay
cd yay
makepkg -sri
```

&nbsp;

### Installation des paquets nécessaires

On passe ensuite aux installations de paquets avec yay (toujours loggé en tant que [user]), qui propose également une interface pour pacman :

* Base

```
yay -S zip unzip p7zip mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion unrar rsync --noconfirm
yay -S ntp cronie --noconfirm
yay -S xorg-{server,xinit,apps} xf86-input-{mouse,keyboard} xdg-user-dirs --noconfirm
```

&nbsp;

* Pilotes graphiques (ici pour Intel)

```
yay -S xf86-video-intel --noconfirm
```

> Pour les pilotes libres, il faut choisir ```xf86-video-ati``` pour une carte AMD et ```xf86-video-nouveau``` pour une carte Nvidia.

&nbsp;

* Impression (La deuxième ligne est valable dans le cas d'une imprimante Canon)

```
yay -S cups system-config-printer --noconfirm
yay -S cnijfilter2 scangearmp2
```

&nbsp;

* Multimedia (Codecs et gestion de l'ouverture des fichiers par mimetype)

```
yay -S gst-plugins-{base,good,bad,ugly} gst-libav --noconfirm
yay -S perl-file-mimeinfo --noconfirm
```

&nbsp;

* Pour un PC portable (Gestion de l'énergie et du pavé tactile)

```
yay -S tlp --noconfirm
yay -S xf86-input-libinput --noconfirm
```

&nbsp;

* Quelques polices

```
yay -S ttf-{bitstream-vera,liberation,freefont,dejavu} freetype2 otf-fira-sans otf-fira-mono ttf-roboto ttf-ubuntu-font-family --noconfirm
```

&nbsp;

* Et l'environnement de bureau :)

	* Pour Gnome :
	```
	yay -S gnome gnome-extra --noconfirm
	```

	* Pour Kde :
	```
	yay -S plasma kde-applications --noconfirm
	```

	* Pour Xfce :
	```
	yay -S xfce4 xfce4-goodies network-manager-applet lightdm-gtk-greeter lightdm-gtk-greeter-settings --noconfirm
	```

&nbsp;

### Derniers réglages

* Pour la gestion des logs

```
sudo vim /etc/systemd/journald.conf 
ForwardToSyslog=yes
```

&nbsp;

* Pour avoir le clavier en français dès le gestionnaire de sessions :

```
sudo localectl set-x11-keymap fr
```

&nbsp;

* On active quelques services au démarrage :

```
sudo systemctl enable syslog-ng@default
sudo systemctl enable cronie
sudo systemctl enable avahi-daemon
sudo systemctl enable avahi-dnsconfd
sudo systemctl enable org.cups.cupsd
sudo systemctl enable ntpd
```

* Puis on active le gestionnaire de sessions :

	* Pour Gnome (gdm) :

	```
	sudo systemctl enable gdm
	```

	* Pour Kde (sddm) :

	```
	sudo systemctl enable sddm
	```

	* Pour Xfce (lightdm) :

	```
	sudo systemctl enable lightdm
	```

&nbsp;

* Pour un boot plus "sexy" :

Il faut modifier les hooks dans le fichier `/etc/mkinitcpio.conf` :

```
#HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
HOOKS=(base systemd autodetect keyboard sd-vconsole modconf block filesystems fsck)
```

Et regénérer le noyau : 

```
sudo mkinitcpio -p linux
```

&nbsp;

* Pour la gestion du lecteur d'empreintes :

```
yay -S fprintd
```

Dans le fichier `/etc/pam.d/system-local-login` :

```
auth      sufficient pam_fprintd.so
auth      include   system-login
```

Et on enregistre son empreinte avec `fprintd-enroll`

Pour des raisons de sécurité, on crée le fichier `/usr/share/polkit-1/rules.d/50-net.reactivated.fprint.device.enroll.rules` avec le contenu suivant :

```
polkit.addRule(function (action, subject) {
  if (action.id == "net.reactivated.fprint.device.enroll") {
    return subject.user == "root" ? polkit.Result.YES : polkit.result.NO
  }
})
```

&nbsp;

### Customisation

* Outils :

```
yay -S galculator --noconfirm
yay -S filezilla --noconfirm
yay -S otpclient --noconfirm
yay -S jdk --noconfirm
yay -S imagemagick --noconfirm
yay -S wine lib32-libpulse --noconfirm
yay -S lanshare --noconfirm
yay -S evince --noconfirm
```

&nbsp;

* Bureautique :

```
yay -S libreoffice-fresh-fr hunspell-fr --noconfirm
```

&nbsp;

* Internet :

```
yay -S vivaldi vivaldi-ffmpeg-codecs chrome-gnome-shell --noconfirm
yay -S deluge pygtk --noconfirm
yay -S jdownloader2 --noconfirm
```

&nbsp;

* Multimédia :

```
yay -S pavucontrol --noconfirm
yay -S spotify --noconfirm
yay -S mkvtoolnix-gui --noconfirm
yay -S soundconverter --noconfirm
yay -S smplayer smplayer-themes --noconfirm
yay -S vlc --noconfirm
yay -S gimp gimp-help-fr --noconfirm
yay -S inkscape --noconfirm
yay -S blender --noconfirm
yay -S easytag --noconfirm
```

&nbsp;

* Gaming :

```
yay -S steam --noconfirm
yay -S retroarch retoarch-assets-xmb libretro-shaders-glsl libretro-snes9x libretro-mgba libretro-bsnes --noconfirm
```

&nbsp;

* Apparence :

```
yay -S pacman -S gtk-engine-murrine gtk-engines --noconfirm
```

```
mkdir .themes
cd .themes
git clone https://github.com/vinceliuice/vimix-gtk-themes.git
```

Et on installe le thème.

```
mkdir .icons
cd .icons
git clone https://github.com/vinceliuice/vimix-icon-theme.git
```

Et on installe le thème.

&nbsp;

* Terminal

```
yay -S rxvt-unicode
```

Et on crée le fichier `~/.Xresources`

Sous Gnome, il faut créer le fichier `~/.pam_environment` pour charger le fichier `.Xresources` au démarrage avec Wayland, avec le contenu suivant :

```
XENVIRONMENT             DEFAULT=@{HOME}/.Xresources
```
