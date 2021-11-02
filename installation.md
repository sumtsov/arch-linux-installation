# Arch Linux Installation and Configuration Guide

## Table of Contents
1. [Pre Installation](#pre-installation)\
    1.1. [Prepare ISO](#prepare-iso)\
    1.2. [Connect to the Network](#connect-to-the-network)\
    1.3. [Select Mirror](#select-mirror)\
    1.4. [Check package databases](#check-package-databases)\
    1.5. [Create disk partitions](#create-disk-partitions)\
    1.6. [Format partitions](#format-partitions)
2. [Installation](#installation)\
    2.1. [Configure locale settings](#configure-locale-settings)\
    2.2. [Set Up users](#set-up-users)\
    2.3. [Set Up GRUB](#set-up-grub)\
    2.4. [Create a swapfile](#create-a-swapfile)\
    2.5. [Additional packages](#additional-packages)\
    2.6. [Finish installation](#finish-installation)
3. [Configuration](#configuration)\
    3.1. [Install](#install)\
    3.2. [Set Up fonts](#set-up-fonts)\
    3.3. [Set Up terminal](#set-up-terminal)   
    3.4. [Set Up bash](#set-up-bash)\
    3.5. [Set Up zsh](#set-up-zsh)\
    3.6. [Customize user startup settings](#customize-user-startup-settings)\
    3.7. [Rename user directories](#rename-user-directories)\
    3.8. [Set Up Wi-Fi connection](#set-up-wi-fi-connection)
4. [System backup](#system-backup)
5. [Useful links](#useful-links) 

## 1. Pre Installation <a name="pre-installation"></a>
### 1.1. Prepare ISO <a name="prepare-iso"></a>
[Download the image](https://mirror.yandex.ru/archlinux/iso/)

### 1.2. Connect to the Network <a name="connect-to-the-network"></a>
The following command will show ip if you have a wired internet conection:\
`# ip addr show`\
If you need a wifi-connection, use [iwd (iwctl utility)](https://wiki.archlinux.org/title/Iwd#iwctl)\
Check the internet connection:\
`# ping www.google.com`

### 1.3. Select Mirror <a name="select-mirror"></a>
Take a look at mirrorlist file. The higher a mirror is placed in the list, the more priority it is given when downloading a package. If you have problems with mirrors and download speed is poor (also recommended to do it after installation), go to archlinux.org/mirrorlist/, select your country, select 'mirror status' checkbox and generate new mirrorlist. Then paste generated list to the mirrorlist file:\
`# nano /etc/pacman.d/mirrorlist`

### 1.4. Check package databases <a name="check-package-databases"></a>
`# packman -Syyy`

### 1.5. Create disk partitions <a name="create-disk-partitions"></a>
The following command shows all disks. `fdisk -l` is the same as `lsblk`.\
Find the disk you want to work with:\
`# fdisk -l`
`# fdisk /dev/[your disk]`

**Create new partition table:**
- Enter 'g' for GPT
- Enter 'p' to preview changes

**Create Partition for EFI:**
- Enter 'n' for new partition
- Partition number - default 1 enter
- First sector - default enter
- Last sector - '+xxxM' (300-500M recommended)
- Remove the signature - Yes

Deside if use SWAP partition or SWAP file (for swap partition 100+50% of RAM size; 16+8)\
With swapfile, you can safely resize it by deleting and creating new swapfile

**Create Partition for root:**
- Enter 'n' for new partition
- Partition number - default 2 enter
- First sector - default enter
- Last sector - '+xxxM' (60-100G recommended)

**Create Partition for home:**
Separate partition for home is needed to save /home data during arch-linux re-installation;
Choose not to format home partition during re-installation

- Enter 'n' for new partition
- Partition number - default 3 enter
- First sector - default enter
- Last sector - default enter; takes the remainder of hard disk

Enter 'w' to write and finalize all the changes

### 1.6. Format partitions <a name="format-partitions"></a>

**Format EFI partition:**\
`# mkfs.fat -F32 /dev/xxx1`

**Format root partition**\
`# mkfs.ext4 /dev/xxx2`

**Format home partition**\
`# mkfs.ext4 /dev/xxx3`

### 1.7. Mount partitions
        
**Mount root partition**\
`# mount /dev/xxx2 /mnt`

**Mount home partition**\
`# mkdir /mnt/home`\
`# mount /dev/xxx3 /mnt/home`

The following command shows mounted partitions, its also an easy way to check available disk space:\
`# df -h`

**Create fstab file**\
fstab file - in UNIX systems describes how disk works

`# mkdir /mnt/etc`\
`# genfstab -U /mnt >> /mnt/etc/fstab`\
Check fstab file if all mounted correctly:\
`# cat /mnt/etc/fstab`

## 2. Installation <a name="installation"></a>

Install essential packages:\
`# pacstrap -i /mnt base linux-firmware`

Connect to the in-progress installation to finish it:\
`# arch-chroot /mnt`

Install Linux kernel:\
`# pacman -S linux-lts linux-lts-headers`

Package for building aur-packages manually:\
`# pacman -S base-devel`

There is no text editor available in in-progress installation process, so we need to install one:\
`# pacman -S nano`

Packages for network managing:\
`# pacman -S networkmanager network-manager-applet wireless_tools netctl`

A tool to display dialog boxes from shell scripts, used for wifi-menu:\
`# pacman -S dialog`

Enable networkmanager:\
`# systemctl enable NetworkManager`\
`# systemctl enable [APP]` - to start [APP] on every boot

Generate initial RAM disk for the linux kernel (initramfs):\
`# mkinitcpio -p linux-lts`

### 2.1. Configure locale settings <a name="configure-locale-settings"></a>

Find your locale and uncomment it (en_US.UTF-8):\
`# nano /etc/locale.gen`\
`# locale-gen`\
`# localectl set-locale LANG=en_US.UTF-8` - changes /etc/locale.conf\
`# timedatectl list-timezones`\
`# timedatectl set-timezone Zone/SubZone`

### 2.2. Set Up users <a name="set-up-users"></a>

Create password for root user:\
`# passwd`

Create new user (wheel - gives access to sudo):\
`# useradd -m -g users -G wheel [USERNAME]`

Set password for created user:\
`# passwd [USERNAME]`

Get access to sudo command, if not installed - pacman -S sudo:\
`# which sudo`

To give sudo acces to the user: `# nano /etc/sudoers` - uncomment `%wheel ALL = (ALL) ALL`

### 2.3. Set Up GRUB <a name="set-up-grub"></a>

`# pacman -S grub efibootmgr dosfstools os-prober mtools`\
`# mkdir /boot/EFI // create directory for EFI`\
`# mount /dev/xxx1 /boot/EFI // mount EFI partition`\
`# grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`\
`# mkdir /boot/grub/locale`\ 
`# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`\
`# grub-mkconfig -o /boot/grub/grub.cfg`

To hide grub menu at startup:\
`# sudo nano /etc/default/grub`\
`# GRUB_TIMEOUT=0`\
`# grub-mkconfig -o /boot/grub/grub.cfg`

### 2.4. Create a swapfile <a name="create-a-swapfile"></a>

`# fallocate -l 8G /swapfile`\ 
`# chmod 600 /swapfile`\
`# mkswap /swapfile`\
`# cp /etc/fstab /etc/fstab.bak`\
`# echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab`

Check fstab if swapfile was created properly:
`# cat /etc/fstab`

### 2.5. Additional packages <a name="additional-packages"></a>

`# pacman -S intel-ucode`\
`# pacman -S xorg-server xorg-xinit`\
Drivers for video-card, don't install on virtualbox installation:\
`# pacman -S xf86-video-intel libgl mesa`\
Only for virtualbox installation:\
`# pacman -S virtualbox-guest-utils xf86-video-vmware mesalinux installation`

### 2.6. Finish installation <a name="finish-installation"></a>
        
`# exit`
`# unmount -a`
`# reboot`

## 3. Configuration <a name="configuration"></a>

### 3.1. Install [vim](https://guides.hexlet.io/vim/) (neovim) <a name="install-vim"></a>

`# pacman -S neovim`

### 3.2. Set Up fonts <a name="set-up-fonts"></a>

Font configuration:\
https://wiki.archlinux.org/index.php/Font_configuration_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)\
https://wiki.archlinux.org/index.php/Fonts_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)\

Noto emoji charmap:\
https://www.google.com/get/noto/help/emoji/

Needed emojis can be found here:\
https://emojipedia.org/

[Fonts configuration tutorial](https://www.youtube.com/watch?v=LJ7CEhnS0OM)\
[Fontsareawesome for icons configuration](https://www.youtube.com/watch?v=DK6aun1FbhM)\
[How to work with emoji #1](https://www.youtube.com/watch?v=ny8N5mxk7aY&t=1s)\
[How to work with emoji #2](https://www.youtube.com/watch?v=UCEXY46t3OA&t=193s)\
[How to work with emoji #3](https://christitus.com/emoji/)

Install fontconfig:\
`# pacman -S fontconfig`\
Show all fonts in system:\
`# fc-list`\
Install fonts:\
`# pacman -S ttf-dejavu`\ 
`# pacman -S noto-fonts-emoji`

### 3.3. Set Up terminal <a name="set-up-terminal"></a>

Install urxvt or rxvt-unicode-256xresources:\
`# pacman -S rxvt-unicode`

[Urxvt tutorial](https://www.youtube.com/watch?v=_kjbj-Ez1vU&t=782s)\
[How to configure fonts in urxvt](https://www.youtube.com/watch?v=SRXLdy-CQbI)\
[How to configure colors in urxvt #1](https://addy-dclxvi.github.io/post/configuring-urxvt/)\
[How to configure colors in urxvt #2](https://www.youtube.com/watch?v=eaBf_yFHps8&t=327s)

To configure terminal colors use [terminal.sexy](https://terminal.sexy/)

[Xterm-256color chart](https://upload.wikimedia.org/wikipedia/commons/1/15/Xterm_256color_chart.svg)\
[What's the meaning of color(0-15) in urxvt settings?](https://stackoverflow.com/questions/29447692/whats-the-meaning-of-color0-15-in-urxvt-settings)

Create ~/.Xresources file with the following content:

    URxvt*background:             Black
    URxvt*foreground:             White

    URxvt.font:                   xft:DejaVu Sans Mono:size=12
    URxvt.antialias:              true

    URxvt.scrollBar:              false

    URxvt.iso14755:               false
    URxvt.iso14755_52:            false

    URxvt.clipboard.autocopy:     true
    URxvt.keysym.Shift-Control-V: eval:paste_clipboard
    URxvt.keysym.Shift-Control-C: eval:selection_to_clipboard

    ! black dark/light
    URxvt*color0  : #080808
    URxvt*color8  : #121212
    ! red dark/light
    URxvt*color1  : #ff0000
    URxvt*color9  : #ff5f00
    ! green dark/light
    URxvt*color2  : #00af00
    URxvt*color10 : #00d700
    ! yellow
    URxvt*color3  : #dfaf00
    URxvt*color11 : #dfdf00
    ! blue dark/light
    URxvt*color4  : #8700ff
    URxvt*color12 : #875fff
    ! magenta dark/light
    URxvt*color5  : #af00af
    URxvt*color13 : #af5faf
    ! cyan dark/light
    URxvt*color6  : #5fafd7
    URxvt*color14 : #5fd7d7
    ! white dark/light
    URxvt*color7  : #eeeeee
    URxvt*color15 : #e4e4e4

Add the following line to .xinitrc (it must be the first line):\
`[[ -f ~/.Xresources ]] && xrdb -merge -I$HOME ~/.Xresources`

### 3.4. Set Up bash <a name="set-up-bash"></a>

To customize colors for user, host and directory info in bash, add the following code to .bashrc:

    #
    # ~/.bashrc
    #

    # If not running interactively, don't do anything
    [[ $- != *i* ]] && return

    # Using 256 colors:
    # export PS1="\[\033[38;5;046m\]\u@\h\[\033[00m\]:\[\033[38;5;069m\]\w\[\033[00m\]\$ "

    alias ls='ls --color=auto'
    export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "

To change cursor to blinking bar, add to .bashrc:\
`echo -e -n '\x1b[\x35 q'`

[How to change colors for user, host and directory](https://askubuntu.com/questions/123268/changing-colors-for-user-host-directory-information-in-terminal-command-prompt)\
[Prompt customization#1](https://wiki.archlinux.org/index.php/Bash/Prompt_customization)\
[Prompt customization#2](https://tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html)\
[.bashrc generator](http://bashrcgenerator.com/)

### 3.5. Set Up zsh <a name="set-up-zsh"></a>

Install zsh:\
`# pacman -S zsh zsh-completions`\
Start initial zsh setup:\
`# zsh`

- Check 1: defaults
- Check 2: choose 1 
- Check 3: choose vi
- Check 4: defaults
- Hit 0 to save

To udgrage zsh:
`# upgrade_oh_my_zsh`

Install oh-my-zsh:\
`# pacman -S curl git wget`\
`# sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`\
After oh-my-zhs installation, previous .zshrc becomes .zshrc.pre-oh-my-zsh

In .zshrc change THEME to 'simple', change CURSOR to 'blinking bar'

To upgrade PROMPT in your theme read zshmisc man:\
https://linux.die.net/man/1/zshmisc\
http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Visual-effects

Install [spaceship zsh-theme](https://github.com/spaceship-prompt/spaceship-prompt)

Custom zsh theme:
        
    NEWLINE=$'\n'

    PROMPT='%{$fg_bold[green]%}%m@%n %{$fg_bold[blue]%}%~%{$fg[yellow]%}$(git_prompt_info)%{$reset_color%}\
    ${NEWLINE}%{${fg_bold[cyan]}%}»%{${reset_color}%} '

    ZSH_THEME_GIT_PROMPT_PREFIX=" ("
    ZSH_THEME_GIT_PROMPT_SUFFIX=")"

### 3.6. Customize user startup settings <a name="customize-user-startup-settings"></a>

[Skip GRUB menu](https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks)

If using bash (bash reads .bash_profile, .bash_login, .profile), add the following code to .bash_profile:

    [[ -f ~/.bashrc ]] && . ~/.bashrc

    # startx after login in tty1
    if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
        startx
    fi

    export EDITOR="vim"
    export TERMINAL="urxvt"
    export BROWSER="google-chrome-stable"

    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
    export PATH=$PATH:$JAVA_HOME/bin

    export M2_HOME=/opt/apache-maven-3.6.3
    export PATH=$PATH:$M2_HOME/bin

If using zsh (zsh reads .zprofile, .profile), add the following code to .zprofile:

    # startx after login in tty1
    if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
        startx
    fi

and to .zshenv:

    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
    export PATH=$PATH:$JAVA_HOME/bin

    export M2_HOME=/opt/apache-maven-3.6.3
    export PATH=$PATH:$M2_HOME/bin

[What should/shouldn't go in .zshenv, .zshrc, .zlogin, .zprofile, .zlogout](https://unix.stackexchange.com/questions/71253/what-should-shouldnt-go-in-zshenv-zshrc-zlogin-zprofile-zlogout)   

### 3.7. Rename user directories <a name="rename-user-directories"></a>

Download xdg user directories:\
`# sudo pacman -S xdg-user-dirs`\
Create xdg config files:\
`# xdg-user-dirs-update`

Add the following code to .zshenv:
    
    # XDG Paths
    export XDG_DATA_HOME=${XDG_DATA_HOME:="$HOME/.local/share"}
    export XDG_CACHE_HOME=${XDG_CACHE_HOME:="$HOME/.cache"}
    export XDG_CONFIG_HOME=${XDG_CONFIG_HOME:="$HOME/.config"}

Under /home directory create standard user directories with your custom names
(Desktop, Downloads, Templates, Public, Documents, Music, Pictures, Videos)

In user-dirs.dirs file set created names of standard diretories:

    XDG_DESKTOP_DIR="$HOME/desktop"
    XDG_DOWNLOAD_DIR="$HOME/downloads"
    XDG_TEMPLATES_DIR="$HOME/templates"
    XDG_PUBLICSHARE_DIR="$HOME/public"
    XDG_DOCUMENTS_DIR="$HOME/documents"
    XDG_MUSIC_DIR="$HOME/music"
    XDG_PICTURES_DIR="$HOME/pictures"
    XDG_VIDEOS_DIR="$HOME/videos"

https://www.youtube.com/watch?v=tnspdGZGEPE\
https://wiki.archlinux.org/title/XDG_user_directories 

### 3.8. Set Up Wi-Fi connection <a name="set-up-wi-fi-connection"></a>

https://wiki.archlinux.org/index.php/Network_configuration_

Check if wireless interface was created; wireless interfaces start from letter 'w':\
`# ip link`\
Another way to check wireless interface:\
`# iwconfig`\
If no wireless interface found, find wireless network controller:\
`# lspci`\
Install firmware for it or install linux-firmware package if it was not installed;\
Check if kernel loaded driver for wireless card:\
`# lspci -k`
    
If all checks passed, connect to wi-fi with network-manager-applet or use nmcli in a command-line

### 4. System backup <a name="system-backup"></a>

Backup home using tar:\
`# sudo tar -zcvpf mnt/usbstick/backup/dmitry-backup-$(date +%d-%m-%Y).tar.gz /home/[USERNAME]`
Restore home using tar:\
`# sudo tar -xvpzf /path/to/backup.tar.gz -C /home/dmitry --numeric-owner`

Using rsync:\
https://www.youtube.com/watch?v=G2gbun8LEC4\
https://wiki.archlinux.org/index.php/Rsync\
https://ostechnix.com/backup-entire-linux-system-using-rsync/

### 5. Useful links <a name="useful-links"></a>

https://wiki.archlinux.org/index.php/installation_guide\
https://wiki.archlinux.org/index.php/VirtualBox/Install_Arch_Linux_as_a_guest#Install_the_Guest_Additions\
https://zalinux.ru/?p=1860\
https://www.youtube.com/watch?v=_3-OMUQTf_k&t=734s - up to 3:20\
https://www.youtube.com/watch?v=a00wbjy2vns - full installation walkthrough\
https://www.youtube.com/watch?v=KN7bBcfIhJk&list=PLlloYVGq5pS6VC6mT9yj7d4lwoYqyt4SL - arch installation playlist\
https://www.youtube.com/watch?v=fV17DgsQcsY\
https://www.youtube.com/watch?v=wwSkFi3h2nI - arch maintenance\
https://wiki.archlinux.org/title/XDG_user_directories