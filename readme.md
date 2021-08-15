Useful links:

https://wiki.archlinux.org/index.php/installation_guide
https://wiki.archlinux.org/index.php/VirtualBox/Install_Arch_Linux_as_a_guest#Install_the_Guest_Additions
https://zalinux.ru/?p=1860
https://www.youtube.com/watch?v=_3-OMUQTf_k&t=734s (up to 3:20)
https://www.youtube.com/watch?v=a00wbjy2vns // full installation walkthrough
https://www.youtube.com/watch?v=KN7bBcfIhJk&list=PLlloYVGq5pS6VC6mT9yj7d4lwoYqyt4SL // arch installation playlist
https://www.youtube.com/watch?v=fV17DgsQcsY

Arch maintenance:
https://www.youtube.com/watch?v=wwSkFi3h2nI

1) prepare ISO (get from yandex mirror)
2) Arch Linux installation:
    2.1) Connect to the Network
        # ip addr show // will show ip if wired connected (inet)
        https://qna.habr.com/q/856469 // wifi-case; to access wireless network, use iwd(утилита iwctl) will show available networks; choose network
        # ping www.google.com // check the internet connection

    2.2) Take a look at mirrorlist file. The higher a mirror is placed in the list, the more priority it is given when downloading a package.
         (Recommended after installation) If you have problems with mirrors and download speed is poor, go to archlinux.org/mirrorlist/
         select your country, select 'mirror status' checkbox and generate new mirrorlist.
        # nano /etc/pacman.d/mirrorlist

    2.3) Check package databases
        # packman -Syyy

    2.4) Create disk partitions
        # fdisk -l // shows all disks, find disk you want to work with
                   // # lsblk - same as fdisk -l
        # fdisk /dev/[your disk]

        // create new partition table
        // enter 'g' for GPT
        // enter 'p' to preview changes

        [Partition for EFI]
        // enter 'n' for new partition
        // partition number - default 1 enter
        // first sector - default enter
        // last sector - '+xxxM' (300-500M recommended)
        // remove the signature - Yes
        
        Deside if use SWAP partition or SWAP file (for swap partition 100+50% of RAM size; 16+8)
        With swapfile, you can safely resize it by deleting and creating new swapfile

        [Partition for root]
        // enter 'n' for new partition
        // partition number - default 2 enter
        // first sector - default enter
        // last sector - '+xxxM' (60-100G recommended)

        [Partition for home] // separate partition for home is needed to save /home data during arch-linux re-installation;
                              // choose not to format home partition during re-installation
        // enter 'n' for new partition
        // partition number - default 3 enter
        // first sector - default enter
        // last sector - default enter; takes the remainder of hard disk

        // enter 'w' to write and finalize all the changes
        
    2.5) Format partitions
        [Format EFI partition]
        # mkfs.fat -F32 /dev/xxx1 (sda or nvme0n1)

        [Format root partition]
        # mkfs.ext4 /dev/xxx2

        [Format home partition]
        # mkfs.ext4 /dev/xxx3

    2.6) Mount partitions
        [Mount root partition]
        # mount /dev/xxx2 /mnt

        [Mount home partition]
        # mkdir /mnt/home
        # mount /dev/xxx3 /mnt/home

        # df -h // shows mounted partitions; also easy way to check available disk space

        [Create fstab file]
        # mkdir /mnt/etc
        # genfstab -U /mnt >> /mnt/etc/fstab 
        # cat /mnt/etc/fstab // check fstab file if all mounted correctly; fstab file - in UNIX systems describes how disk works
        
    2.7) Installation
        # pacstrap -i /mnt base linux-firmware // essential packages
        # arch-chroot /mnt // connect to the in-progress installation to finish it
        # pacman -S linux-lts linux-lts-headers // install linux kernel

        # pacman -S base-devel // package for building aur-packages manually

        # pacman -S nano // there is no text editor available in in-progress installation process, so we need to install one
        # pacman -S networkmanager network-manager-applet wireless_tools netctl // packages for network managing
        # pacman -S dialog // A tool to display dialog boxes from shell scripts; used for wifi-menu

        // enable networkmanager; systemctl enable [APP] - to start [APP] on every boot
        # systemctl enable NetworkManager

        [Generate initial RAM disk for the linux kernel (initramfs)]
        # mkinitcpio -p linux-lts

        [Configure locale settings]
        # nano /etc/locale.gen // find your locale and uncomment it (en_US.UTF-8)
        # locale-gen
        # localectl set-locale LANG=en_US.UTF-8 // changes /etc/locale.conf
        # timedatectl list-timezones
        # timedatectl set-timezone Zone/SubZone

        # passwd // create pasword for root user
        # useradd -m -g users -G wheel [USERNAME] // create new user; wheel - gives access to sudo
        # passwd [USERNAME] // set password for created user

        # which sudo // get access to sudo command; if not installed - pacman -S sudo
                     // to give sudo acces to the user: # nano /etc/sudoers - uncomment %wheel ALL = (ALL) ALL


        [Set up GRUB]
        # pacman -S grub efibootmgr dosfstools os-prober mtools    
        # mkdir /boot/EFI // create directory for EFI
        # mount /dev/xxx1 /boot/EFI // mount EFI partition
        # grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
        # mkdir /boot/grub/locale 
        # cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
        # grub-mkconfig -o /boot/grub/grub.cfg

        To hide grub menu at startup:
        # sudo nano /etc/default/grub
        # GRUB_TIMEOUT=0
        # grub-mkconfig -o /boot/grub/grub.cfg

        [Create a swapfile]     
        # fallocate -l 8G /swapfile 
        # chmod 600 /swapfile
        # mkswap /swapfile
        # cp /etc/fstab /etc/fstab.bak
        
        # echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
        # cat /etc/fstab // check fstab if swapfile was created properly

        [Additional packages]
        # pacman -S intel-ucode
        # pacman -S xorg-server xorg-xinit
        # pacman -S xf86-video-intel libgl mesa // drivers for video-card; dont install on virtualbox installation
        # pacman -S virtualbox-guest-utils xf86-video-vmware mesa // only for virtualbox archlinux installation

        [Finish installation]
        # exit
        # unmount -a
        # reboot

    2.8) Install vim (neovim). https://guides.hexlet.io/vim/
        # pacman -S neovim // neovim is fork of vim with improved functionality

    2.9) Set up terminal

        - Working with fonts:
        https://wiki.archlinux.org/index.php/Font_configuration_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)
        https://wiki.archlinux.org/index.php/Fonts_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)
        https://christitus.com/emoji/

        Noto emoji charmap:
        https://www.google.com/get/noto/help/emoji/

        pacman -S noto-fonts-emoji

        https://emojipedia.org/ - copy needed emojis

        https://www.youtube.com/watch?v=LJ7CEhnS0OM (basic working with fonts)
        https://www.youtube.com/watch?v=DK6aun1FbhM (fontsareawesome used for icons)
        https://www.youtube.com/watch?v=ny8N5mxk7aY&t=1s (how to work with emoji)
        https://www.youtube.com/watch?v=UCEXY46t3OA&t=193s        

        Fonts can be found in archlinux repositories; 
        if no found there - fontsquirrel.com; 
        try droid-sans-slashed

        # pacman -S fontconfig
        # fc-list // show all fonts in system
        # pacman -S ttf-dejavu // install font
        # pacman -S rxvt-unicode // install urxvt or rxvt-unicode-256xresources

        https://www.youtube.com/watch?v=_kjbj-Ez1vU&t=782s (urxvt tutorial)
        https://www.youtube.com/watch?v=SRXLdy-CQbI (how to configure fonts in urxvt)
        https://addy-dclxvi.github.io/post/configuring-urxvt/ (how to configure colors in urxvt)
        https://www.youtube.com/watch?v=eaBf_yFHps8&t=327s (how to configure colors in urxvt)

        // create ~/.Xresources 
        // content:

        URxvt*background: Black
        URxvt*foreground: White

        URxvt.font: xft:DejaVu Sans Mono:size=12
        URxvt.antialias: true

        URxvt.scrollBar: false

        URxvt.iso14755:        false
        URxvt.iso14755_52:     false

        URxvt.clipboard.autocopy: true
        URxvt.keysym.Shift-Control-V: eval:paste_clipboard
        URxvt.keysym.Shift-Control-C: eval:selection_to_clipboard

        To configure terminal colors use terminal.sexy
        https://stackoverflow.com/questions/29447692/whats-the-meaning-of-color0-15-in-urxvt-settings
        https://upload.wikimedia.org/wikipedia/commons/1/15/Xterm_256color_chart.svg

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

    - add the following line to .xinitrc (it must be the first line)
    [[ -f ~/.Xresources ]] && xrdb -merge -I$HOME ~/.Xresources

    If using bash:
        - To customize colors for user, host and directory info in terminal use .bashrc:

        https://askubuntu.com/questions/123268/changing-colors-for-user-host-directory-information-in-terminal-command-prompt
        https://wiki.archlinux.org/index.php/Bash/Prompt_customization
        https://tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html
        http://bashrcgenerator.com/

        - To use 256 colors:
        https://unix.stackexchange.com/questions/124407/what-color-codes-can-i-use-in-my-ps1-prompt

        - To customize colors for user, host and directory info in terminal use .bashrc:

        #
        # ~/.bashrc
        #

        # If not running interactively, don't do anything
        [[ $- != *i* ]] && return

        # Using 256 colors:
        # export PS1="\[\033[38;5;046m\]\u@\h\[\033[00m\]:\[\033[38;5;069m\]\w\[\033[00m\]\$ "

        alias ls='ls --color=auto'
        export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "

        - Change cursor to blinking bar, add to .bashrc:
        echo -e -n '\x1b[\x35 q'

    If using zsh:
        # pacman -S zsh zsh-completions 
        # zsh // initial setup; check 1 (defaults), check 2 (choose 1), check 3 (choose vi), check 4 (defaults); hit 0 to save

        Install oh-my-zsh:
        # pacman -S curl git wget
        # sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" // installs oh-my-zhs, previous .zshrc becomes .zshrc.pre-oh-my-zsh
    
        - in .zshrc change THEME to simple, change cursor to blinking bar
        - to upgrade PROMPT (in your theme) read zshmisc man: 
        https://linux.die.net/man/1/zshmisc
        http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html#Visual-effects

        - simple.zsh-theme:
        
        NEWLINE=$'\n'

        PROMPT='%{$fg_bold[green]%}%m@%n %{$fg_bold[blue]%}%~%{$fg[yellow]%}$(git_prompt_info)%{$reset_color%}\
        ${NEWLINE}%{${fg_bold[cyan]}%}»%{${reset_color}%} '

        ZSH_THEME_GIT_PROMPT_PREFIX=" ("
        ZSH_THEME_GIT_PROMPT_SUFFIX=")"

        - to udgrage zsh:
        # upgrade_oh_my_zsh
        
    2.10) 
    
    - Customize user startup settings using .bash_profile (bash reads .bash_profile, .bash_login, .profile)

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

    - Customize user startup settings using .zprofile (zsh reads .zprofile, .profile)

    # startx after login in tty1
    if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
        startx
    fi


    - Export variables in .zshenv:
    https://unix.stackexchange.com/questions/71253/what-should-shouldnt-go-in-zshenv-zshrc-zlogin-zprofile-zlogout

    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
    export PATH=$PATH:$JAVA_HOME/bin

    export M2_HOME=/opt/apache-maven-3.6.3
    export PATH=$PATH:$M2_HOME/bin

2.11) Skip grub menu: https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks

3) Pacman usage (short pacman tutorial - https://www.youtube.com/watch?v=-dEuXTMzRKs&t=851s)
    # pacman -S package-name // install package
    # pacman -R package-name // remove package
    # pacman -Rs package-name // remove package and all its dependencies that are not used anymore (-Rns uninstalls config files also)
    # pacman -Sy // check repositories for updates
    # pacman -Su // update installed packages
    # pacman -Syu // update main repositories and update all packages installed using pacman (from main repo); recommended to run it once a week
    # pacman -Ss package-name // search for package
    # pacman -Q // lists all packages installed on computer
    # pacman -Qe // lists packages installed explicitly by user
    # pacman -Qm // lists packages installed from AUR
    # pacman -Qdt // lists unused dependencies
    # pacman -Rns $(pacman -Qdt) // remove all unused packages installed from main repo
    # pacman -U xxx.tar.xz // install tar package

    After updating packages, old versions of packages (that you will probably never install again) stay on machine. 
    # pacman -Sc // remove obsolete packages installed from main repos (packages that are stored in cache and not installed)

    Check size of user cache folder:
    du -sh .cache/
    rm -rf .cache/* - remove everything inside directory

    Check size of system logs:
    du -sh /var/log/journal/
    sudo journalctl --vacuum-time=2weeks - will leave in journal only logs for last two weeks

    Update mirrorlist file to have better package downloading speed:
    pacman -S reflector
    sudo reflector -c Russia -l 10 --sort rate --save /etc/pacman.d/mirrorlist - will take 10 most recently updated mirrors from Russia and sort it

4) AUR usage
    - go to archlinux.org -> AUR
    - find package
    - download snapshot (xxx.tar.gz)
    - extract archive
    - move to created directory with PKGBUILD file
    - # makepkg -s // creates the package (xxx.pkg.tar.xz)
    - # sudo pacman -U xxx.pkg.tar.gz // install created package

    - to simplify downloading package from AUR, helper can be used
    - # sudo pacman -S git
    - # cd ~/Downloads
    - # git clone https://aur.archlinux.org/yay.git
    - # cd yay
    - # makepkg -si
    - # sudo pacman -U yay-xxx.pkg.tar.xz
    - # yay -S package-name // in diff section usually choose N
    - # yay -Rns package-name // remove package
    - # yay -Syu // update AUR repositories and update all packages installed from AUR
    - # yay -Sc // remove uninstalled AUR packages from cache
    - # yay -Yc // remove all unwanted (unused) dependencies from AUR

5) Set up i3wm
    # pacman -S i3
    // add 'exec i3' to ~/.xinitrc
    # startx // start xserver

    App list (https://habr.com/ru/post/515750/):
    - terminal: urxvt, 
    - text editor: nvim, vscode
    - app search: rofi,
    - file manager: ranger, 
    - notifications: dunst, 
    - shell: zsh with oh-my-zsh, dircolors
    - text editor: neovim,
    - web browser: google-chrome
    - messenger: telegram
    - screenshots: scrot, spectacle
    - sound control: pavucontrol
    - video player: vlc
    - music: mocp
    - document reader: zathura
    - pictures viewer: feh 
    - backup tool: rsync
    - keyboard control: obins kit (for anne pro 2)  

    # pacman -S rofi
    Rofi colors can be managed by .Xresources
    add sudo rights to rofi: 
    # bindsym $mod+d exec --no-startup-id "rofi -show drun -run-command 'gksudo {cmd}'"

    rofi shows apps defined in /usr/share/applications and ~/.local/share/applications

    - set up GTK (GTK is used for creating GUIs)
    # pacman -S lxappearance-gtk3 / GTK configurer
    # pacman -S materia-gtk-theme
    # pacman -S papirus-icon-theme 

    - set up login screen (optional)
    # pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
    # systemctl enable lightdm 
    # sudo lightdm-gtk-greeter-settings

    - set up audo:
    # pacman -S pulseaudio
    # pacman -S pavucontrol // optional

    - set up video player:
    # pacman -S vlc

    - set up pictures viewer:
    # pacman -S feh

6) Java development setup:

    6.1) openjdk
    6.2) git
    6.3) maven
        - download xxx.bin.tar.gz (http://maven.apache.org/download.cgi)
        - cd /opt
        - sudo tar -xvzf ~/Downloads/apache-maven-3.6.3-bin.tar.gz
    6.4) docker
    6.5) kubernetes
    6.6) postman
    6.7) vscode
    6.8) Install Intellij Idea Ultimate:
    https://www.javahelps.com/2015/04/install-intellij-idea-on-ubuntu.html
    https://linuxhint.com/install_jetbrains_intellij_ubuntu/

    font: Menlo

    start Idea as root, go to 'tools -> create command line launcher'
    sudo chmod ugo+rwx usr/local/bin/idea

    create usr/share/applications/intellij.desktop to show idea in rofi:
    [Desktop Entry]
    Name=Intellij Idea Ultimate
    GenericName=IDE
    Exec=/opt/idea/bin/idea.sh
    Icon=/opt/idea/bin/idea.png
    Type=Application
    StartupNotify=false
    Categories=Utility;TextEditor;Development;IDE;
    Actions=new-empty-window;

7) Wi-fi connection:
    
    https://wiki.archlinux.org/index.php/Network_configuration_

    # ip link // check if wireless interface was created; wireless interfaces start from letter 'w'
    # iwconfig // another way to check wireless interface
    If no wireless interface found, find wifi device:
    # lspci // find wireless network controller, install firmware for it or install linux-firmware package if it was not installed
    # lspci -k // check if kernel loaded driver for wireless card
    
    If all checks passed, connect to wi-fi with network-manager-applet or use nmcli in a command-line
    
    If connection with wifi-menu failed:

    sudo systemctl disable dhcpcd.service // check exact name of dhcpcd.service with systemctl
    sudo systemctl disable dhcpcd@.service
    sudo systemctl stop dhcpcd.service
    sudo systemctl stop dhcpcd@.service

    sudo systemctl enable NetworkManager.service
    sudo systemctl enable wpa_supplicant.service

    gpasswd -a USERNAME network

    ip link set down eth0 // check 'ip link' to see actual names of eth0 and wlan0
    ip link set down wlan0

    sudo systemctl start wpa_supplicant.service
    sudo systemctl start NetworkManager.service

    sudo systemctl reboot

    systemctl // shows all running services

8) Backups with rsync:
    https://www.youtube.com/watch?v=G2gbun8LEC4
    https://wiki.archlinux.org/index.php/Rsync
    https://ostechnix.com/backup-entire-linux-system-using-rsync/

    backup home using tar:
    sudo tar -zcvpf mnt/usbstick/backup/dmitry-backup-$(date +%d-%m-%Y).tar.gz /home/dmitry
    restore home:
    sudo tar -xvpzf /path/to/backup.tar.gz -C /home/dmitry --numeric-owner

9) Set up VPN:
    -use vpn-extension for Chrome (Touch VPN)

10) Install docker
    sudo pacman -S docker
    sudo pacman -S docker-compose

    to use docker without sudo:
    https://docs.docker.com/engine/install/linux-postinstall/

    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker 

    to run docker daemon on system startup:
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service

11) Working with USB:
    lsblk // find your usb-stick
    sudo mkdir /mnt/usbstick // if not created
    sudo mount /dev/sdb1 /mnt/usbstick
    sudo umount /mnt/usbstick
    df -H // check available disk space
    du -h -d 1 // estimate file space usage - shows space used under a particular directory (-h - human readable form, -d 1 - depth=1)

12) Git basics:

git init
git add . 
git commit -m "init commit"
git remote add origin https://...
git push origin master

git remote -v // view remotes
git remote set-url origin https://github.com/sumtsov/book-library-service.git

13) Working with torrents

Transmision:
$ yay -S transmission-gtk

aria2:
https://aria2.github.io/
https://wiki.archlinux.org/title/aria2
https://www.addictivetips.com/ubuntu-linux-tips/download-torrents-from-the-command-line-linux/

$ sudo pacman -S aria2
$ aria2c --dir [download-location] [torrent-file-location]

sudo find . -name "name"
sudo find . -name "name" -delete

14) Tomcat installation:

download .tar.gz from Binary Distributions - Core 
https://tomcat.apache.org/download-90.cgi

create folder:
/opt/apache-tomcat-{version}

unpack archive:
sudo tar xzvf apache-tomcat-*tar.gz -C /opt/tomcat --strip-components=1

chmod 755 -R {your-tomcat-dir} // lets Intellij IDEA manage your Tomcat Server

Create Tomcat startup configuration in IDEA:
- File -> Settings -> Application Servers -> + (add your Tomcat directory)
- Edit configurations -> + -> Tomcat Server -> Local -> Deployment -> + -> *:war exploded

add Web Descriptor in Intellij:
- Project Settings -> Facets -> + -> Web -> {select module for web descriptor adding} -> {add/remove Web Module Deployment Descriptor/Web Resource Directory}