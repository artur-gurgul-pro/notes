---
layout: default
title: Recipes MD
---

#### Read a DNS records

    dig artgur.net +nostats +nocomments +nocmd

#### I/O Stats

    iostat

#### Checking type of executable files

    otool -hv test.so

#### #Receipe Gzip of image

```shell
dd if=/dev/sdb | gzip > ~/backup.img.gz
```

#### Progress with `dd`

```
sudo dd if=2024-11-19-raspios-bookworm-armhf.img of=/dev/disk27 status=progress
sudo dd if=/dev/sdb | pv -s 5.29G | dd of=DriveCopy1.dd bs=4096
sudo pv ubuntu-24.04.1-desktop-amd64.iso | sudo  dd of=/dev/disk27
```

#### Get directory size

    du -sh MacOSBackup

##### Print all sizes in directory
```bash
du -sh *
```

```bash
du -shc *
```

#### Compare two files

```bash
vim -d file1 file2
mcdiff file1 file2
```

### Disk manager

```bash
cfdisk /dev/sda
```

#### List disks

```
parted -l
```

#### informations about disk

```bash
fdisk -l /dev/sda
```

#### Power off the disk 

```
udisksctl power-off -b /dev/sdX
```
#### Generate random password

```bash
pwgen -s -1 32
```
or
```
openssl rand -hex 12
```
#### List block devices

    lsblk

#### Linux headers

```
uname -r
apt search linux-headers-$(uname -r)
```

Show all disks with json format
```shell
lsblk -J
```
List disk with uuid's
```shell
lsbkl -f
```


#### MKFS

```bash
mkfs.vfat -F 32 /dev/sdb4
```

```bash
mount -i -t vfat -oumask=0000,iocharset=utf8 /dev/sdb4 /root
```

#### See what processes are using the drive 

```
lsof /where/drive/is/mounted
```

### See what process opens the port

```bash
lsof -p 6919
lsof -i :6919
```
#### See the stats of IO

```
apt install sysstat iotop
```


```
iostat -dh 2
iotop -o
sar -p -d -b 1
vmstat -d 1
vmstat -p /dev/sda2 1
```

### Rsync 

```
rsync -ah --progress /Volumes/Data /Volumes/Data\ 1/Junk/1TB\ Drive
```


#### reloading local DNS

```shell
sudo /etc/init.d/dns-clean start
```

#### Print all processes in json format

``` shell
ps aux |
  awk -v OFS=, '{print $1, $2}' |
  jq -R 'split(",") | {user: .[0], pid: .[1]}'
```

### Split files

```
split -b 70M deno
```

#### Search and execute command from the history

```bash
eval `history | fzf | cut  -s -d " " -f4-`
```

Adding this to `.zshrc`

```
export HISTSIZE=100000000
alias hexec='eval `history | fzf | cut  -s -d " " -f4-`'
```
#### Editing command with editor

`~/.zshrc`

```
bindkey '^e' edit-command-line
```
#### Copy public ssh key

```bash 
cat ~/.ssh/id_rsa.pub | pbcopy
```

#### change password that was saved in a variable

```bash
cho "$archpass" | passwd "$archuser" --stdin
```
#### Git diff between branches 

    git diff release-1.2.0..release-1.2.1


#### MacOS info aliases in`.zhrc`

```
  alias cpu='sysctl -n machdep.cpu.brand_string'
  alias cpu-temp='sudo powermetrics --samplers smc | grep -i "CPU die temperature"'
  alias gpu-temp='sudo powermetrics --samplers smc | grep -i "GPU die temperature"'
  alias lsusb='sudo ioreg -p IOUSB'
  alias allusb='ioreg -p IOUSB -w0 -l'
```

**Power metrics**

```bash
sudo powermetrics --samplers all
```
#### Install pods from non standard localisations

```ruby
 pod 'WASHD', :git => 'https://github.com/vatlib/EasyUITextFields.git'
 pod 'WASHD', :path => '/Users/artur/projs/easyuitextfields'
 ```

#### SQLite select and search results with FZF

```bash
echo "select * from bookmarks" | sqlite3 bookmarks.db | fzf
```

#### Open file with FZF

```bash
nvim -o `fzf`
```

#### Set default shell. ZSH in this case

	sudo chsh --shell /usr/bin/zsh user

#### Show Git object
```sh
pigz -d < .git/objects/02/f2cc93fee0b3cb7c9b75f49e4ded3f9b1480eb
```

#### list of wireless cards

    lspci -knn | grep Net -A2

#### Scan networks 

    iwlist scan

#### Shutdown

    shutdown -h now

#### Connect to the network

    nmcli dev wifi connect TP-Link_5828 password my-secret-pass

#### You can forward port `80` to `8090`

```shell
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8090
```

#### Allow accepting connections on `8090` 

```shell
iptables -I INPUT -m tcp -p tcp --dport 8090 -j ACCEPT
```
#### Search files that contains particular string 

``` shell
grep -rnw "." -e "Search key"
```

#### Remove garbage files

    find ./ -name ".DS_Store" -depth -exec rm {} \;

#### Find files, directories and symbolic links using regex
    
    find ./ -iname `fo*` and `F??` -type f,d,l

#### Make text from pipe uppercased

```bash
cat file.txt | tr [:lower:] [:upper:]
cat file.txt | tr [a-z] [A-Z]
tr [a-z] [A-Z] < linux.txt > output.txt
```

#### Installing packages for python

**_just for user_**

```bash 
pip3 install --user meson
```

**_calling module through interpreter_**

```bash
python3 -m pip install six
```

#### Remove spaces

```bash
cat file.txt | tr -d ' ' 
```

#### Remove duplicate characters


```bash 
$ cat domains.txt

www.google.....com
www.linkedin.com
www.linuxsay.com
```

```bash
$ cat domains.txt | tr -s '.' 

www.google.com
www.linkedin.com
```

#### Extract digit

``` bash
echo "My UID is $UID" | tr -cd "[:digit:]\n"
echo "My UID is $UID" | tr -d "a-zA-Z"
```

#### Translate single character

```bash
echo "My UID is $UID" | tr " "  "\n"
```

#### Get path by number

```bash
echo $PATH | cut -d ":" -f 1
```

#### list search path line by line

```bash
echo $PATH | tr ":" "\n"
```

#### Screen capture

```
ffmpeg -f x11grab -video_size 1280x800 -framerate 25 -i $DISPLAY -c:v ffvhuff screen.mkv

ffmpeg -video_size 1280x800 -framerate 25 -f x11grab -i :0.0 -f pulse -ac 2. \
       -i default -vcodec vp8 -acodec libvorbis myvideo_$(date +%d_%B_%Y_%H:%M).webm
```

#### Take a screenshot

```bash
xwd -root -out screenshot.xwd
maim -s -u | xclip -selection clipboard -t image/png -i
imlib2_grab screenshot.png
```

#### Install Python package for the user

    python3 -m pip install --user pyelftools

#### Erase free space 

    sudo diskutil secureErase freespace 1 /Volumes/Data\ Drive

#### Format disk

	sudo diskutil eraseDisk ExFAT data /dev/disk26
#### Search for commit

```bash
alias gf='git log --all --oneline | fzf'
```

#### Remove alpha channel from all files

```bash
# ➜ brew install imagemagick
```

**Converts all files in current directory revursevely**

```bash
alias rmalfa='find . -name “*.png” -exec convert “{}” -alpha off “{}” \;'
```

#### Weather alias

```sh
alias weather='curl wttr.in'
```

#### Starting an electron app on wayland

Start chromium using wayland

```bash
chromium --enable-features=UseOzonePlatform --ozone-platform=wayland
```

It’s the same for electron-based apps:

```bash
`app-executable` --enable-features=UseOzonePlatform \
                 --ozone-platform=wayland
```

#### Save website As PDF

```bash
function aspdf {
  /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --headless --print-to-pdf="./$1.pdf" $2
}
```

Usage

```bash
aspdf "filename" "https://superuser.com/questions/592974/how-to-print-to-save-as-pdf-from-a-command-line-with-chrome-or-chromium"
```

### Export Markdown as PDF

```bash
pandoc README.md -o README.pdf
```

```bash
pandoc --from=gfm --to=pdf -o README.pdf README.md
```

#### Gem path

```hash
export GEM_HOME=$HOME/.gem
path=("$GEM_HOME/bin" $path)
```

#### QEMU - port forwarding

```bash
qemu-system-i386 -net nic,model=rtl8139 \
        -net user,hostfwd=tcp::3389-:3389,hostfwd=tcp::443-:443,hostfwd=tcp::992-:992 
```

#### SQL using regex
Add a check constraint to the `id` column to enforce alphanumeric strings of exactly 5 characters long

```sql
ALTER TABLE short_urls ADD CONSTRAINT id CHECK (id ~ '^[a-zA-Z0-9]{5}$');
```

#### Console font size

Edit file &rarr; `/etc/default/console-setup`

```bash
dpkg-reconfigure -plow console-setup
```

#### Redirect errors to null device

    find / 2>/dev/null

#### Installing nonfree firmware from repository

 I.e: Firmware for nonfree driver for Intel's WIFI cards.

```
https://packages.debian.org/sid/firmware-iwlwifi
```    

```bash
apt-get update && apt-get install firmware-linux-nonfree
```

#### Installing nonfree firmware from ​manufacturer

Search for binary. An example:

[https://www.intel.com/content/www/us/en/support/articles/000005511/wireless.html](https://www.intel.com/content/www/us/en/support/articles/000005511/wireless.html)

Extract and copy like

```bash
cp iwlwifi-cc-a0-46.ucode /lib/firmware
```

### Linux - RAM disk

This might be useful for spead up programs that heavily use disk.

```
mount -t TYPE -o size=SIZE FSTYPE MOUNTPOINT
```
* `TYPE` &rarr; either `tmpfs` or `ramfs`.
* `SIZE` &rarr; ie. `512m`
* `FSTYPE` &rarr; File system type, either `tmpfs`, `ramfs`, `ext4`, etc.

To make this setting persistent you might want to add to `/etc/fstab` fallowing line

```plain
tmpfs /mnt/ramdisk tmpfs nodev,nosuid,noexec,nodiratime,size=1024M 0 0
```

#### fstab

Use 'blkid' to print the universally unique identifiers, and can be used in fstab file like

```
# <file system>                            <mount point>   <type>  <options>                        <dump>  <pass>

UUID=1a38b8ca-e1f5-45e6-bbe8-3abd2775b3a6  /               ext4    errors=remount-ro                0       1
/swapfile                                  none            swap    sw                               0       0
/dev/disk/by-uuid/4D3C-4E36                /mnt/4D3C-4E36  auto    nosuid,nodev,nofail,x-gvfs-show  0       0

UUID=e21eebe4-471a-4375-8c4c-618b3733a940  /home           ext4    nodev,nosuid                     0       2
```


### Linux - Mount disk from `qcow2` image


Step 1 - Enable NBD on the Host

```bash
modprobe nbd max_part=8
```

Step 2 - Connect the QCOW2 as network block device

```bash
qemu-nbd --connect=/dev/nbd0 /var/lib/vz/images/100/vm-100-disk-1.qcow2
```

Step 3 - Find The Virtual Machine Partitions

```bash
fdisk /dev/nbd0 -l
```

Step 4 - Mount the partition from the VM

```bash
mount /dev/nbd0p1 /mnt/somepoint/
```

Step 5 - After you done, unmount and disconnect

```bash
umount /mnt/somepoint/
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```

### Ubuntu - Power management

To make Ubuntu do nothing when laptop lid is closed:

From For 13.10 onwards:

Open the `/etc/systemd/logind.conf` file in a text editor as root, for example:

```bash
sudo -H gedit /etc/systemd/logind.conf
```

If `HandleLidSwitch` is not set to ignore then change it:

```bash
HandleLidSwitch=ignore
```

Other settings that the action can be ignored: `HandleLidSwitchExternalPower`, `HandleLidSwitchDocked`, `IdleAction`.

Restart the systemd daemon (be aware that this command will log you out):

```bash
sudo systemctl restart systemd-logind
```

or, from 15.04 onwards:

```bash
sudo service systemd-logind restart
```

### Chroot environment of Debian sid

 Install Bootstrap

```bash
sudo apt install debootstrap
```

Create a directory that you want to use for the base system (_chroot-debian_ in this case)

```bash
mkdir chroot-debian
```   

Create a base system

```bash
sudo debootstrap sid chroot-debian http://deb.debian.org/debian
```   

Valid names `sid`, `stable` or any debian code name

Mount  filesystems

```bash
sudo mount -o bind /dev chroot-debian/dev
sudo mount -t sysfs none chroot-debian/sys
sudo mount -o bind /proc chroot-debian/proc
```   

Optionally, copy DNS resolver configuration.

```
sudo cp /etc/resolv.conf /path/to/chroot-env/etc/resolv.conf
```

Start chrooting

```bash
sudo chroot chroot-debian /bin/bash
```

Once done, exit the session and unmount

```bash
sudo umount chroot-debian/dev chroot-debian/proc
```

#### Pass variables to chrooted environment 

```bash
chroot ./ env -i PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

# Allowing user to run a command as root

```shell
sudo visudo
```

```
artur ALL=(ALL) chroot /path/to/chroot-env
```


#### Nginx - serving files setup DAV

Full version with 3rd party extensions

```
apt install nginx-full nginx-extras
```

```
    location / {
        index nonextistent;
        autoindex on;
        autoindex_format json;
    }

	location /restricted {
		fancyindex on;
        fancyindex_exact_size off;
		auth_basic "Restricted";
	    auth_basic_user_file "/etc/nginx/.htpasswd";
	}

    location /dropbox {
        index nonextistent;
        autoindex on;
        autoindex_format json;

        dav_methods PUT DELETE MKCOL COPY MOVE;
        dav_ext_methods PROPFIND OPTIONS LOCK UNLOCK;
        dav_access user:rw group:r all:r;

        # client_max_body_size 0;
        create_full_put_path on;
        client_body_temp_path /tmp/;

		limit_except GET PROPFIND OPTIONS HEAD {
	        auth_basic "Restricted";
	        auth_basic_user_file "/etc/nginx/.htpasswd";
	    }
        # auth_pam "Restricted";  
        # auth_pam_service_name "common-auth";
    }
```

Create password

``` bash
echo -n 'sammy:' >> /etc/nginx/.htpasswd
openssl passwd -apr1 >> /etc/nginx/.htpasswd
```

Another method to set the password

```bash
htpasswd -c /etc/nginx/.htpasswd sammy
htpasswd /etc/nginx/.htpasswd another_user
```

**UI Client `Cyberduck`**

#### Backing up the entire OS

```
sudo rsync -aAXHv / --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} /mnt
```

Python

```
install python3-full
```

### Check battery

```bash
upower -i /org/freedesktop/UPower/devices/battery_BAT0
upower -i `upower -e | grep 'BAT'`
upower -i $(upower -e | grep BAT) | grep --color=never -E "state|to\ full|to\ empty|percentage"
```

Battery capacity

```bash
cat /sys/class/power_supply/BAT0/capacity
```

`apt install acpi`

acpi -V
acpi -t

Watch the status for example: 

```
watch --interval=5 acpi -V
```

### How to scan open ports 

```
nmap -sT -p- 10.10.8.8
nmap -p 80 127.0.0.1
```


### Remove non ASCII characters form the file

```bash
sed 's/[^[:print:]\t]//g' script.sh > cleaned_script.sh
```

## Network Manager

Check Status

```
pacman -S networkmanager
systemctl status NetworkManager
```

```bash
nmcli dev status
nmcli radio wifi
```

```bash
nmcli radio wifi on
```

```bash
nmcli dev wifi list
```

```bash
nmcli dev wifi connect network-ssid
```

```bash
nmcli dev wifi connect network-ssid password "network-password"

# with password promot
nmcli dev wifi connect network-ssid password "network-password"

```

```
nmcli con show
```

```
nmcli con down ssid/uuid
nmcli con up ssid/uuid
```

```
ip link set eno1 up
```

### DHCP Client

```
pacman -S dhclient dhcpcd
```

Turn dchp on for the interface

```
dhclient eth0 -v
```


### Create file

```bash
cat > filename <<- "EOF"
file contents
more contents
EOF
```


### Get data about file include number of links

```bash
stat f1
```

### File decryption 

```bash
openssl aes256 -md sha256 -d -in file.enc.zip -out file.zip -pass pass:"<password>"
```


### Base64

```bash
openssl base64 -in qrcode.png -out qrcode.png.base64
openssl base64 -in qrcode.png 
```


### tar.xz

```
tar -xJf file.tar.xz -C destination
```


## Execute command 

Execute `ls` for each each object in current directory that starts from `D` or `M` 

```
ls {D*,M*}
```

Utworzenie dwóch katalogów `test_a`, `test_b`

```
mkdir test_{a,b}
```

### copy content of directory and merge it. Dereference links

```
cp -LTr /form/ .
```

## JSON Linux apis

```
tree -J
ls -l | jq -R -s -c 'split("\n")[:-1]'
```


### List dependencies 

```bash
pacman -Sy lld
llvm-nm
```

### Find location of executable 

```
type -a python
```

### Reload each file save 

```bash
brew install fswatch
fswatch -o . --exclude "\.build.*$" | xargs -n1 -I{} your-command
```


## Add capabilities 
allow all to open 443 port

```bash 
sudo setcap 'cap_net_bind_service=+ep' ./service
```


# Test speed of the drive

```
sudo hdparam -t --direct /dev/mmcblk0
```