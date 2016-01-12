# Arch Pi 2 Setup #
_____

# Arch Linux Tweaks: #

## Disable tmpfs for /tmp since our RAM is limited: ##

    sudo systemctl stop tmp.mount
    sudo systemctl disable tmp.mount

## Setup swapfile: ##

    fallocate -l 1024M /swapfile
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile
    echo 'vm.swappiness=1' > /etc/sysctl.d/99-sysctl.conf

## Edit fstab and add the following 2 lines: ##

    /dev/root  /  ext4  noatime,discard,async  0  0
    /swapfile none swap defaults 0 0

## Network Speed Tweaks: ##

    echo -e "net.core.rmem_max = 16777216\nnet.core.wmem_max = 4194304" > /etc/sysctl.d/60-net_buffer.conf

## Disable IPv6: ##

    echo net.ipv6.conf.all.disable_ipv6 = 1 > /etc/sysctl.d/40-ipv6.conf

## Set Hostname: ##
    hostnamectl set-hostname <Insert Your Hostname Here>

## Hardware Clock and Timezone: ##

Remove /etc/localtime:

    rm /etc/localtime

Select a time zone:

    tzselect

Create the symbolic link /etc/localtime, where Zone/Subzone is the TZ value from tzselect:

    ln -s /usr/share/zoneinfo/America/Eastern /etc/localtime

Set Hardware clock to UTC:

    timedatectl set-local-rtc 0

## Pacman Tweaks: ##

    nano /etc/pacman.conf

Uncomment the following:

    CleanMethod = KeepInstalled
    Color

## Locale: ##
The Locale defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets.

    nano /etc/locale.gen

Uncomment correct Locale:

    en_US.UTF-8 UTF-8

Update Locale:

    locale-gen

Create /etc/locale.conf, where LANG refers to the first column of an uncommented entry in /etc/locale.gen:

    echo LANG=en_US.UTF-8 > /etc/locale.conf

## Set Nano as default Editor ##
    echo export=EDITOR=nano /etc/bash.bashrc

## System Update: ##

    pacman -Sy pacman
    pacman-key --init
    pacman -S archlinux-keyring
    pacman-key --populate archlinux
    pacman -Syu --ignore filesystem
    pacman -S filesystem --force

## Reboot: ##

    reboot
_________________________________________

# Wireless: #

#### Install necessary packages: ####

    pacman -S libnl wireless-regdb iw crda dialog wpa_supplicant

#### Set correct Wireless Region: ####

    nano /etc/conf.d/wireless-regdom

#### Uncomment out correct Region: ####

    WIRELESS_REGDOM="US"

### Wireless Configuration: ###

    wifi-menu

#### Enable Wifi at Boot ####

    netctl enable yourWifiSSID

----
# Setup Users: #

### Install Sudo: ###
    pacman -S sudo

Edit sudoers:

    visudo

Uncomment the following:

    %wheel ALL=(ALL) NOPASSWD: ALL

## Customize Root User: ##

Copy Bash Files to Root Home Directory

    cp /etc/skel/.bash* .

Edit Root Bash File

    nano .bashrc

Replace current PS1 with the following:
    
    PS1="\[$(tput bold)\]\[\033[38;5;9m\]\u@\[$(tput sgr0)\]\[\033[38;5;10m\]\h\[$(tput sgr0)\]\[\033[38;5;11m\]:\[$(tput sgr0)\]\[\033[38;5;69m\][\w]:\[$(tput sgr0)\]\[$(tput sgr0)\]\[\033[38;5;15m\] \[$(tput sgr0)\]"
    
## Add Admin User and Customize: ##

    useradd -m -G wheel -s /bin/bash <username>

Set Password:

    passwd <username>

Login as New User:

    su <username>

Goto Home Directory:

    cd ~

Replace Bash Prompt and add Reboot, Shutdown, Poweroff aliases:

    nano .bashrc

Add the following Aliases:

    alias reboot="sudo systemctl reboot"
    alias poweroff="sudo systemctl poweroff"
    alias halt="sudo systemctl halt"

Replace current PS1 with the following:

    PS1="\[$(tput bold)\]\u@\[$(tput sgr0)\]\[\033[38;5;10m\]\h\[$(tput sgr0)\]\[\033[38;5;11m\]:\[$(tput sgr0)\]\[\033[38;5;69m\][\w]:\[$(tput sgr0)\]\[$(tput sgr0)\]\[\033[38;5;15m\] \[$(tput sgr0)\]"

----

# Additional Arch Tweaks: #

## Install Packer: ##

    pacman -S packer

## NTP: ##

    pacman -S ntp
    systemctl enable ntpd.service
    systemctl start ntpd.service

## Install Nano Tweaks: ##

    packer -S nano-syntax-highlighting-git

Add syntax highlighting to to /etc/nanorc

    cat /usr/share/nano-syntax-highlighting/nanorc.sample >> /etc/nanorc

## Install Alsi ##

Install Alsi:


    packer -S alsi

Add Alsi to /etc/bash.bashrc:

    echo alsi >> /etc/bash.bashrc

_____

# Kodi: #

Install the following packages:

    pacman -S kodi-rbp omxplayer-git xorg-xrefresh xorg-xset xorg-server xorg-xinit udisks upower unrar lsb-release polkit wget

**Note: Select 1 for mesa-libgl and xf86-input-evdev**

Enable permission for Kodi in Xorg-Server:

    echo -e "allowed_users=anybody\nneeds_root_rights = yes" > /etc/X11/Xwrapper.config

Enable typing with a physical keyboard:

    nano /etc/udev/rules.d/raspberrypi.rules

Add the following rule:

    SUBSYSTEM=="tty", KERNEL=="tty0", GROUP="tty", MODE="0666"

Tweak Kodi systemd file so it starts after Network:

    nano /usr/lib/systemd/system/kodi.service

Add the network.target to the After line:

    After = remote-fs.target network.target

Reload systemd with new change:

    systemctl daemon-reload

Enable Kodi to start on Boot:

    systemctl enable kodi

Increase RAM to GPU:

    nano /boot/config.txt

At the very bottom edit gpu_mem:

    gpu_mem=256

Reboot:

    reboot
____

# Plymouth Boot Splash: #

Download Plymouth-lite

    git clone https://github.com/T4d3o/Plymouth-lite.git && cd Plymouth-lite
    
Compile and Install Plymouth-lite

    ./configure && make && make install

Edit Plymouth-list service:

    nano systemd/plymouth-lite-start.service

Replace ExecStart= with the following:

    ExecStart=/usr/bin/echo 0 > /sys/class/graphics/fbcon/cursor_blink ; /usr/bin/echo 0 > /sys/devices/virtual/graphics/fbcon/cursor_blink ; /usr/bin/chvt 7 ; /usr/bin/ply-image

Copy Plymouth-lite systemd to /usr/lib/systemd/system:

    cp systemd/plymouth-lite-start.service /usr/lib/systemd/system

Enable Plymouth-lite to start at boot:

    cd /usr/lib/systemd/system/sysinit.target.wants && ln -s ../plymouth-lite-start.service && systemctl daemon-reload && cd ~

Move the default Plymouth Splash Image:

    cd /usr/share/plymouth/ && mv splash.png splash.png-org

Download Kodi Splash Image:

    wget https://raw.githubusercontent.com/SchoolBusDriver/Arch-Pi2/master/Splash/splash.png

### Install mkinitcpio: ###

    pacman -S mkinitcpio

Modify /usr/lib/initcpio/init on line 35:

    nano +35 /usr/lib/initcpio/init

Add the following:

    ply-image /usr/share/plymouth/splash.png &> /dev/null

Should look like this when finished:

    if [ -n "$earlymodules$MODULES" ]; then
        modprobe -qab ${earlymodules//,/ } $MODULES
    fi

    ply-image /usr/share/plymouth/splash.png &> /dev/null

    run_hookfunctions 'run_hook' 'hook' $HOOKS
	
Modify /etc/mkinitcpio.conf:

    nano /etc/mkinitcpio.conf

Should look like this:

    # BINARIES
    # This setting includes any additional binaries a given user may
    # wish into the CPIO image.  This is run last, so it may be used to
    # override the actual binaries included by a given hook
    # BINARIES are dependency parsed, so you may safely ignore libraries
    BINARIES="ply-image"
    
    # FILES
    # This setting is similar to BINARIES above, however, files are added
    # as-is and are not parsed in any way.  This is useful for config files.
    FILES="/usr/share/plymouth/splash.png"

Create new initramfs:

    mkinitcpio -g /boot/initrd -v

Add initramfs to /boot/config.txt:

    nano /boot/config.txt

Add the following at the very bottom:

    #initramfs
    initramfs initrd 0x00f00000

Add the Linux Kernel Boot Paramaters to /boot/cmdline.txt:

    nano /boot/cmdline.txt

Add the following right after "rootwait" and before "console=ttyAMA0,115200":

    rootfstype=ext4 initrd=0x00f00000 quiet logo.nologo vt.cur_default=1

Should look like this when finished:

    root=/dev/mmcblk0p2 rw rootwait rootfstype=ext4 initrd=0x00f00000 quiet logo.nologo vt.cur_default=1 console=ttyAMA0,115200

Reboot:

    reboot

_____

# Pi 2 Speed Tweak: #

Edit /boot/config.txt

    nano /boot/config.txt

Add the following to the very bottom:

    arm_freq=1000
    sdram_freq=500
    core_freq=500
    over_voltage=2
    temp_limit=80