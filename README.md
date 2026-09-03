fdisk /dev/nvme0n1p3
mkfs.ext4 /dev/nvme0n1p3
mkfs.vfat -F 32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
swapon /dev/nvme0n1p2
mount /dev/nvme0n1p3 /mnt/gentoo/
cd /mnt/gentoo
links gentoo.org
tar xpvf stage3-amd64-openrc-20260830T151604Z.tar.xz --xattrs-include='*.*' --numeric-owner
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
chroot /mnt/gentoo/ /bin/bash


mount /dev/nvme0n1p1 /boot/
emerge-webrsync
env-update && source /etc/profile
PARTITION_ROOT=$(findmnt -n -o SOURCE /) 
PARTITION_BOOT=$(findmnt -n -o SOURCE /boot)  
UUID_ROOT=$(blkid -s UUID -o value $PARTITION_ROOT) 
UUID_BOOT=$(blkid -s UUID -o value $PARTITION_BOOT) 
PARTUUID_ROOT=$(blkid -s PARTUUID -o value $PARTITION_ROOT)
echo "UUID=$UUID_BOOT /boot vfat defaults,noatime 0 2
UUID=$UUID_ROOT / ext4 defaults,noatime 0 1
/dev/nvme0n1p2 none swap sw 0 0" > /etc/fstab


echo "Brazil/East" > /etc/timezone
emerge --config timezone-data

emerge app-eselect/eselect-repository dev-vcs/git
eselect repository enable hyproverlay 
emaint sync -r hyproverlay
mkdir -p /home/diogo/.config/hypr
curl -L https://raw.githubusercontent.com/hyprwm/Hyprland/3229862dd4cbfa93638a4d16ed86ec2fda5d38a6/example/hyprland.conf -o /home/diogo/.config/hypr/hyprland.conf
echo "exec-once=dbus-launch gentoo-pipewire-launcher & hyprpaper" >> /home/diogo/.config/hypr/hyprland.conf
echo "exec-once=/home/diogo/.config/hypr/portalstart" >> /home/diogo/.config/hypr/hyprland.conf


rm -rf /etc/portage/package.use
rm -rf /etc/portage/package.accept_keywords
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/package.use -o /etc/portage/package.use
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/package.accept_keywords -o /etc/portage/package.accept_keywords
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/make.conf -o /etc/portage/make.conf
emerge --update --newuse @world gui-wm/hyprland foot wofi dunst imv doas gnome-base/gsettings-desktop-schemas wl-clipboard xdg-desktop-portal-hyprland dhcpcd efibootmgr doas
emerge @preserved-rebuild
emerge --depclean


sed -i "s/hostname=.*/hostname=\"alqola\"/g" /etc/conf.d/hostname


echo "permit :wheel
permit nopass keepenv :diogo
permit nopass keepenv :root" > /etc/doas.conf


mkdir -p /etc/portage/env
curl -L https://gist.githubusercontent.com/emrakyz/23bf6fe9c30aa0b1eb88021889750ace/raw/832a0160ac0d0383c4f600da5cf8af4290019ff6/compiler-firefox -o /etc/portage/env/compiler-firefox 
echo "www-client/firefox compiler-firefox" > /etc/portage/package.env


emerge gentoo-sources
cd /usr/src/linux
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/.config -o .config
sed -i -e '/^CONFIG_CMDLINE="root=PARTUUID=.*/c\' -e "CONFIG_CMDLINE=\"root=PARTUUID=$PARTUUID_ROOT\"" /usr/src/linux/.config
make -j$(nproc)
emerge linux-firmware
dispatch-conf
make modules_install
mkdir -p /boot/EFI/BOOT && cp /usr/src/linux/arch/x86/boot/bzImage /boot/EFI/BOOT/BOOTX64.EFI


cd /etc/init.d/
ln -s net.lo net.enp4s0
rc-update add dhcpcd
rc-update add net.enp4s0
rc-update add seatd default
useradd -mG wheel,audio,video,usb,input,portage,pipewire,seat diogo




echo "misc {
disable_hyprland_logo=1
disable_splash_rendering=1
}

env = QT_SCREEN_SCALE_FACTORS,1;1
env = WLR_NO_HARDWARE_CURSORS,1
env = GBM_BACKEND,nvidia-drm
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = _JAVA_AWT_WM_NONREPARENTING,1
env = ANV_QUEUE_THREAD_DISABLE,1
env = QT_QPA_PLATFORM,wayland
env = CLUTTER_BACKEND,wayland
env = SDL_VIDEODRIVER,wayland
env = XDG_SESSION_TYPE,wayland
env = XDG_CURRENT_DESKTOP,Hyprland
env = XDG_SESSION_DESKTOP,Hyprland
env = MOZ_ENABLE_WAYLAND,1
env = MOZ_DBUS_REMOTE,1" >> /home/diogo/.config/hypr/hyprland.conf


echo "#\!/bin/bash
sleep 1
killall xdg-desktop-portal-hyprland
killall xdg-desktop-portal-wlr
killall xdg-desktop-portal
/usr/libexec/xdg-desktop-portal-hyprland &
sleep 2
/usr/libexec/xdg-desktop-portal &" > /home/diogo/.config/hypr/portalstart


echo "#\!/bin/sh
cd ~
export XDG_RUNTIME_DIR=\"/tmp/hyprland\"
mkdir -p \$XDG_RUNTIME_DIR
chmod 0700 \$XDG_RUNTIME_DIR
exec dbus-launch --exit-with-session Hyprland" >> /home/diogo/.config/hypr/start.sh


chmod +x /home/diogo/.config/hypr/portalstart
chmod +x /home/diogo/.config/hypr/start.sh


[ "$(tty)" = "/dev/tty1" ] && ! pidof -s Hyprland >/dev/null 2>&1 && exec "/home/diogo/.config/hypr/start.sh"


eselect fontconfig disable 10-hinting-slight.conf
eselect fontconfig disable 10-no-antialias.conf
eselect fontconfig disable 10-sub-pixel-none.conf
eselect fontconfig enable 10-hinting-full.conf
eselect fontconfig enable 10-sub-pixel-rgb.conf
eselect fontconfig enable 10-yes-antialias.conf
eselect fontconfig enable 11-lcdfilter-default.conf


rm -rf /var/tmp/portage/*
rm -rf /var/cache/distfiles/*
rm -rf /var/cache/binpkgs/*


efibootmgr -c -d /dev/nvme0n1 -p 1 -L "gentoo" '\EFI\BOOT\BOOTX64.EFI'

passwd
