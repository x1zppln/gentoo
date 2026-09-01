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
echo "Brazil/East" > /etc/timezone
emerge --config timezone-data
env-update && source /etc/profile
PARTITION_ROOT=$(findmnt -n -o SOURCE /) 
PARTITION_BOOT=$(findmnt -n -o SOURCE /boot)  
UUID_ROOT=$(blkid -s UUID -o value $PARTITION_ROOT) 
UUID_BOOT=$(blkid -s UUID -o value $PARTITION_BOOT) 
PARTUUID_ROOT=$(blkid -s PARTUUID -o value $PARTITION_ROOT)
rm -rf /etc/portage/package.use
rm -rf /etc/portage/package.accept_keywords
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/package.use -o /etc/portage/package.use
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/package.accept_keywords -o /etc/portage/package.accept_keywords
curl -L https://raw.githubusercontent.com/x1zppln/gentoo/refs/heads/main/make.conf -o /etc/portage/make.conf
mkdir -p /etc/portage/env
 curl -L https://gist.githubusercontent.com/emrakyz/23bf6fe9c30aa0b1eb88021889750ace/raw/832a0160ac0d0383c4f600da5cf8af4290019ff6/compiler-firefox -o /etc/portage/env/compiler-firefox 
 echo "www-client/firefox compiler-firefox" > /etc/portage/package.env
 emerge --update --newuse @world
 
