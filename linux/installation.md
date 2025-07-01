---
layout: default
title: Linux installation
categories: linux
---
In case of installation on VM make the disk image

```
qemu-img create -f qcow2 debian.qcow2 16G
```

Other format option

```
qemu-img create -f raw debian.raw 16G
-drive file=disk.raw,format=raw
```


Start the VM with [bios](https://github.com/clearlinux/common/raw/master/OVMF.fd)

```bash
qemu-system-x86_64 -bios OVMF.fd -m 1G -drive file=debian.qcow2,format=qcow2 \
                   -cdrom debian-12.2.0-amd64-netinst.iso
```

Go through the installation process and then power off the VM

Start VM with command

```bash
qemu-system-x86_64 -bios OVMF.fd -m 1G -smp 6 \
                   -net user,hostfwd=tcp::2222-:22 -net nic \
                   -drive file=debian.qcow2,format=qcow2
```

## Prepare install media on macOS

Insert and unmount an USB stick

```bash
diskutil unmount /dev/disk2s1
```

Here is how we can list all the disks attached to the system

```bash
diskutil list
```

Writing image to the USB device (notice that we do not write to a partition)

```bash
sudo dd if=archlinux-2021.01.01-x86_64.iso of=/dev/disk2
```

then  flush the data by ejecting the drive

```bash
sudo sync
diskutil eject /dev/disk2
```


# Debian sid

This is the Debian setup where most of the examples shown on the website should work.

Download links:

<!-- https://www.debian.org/CD/live/ -->

- [amd64 - Install disc](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.2.0-amd64-netinst.iso)
- [amd64 - Live disc](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/)

During installation select SSH server, standard system utilities and no desktop

#### Allowing login root user from the network

This is our testing installation, so we do not care about security, but easiness and convenience. 

```bash
nano /etc/ssh/sshd_config
```

`Ctrl-o` `Ctrl-x`

Put this line in the file
```plain
PermitRootLogin yes
```

and then

```bash
systemctl restart sshd
```

From now on you can log in to the VM using ssh connection

```
ssh -p 2222 user@localhost
```

### Making it the sid 


/etc/apt/sources.list

```
deb https://deb.debian.org/debian/ sid main contrib non-free non-free-firmware
deb-src https://deb.debian.org/debian/ sid main contrib non-free non-free-firmware
```

```bash
apt update
apt upgrade
apt dist-upgrade
apt autoremove

apt install firmware-linux-nonfree
```

### Tools


```sh
apt install neovim clang
```

**Swift**

Dependencies

```sh
apt install build-essential libcurl4-openssl-dev binutils git gnupg2 libc6-dev \
            libedit2 libsqlite3-0 libxml2-dev libz3-dev pkg-config tzdata \
            tzdata unzip zlib1g-dev libgcc-9-dev libncurses-dev \
            libstdc++-9-dev
```

Missing dependencies for Debian sid

```text
libpython3.8 
```

Installed instead

```plain
apt install libpython3.10-dev python3-clang python3-lldb
```


Downloading and installing

```
wget https://download.swift.org/swift-5.9.1-release/ubuntu2204/swift-5.9.1-RELEASE/swift-5.9.1-RELEASE-ubuntu22.04.tar.gz

tar -xf swift-5.9.1-RELEASE-ubuntu22.04.tar.gz
mv swift-5.9.1-RELEASE-ubuntu22.04 /opt/swift-5.9.1
```


add this line to `/etc/profile` so the path will be added for all the users

```plain
export PATH="$PATH:/opt/swift-5.9.1/usr/bin"
```

# Archlinux

Links:
- [torrents with ISO files](https://archlinux.org/releng/releases/)
- [latest ISO version from HTTP mirror](https://geo.mirror.pkgbuild.com/iso/latest/)
- [Offline: Most recent installation guide](https://wiki.archlinux.org/index.php/Offline_installation)
- [UEFI: Most recent installation guide](https://wiki.archlinux.org/title/installation_guideI)

# Installation

Note: in case case it is Mac mini. Hold alt (option)  button on boot up  and select the install disk.

Here are commands to check what discs are attached to the system

```bash
cat /proc/partitions
ls /dev/[s|x|v]d*
lsblk
fdisk –l
ls /dev | grep ‘^[s|v|x][v|d]’$*
```

<!--
The install disk has free space which we can use (for example to create install scripts in case we use the install disk several times)

    cfdisk /dev/sdb

Select thf freespace and hit `[New]` and `Enter` => `[Write]` => `enter` => `yes`  =>`Enter` => `[Quit]`

Now we have unformatted partition. To screate FAT32 execute this commend

   

Mount it as home folder
   mount /dev/sdb4 /root
   # and go to the new home root
   cd

   # this command was given by arch wiki but do not work for me
   # https://wiki.archlinux.org/index.php/FAT
   


Now I realized I can edit system, so the partition that I have just created I can mount on start. There are steps I took.

- On the USB stick there is prtition named Gap1. I remove it becouse it seems to not be needed. I use `gparted` for that.
- Shrink vfat partition to 8000 MiB and place it at the and.

- Crate partitions to look like this: `[1: 628.97MiB]` `[2: 2.2 GiB]` `[3: 59 MiB]` `[4: 4000 MiB]` `[5: 8000MiB]`
    1. `ARCH202101`: It is the oryginal partition ISO9660 I had after writing image
    2. Freespace that will might be used when I edit the first partition 
    3. The UEFI partition where is placed bootloader that starts sysyem that is located on the first partition. This partition comes form the orygunal image.
    4. `CHROOT`: This parition will contains files of the installer system. They are placed on Ext4 partition, so we can edit files and regenerate ISO file from it.
    5. Home folder for root user, so when we can write scripts, store files so we can use them in other instalation process. 

Now we copy read only files from read only system to writable partition
    
    sudo mount -o loop /media/artur/ARCH202101/arch/x86_64/airfs.sfs /mnt 
    sudo cp -T /mnt /media/artur/CHROOT
-->

Update packages manager

```bash
pacman -Sy
```


Make a partition table

```bash
parted -s /dev/sda mktable GPT
```

### Create partitions
 
In this case: 

- 300MB  &rarr;  UEFI
- 16GB  &rarr;  Swap
- Rest  &rarr;  System 

List all types of partitions

```bash
sfdisk -T
```

#### First way: Using `sfdisk`

```bash
sfdisk /dev/sda << EOF
 ,300,ef
 ,16000,S,h
 ;
EOF
```

or pipe to the program like

```bash
echo ',,c;' | sfdisk /dev/sdd
```

#### Secund way: Using `fdisk`

```bash
fdisk /dev/sda << FDISK_CMDS
g
n
1

+300MiB
t
1
n
2

+16GiB
t
2
19
n
3


t
3
20
w
FDISK_CMDS
```

#### Therd way

```bash
cfdisk /dev/sda
```

Formatting

```bash
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
```

Mounting partitions

```bash
mount /dev/sda3 /mnt
swapon /dev/sda2
```

Installing base packages

```bash
pacstrap /mnt base base-devel linux linux-firmware
```

Chrooting 

```bash
arch-chroot /mnt << CHROOT
	#commands
CHROOT
```

or just `arch-chroot /mnt` and then commands

```bash
echo "archlinux" > /etc/hostname
sed -i "s/#en_US/en_US/g" /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
ln -s /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
hwclock --systohc --utc
```

#### Run 32bit apps

open  &rarr; `/etc/pacman.conf` and uncomment `[multilib]` section

Installing additional software

```bash
pacman -Syu
pacman -S zsh --noconfirm
```

#### Setup users

Set password for root

```bash
passwd
```

Create user 

```bash
useradd -mg users -G wheel,storage,power -s /usr/bin/zsh user
```

```bash
passwd user
```

You can force user to change password

```bash
chage -d 0 user
```

sudoers

```bash
visudo
```

or 

```bash
cat >> /etc/sudoers <<EOL
%wheel ALL=(ALL) NOPASSWD: ALL
EOL
```

### Setting up bootloader

```bash
pacman -S grub efibootmgr dosfstools os-prober mtools --noconfirm
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=grub_uefi --recheck

grub-mkconfig -o /boot/grub/grub.cfg
```


then `exit` and 

Generate `fstab`

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```


- `umount -a`
- `poweroff`


## Install any Linux out from VM

Create OS archive locally using rsync

```bash
rsync -aAXHSv /* /root/winux --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/root/*,/media/*,/lost+found,/home/*/.gvfs}
tar -cvjSf winux.tar.bz2 winux
```

or

```bash
tar -cvjSf winux.tar.bz2 / --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/root/*,/media/*,/lost+found,/home/*/.gvfs}
```

Create archive and move it to the host

```
ssh -p 2222 root@localhost 'tar -cvjS / --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/root/*,/media/*,/lost+found,/home/*/.gvfs} --to-stdout' > winux.tar.bz2
```

#### Extract files to the destination drive

```bash
tar -xvjf winux.tar.bz2 -C /mnt/sys
```

https://wiki.archlinux.org/title/Moving_an_existing_install_into_(or_out_of)_a_virtual_machine

Then generate fstab

```bash
genfstab -U /mnt/sys > /mnt/sys/etc/fstab
```

<!--
## More customisation

#### DHCP

```
pacman -S dhcpcd --noconfirm
```

```s
systemctl enable dhcpcd
systemctl start dhcpcd
```

```bash
dhcpcd enp3s0f0
```

```
ip a

ping -c2 google.com
```

## Sway

```bash
pacman -S git meson wlroots wayland wayland-protocols \
              pcre2 json-c pango cairo gdk-pixbuf2
```

```bash
git clone https://github.com/swaywm/sway.git
```

building 

```bash
git checkout v1.8
```

Disable waring as errors in  `meson.build` 10 line `'werror=false'`

```bash
meson build/
ninja -C build/
sudo ninja -C build/ install
```

# Interesting software

- zathura - PDF reader
- poppler => gives pdftotext



# Building own environment (inside QEMU) - Archlinux

```bash
pacman -S qemu-guest-agent
systemctl enable --now qemu-guest-agent
```

### install software to build and run

```bash
pacman -S egl-wayland meson wlroots wayland wayland-protocols pcre2 json-c pango cairo gdk-pixbuf2 scdoc cmake less xorg-xwayland xdg-desktop-portal-wlr xdg-desktop-portal-gtk ttf-bitstream-vera gnu-free-fonts noto-fonts ttf-croscore ttf-dejavu ttf-droid ttf-ibm-plex ttf-liberation xorg
```

```bash
git clone https://github.com/swaywm/sway.git
cd sway/
git checkout v1.8

meson build/
ninja -C build/
sudo ninja -C build/ install
```


# Building own environment (inside QEMU) - Debian



```
apt install -y libnvidia-egl-wayland-dev   meson libwlroots-dev wayland-utils wayland-protocols libpcre2-dev libjson-c-dev \
  libpango-1.0-0 libpangocairo-1.0-0 libcairo2-dev libpango1.0-dev libgdk-pixbuf2.0-dev scdoc cmake xwayland bochs bochs-sdl bochs-term  bochs-wx bochs-x \
  vgabios xscreensaver-gl xscreensaver-gl-extra mesa-vulkan-drivers mesa-utils-bin mesa-utils mesa-drm-shim mesa-common-dev \
  libglx-mesa0 libglw1-mesa-dev libgl1-mesa-dri libglapi-mesa libgl1-mesa-dri libgbm-dev weston libweston-12-dev curl htop \
  openssh-server neovim zsh ruby python3 cmake rust-all ruby-dev ruby-full build-essential npm r-base r-base-dev fzf rclone \
  rtorrent htop bundler neomutt golang ghc cabal-install gulp npm neovim mc tree cmake scala maven imagemagick hexedit erlang \
  nasm binutils nim tmux wget httpie yarn meson util-linux ninja-build git fakeroot build-essential ncurses-dev xz-utils \
  libssl-dev bc flex libelf-dev bison python3-pip libisl-dev texinfo libmpfr-dev libmpc-dev libgmp3-dev genisoimage clang \
  libboost-tools-dev libboost-dev libboost-system-dev gcc g++ make pkg-config libgtk-3-dev libgstreamer1.0-dev \
  libgstreamer-plugins-base1.0-dev cmake ninja-build coreutils libxml2-dev libsqlite3-dev libicu-dev libxslt-dev libjpeg-dev \
  libpng-dev libwebp-dev libsecret-1-dev binutils git gnupg2 libc6-dev libcurl4 libedit2 libgcc-9-dev libsqlite3-0 libstdc++-9-dev \
  libxml2 libz3-dev pkg-config tzdata zlib1g-dev python3 apt-file libwayland-server++1 libwayland-server0 freerdp2-dev \
  freerdp2-wayland libpam-freerdp2-dev libavutil-dev libavcodec-dev libavutil-dev libavformat-dev fossil krusader texlive-full \
  openssh-server neovim zsh ruby python3 cmake binutils git gnupg2 libc6-dev libcurl4 libedit2 libgcc-9-dev libsqlite3-0 \
  libstdc++-9-dev libxml2 libz3-dev pkg-config tzdata zlib1g-dev python3 rust-all ruby-dev ruby-full build-essential npm \
  r-base r-base-dev fzf rclone rtorrent htop bundler neomutt golang ghc cabal-install gulp npm neovim mc tree cmake scala \
  maven imagemagick hexedit erlang nasm  binutils nim  tmux wget httpie  yarn meson util-linux ninja-build python3-pip libisl-dev \
  texinfo libmpfr-dev libmpc-dev libgmp3-dev genisoimage clang libboost-tools-dev libboost-dev libboost-system-dev git fakeroot \
  build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison gcc g++ make pkg-config cmake ninja-build \
  coreutils  libgtk-3-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libxml2-dev libsqlite3-dev libicu-dev libxslt-dev \
  libjpeg-dev libpng-dev libwebp-dev libsecret-1-dev
```
-->