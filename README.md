# arch_install

```loadkeys fr```

```cfdisk```

=> GPT
=> New 512M Type EFI System
=> New XXXG Type Linux filesystem
=> Quit

```
mkfs.vfat /dev/sda1
mkfs.ext4 /dev/sda2
```

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

```pacstrap /mnt base base-devel pacman-contrib```
```pacstrap /mnt zip unzip p7zip vim mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion unrar rsync```

```genfstab -U -p /mnt >> /mnt/etc/fstab```

```arch-chroot /mnt```

```bootctl install```

```pacman -S intel-ucode```

```vim /boot/loader/loader.conf```
```
default arch
timeout 0
```

```vim /boot/loader/entries/arch.conf```
```
title Archlinux
linux /vmlinuz-linux
initrd /initramfs-linux.img
initrd /intel-ucode.img
options root=PARTUUID=[PARTUUID_sda2_sans_les_guillemets] rw (:r !blkid pour avoir l'id)
```

```vim /etc/vconsole.conf```
```
KEYMAP=fr-latin9
FONT=eurlatgr
```

```vim /etc/locale.conf```
```
LANG=fr_FR.UTF-8
LC_COLLATE=C
```

```vim /etc/locale.gen```
```
fr_FR.UTF-8 UTF-8
```

```locale-gen```

```ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime```
```hwclock --systohc --utc```

```mkinitcpio -p linux```

```pacman -Syy networkmanager```
```systemctl enable NetworkManager```

```exit```
```umount -R /mnt```
```reboot```



pacman -S ntp cronie

vim /etc/systemd/journald.conf 
ForwardToSyslog=yes

pacman -S gst-plugins-{base,good,bad,ugly} gst-libav
pacman -S xorg-{server,xinit,apps} xf86-input-{mouse,keyboard} xdg-user-dirs
pacman -S cups

pacman -S tlp
pacman -S xf86-input-libinput

pacman -S ttf-{bitstream-vera,liberation,freefont,dejavu} freetype2

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

yay -S otf-fira-sans otf-fira-mono ttf-roboto ttf-ubuntu-font-family


