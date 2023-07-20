
# Aprendendo a Instalar o Archlinux

## Instalação do sistema:

_**fontes:**_

<https://archlinux.org/>

<https://wiki.archlinux.org/index.php/Installation_guide>

<https://wiki.archlinux.org/index.php/List_of_applications>

<https://imasters.com.br/devsecops/arch-linux-melhor-distro-de-todos-os-tempos>

https://medium.com/@fredlins/instala%C3%A7%C3%A3o-do-arch-linux-com-btrfs-grub-btrfs-snapper-snapper-gui-dc6eb1371f43

<https://diolinux.com.br/linux/arch-linux/como-instalar-arch-linux-tutorial-iniciantes.html#:~:text=Arch%20Linux%20para%20iniciantes%20%E2%80%93%20Como%20instalar%20o,do%20sistema.%20...%206%20Instalando%20o%20GRUB.>

<https://terminalroot.com.br/2018/07/como-instalar-facilmente-o-arch-linux.html>

### Baixar a iso e iniciar a instalação:
<http://br.mirror.archlinux-br.org/iso/latest/>

### Linha de comando

[user$] =  diretório do usuário

[root#] =  diretório root

[root#/] = diretório chroot

### Configurar o Teclado:

>root# loadkeys br-abnt2

### Configurar o idioma:

>root# nano /etc/locale.gen   (descomentar pt-br)

>root# locale-gen

>root# export LANG=pt_BR.UTF-8

### Instalar o tmux

>root# pacman -S tmux

>root# tmux

### Para recuperar a sessão caso caia a conexão com a internet.

>root>root# tmux attach


1. *Testar a rede:*

>root# ping [endereço de rede ou URL]

>root# iwctl

>root# ip link

>root# pacman - S wifi-menu ( para rede wifi)


### configurar internet:

_conectar cabo usb do smartphone em modo tethering (conexão usb)_

>root# pacman -S networkmanager net-tools

>root# systemctl enable NetworkManager

>root# systemctl start NetworkManager


### configurar wifi:

>root# nmtui

>root# ping -c3 archlinux.org


### atualizar relógio:

>root# timedatectl set-ntp true

2. *Aumentar a fonte:*

>root# pacman -Sy terminus-font

>root# setfont ter-v32n


3. *verificar BIOS se tem efi:*

>root# ls /sys/firmware/efi

>root# ls /sys/firmware/efi/efivars


4. *verificar os discos*

>root# fdisk -l

a partição será do tipo /dev/xda

### particionar o disco:


>root# fdisk /dev/sda

d

g

n

enter

enter

+512M

type efi system


n

enter

enter

+4G

type swap


n

enter

enter

+resto do disco


obs.: para bios boot, criar partição de +2M sem formatar, necessária para MBR. e adicionar flag bios legacy no x A 1 do fdisk.


### *criar partiçoes:*

>root# cfdisk /dev/xda


usando o cfdisk

new  - digite o tamanho da partição desejado.

tamanho   /  tipo

2G  /  BIOS boot ou EFI System

4G  / Linux Swap

restante xxxGB  /  Linux filesystem


apertar a tecla T para selecionar tipo de partição.


write

yes

quit


5. *formatar as partições do  disco: (se Biosboot formatar em ext4,  se em EFI formatar em btrfs)*

visualizar as partições:

>root# lsblk

*partição EFI:*

>root# mkfs.fat -F32 /dev/sda1

*partição SWAP:*

>root# mkswap /dev/sda2

*partição BTRFS:*

>root# mkfs.btrfs /dev/sda3

ou

*partição EXT4:*

>root# mkfs.ext4 /dev/sda3


6. *montar sistema de arquivos EXT4 efi:*

>root# mount /dev/sda3 /mnt

>root# swapon /dev/sda2

>root# mount --mkdir /dev/sda1 /mnt/boot/efi


- *montar as partições btrfs efi*

>root# mount /dev/sda3 /mnt

>root# swapon /dev/sda2

>root# btrfs subvolume create /mnt/@

>root# btrfs subvolume create /mnt/@home

>root# btrfs subvolume create /mnt/@var

>root# btrfs subvolume create /mnt/@snapshots

>root# umount /mnt

>root# mount -o noatime,compress=lzo,space_cache,subvol=@ /dev/sda3 /mnt

>root# mkdir -p /mnt/{boot/efi,home,var,.snapshots}

>root# mount -o noatime,compress=lzo,space_cache,subvol=@home /dev/sda3 $ /mnt/home

>root# mount -o noatime,compress=lzo,space_cache,subvol=@var /dev/sda3 /mnt/var

>root# mount -o noatime,compress=lzo,space_cache,subvol=@snapshots /dev/sda3 /mnt/.snapshots

>root# mount  /dev/sda1 /mnt/boot/efi


- montar as partiçoes ext4 BIOS mbr

>root# swapon /dev/sda2

>root# mount  /dev/sda3 /mnt



7. *instalar o sistema (copiar os arquivos para o disco):*

>root# pacstrap /mnt base base-devel linux linux-headers linux-firmware nano vim neovim  dhcpcd networkmanager net-tools bash-completion dosfstools ntfs-3g networkmanager-applet os-prober snapper


8. *gerar o fstab, para as partições montar automaticamente no boot*


>root# genfstab -U  -p  /mnt  >> /mnt/etc/fstab


9. *criar ambiente isolado para trabalhar dentro do novo sistema (mudar para raiz do novo sistema)*

>root# arch-chroot  /mnt


- dentro do sistema novo:

10. *atribuir o nome da máquina (hostname)*

>root#/  echo "archlinux" >> etc/hostname

- confirmar o initramfs

>root#/ mkinitcpio -P


11.  *Instalando o GRUB (Gerenciador de boot) - configurando o grub*


*se BiosBoot*

formatar em ext4

>root#/ pacman -S grub

>root#/ grub-install --target-i386-pc --recheck /dev/sda

>root#/ grub-mkconfig -o /boot/grub/grub.cfg


*se EFI*

>root#/ pacman -S grub  efibootmgr  grub-btrfs(para partições btrfs)

>root#/ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub_uefi --recheck

>root#/ grub-mkconfig -o /boot/grub/grub.cfg



12. *configurar  a localidade*

>root#/ vim /etc/locale.gen  (descomentar pt-br)

:wq

- configurar o idioma:

>root#/ locale-gen

>root#/ echo "LANG=pt_BR.UTF-8">>/etc/locale.conf


13. *configurar o teclado:*

>root#/ echo "KEYMAP=br-abnt2">>/etc/vconsole.conf


14. *Sincronize o fuso-horário e o clock de hardware:*

>root#/ ln -sf /usr/share/zoneinfo/America/Recife /etc/localtime

>root#/ hwclock --systohc --utc


15. *pausa para beber agua:*

>root#/ beber agua


16. *criando a senha de root*

>root#/ passwd


17. *instalar o sudo:*

>root#/ pacman -S sudo


18. *configurar editor para visudo*

>root#/ EDITOR=vim visudo ( >root#descomentar %wheel ALL=(ALL) ALL)

:wq


19. *Crie um usuário não-root e defina a senha:*

>root#/  useradd -m -g users -G wheel,storage,power -s /bin/bash seuUserName

>root#/ passwd seuUsername



20. *Digite a seguinte sequência de comandos para efetuar um reboot, agora com o seu sistema já instalado.*

>root#/ exit (sair do chroot)

>root# umount /mnt/boot/efi

>root# umount /mnt/.snapshots

>root# umount /mnt/var

>root# umount /mnt/home

>root# umount /mnt

>root# reboot


- configurar tela no virtual box

>root# xrandr -s 1920x1080

- verifica espaço ocupado
>root# du -hc  


21. *Atualizar sistema*

- configurar internet

>root# systemctl enable NetworkManager

>root# systemctl start NetworkManager

>root# systemctl enable dhcpcd

>root# dhcpcd

>root# nmtui

>user$ ping [endereço da rede ou URL]


>user$ sudo pacman -Syyuu


22. *instalar a interface gráfica:*

>user$ sudo pacman -S xorg xorg-server xorg-xinit xorg-apps mesa

*KDE plasma*

>user$ sudo pacman -Syu

>user$ sudo pacman -S plasma

>user$ sudo pacman -S sddm

- ativar sddm

>user$ sudo systemctl enable sddm


*XFCE4*

>user$ sudo pacman -S lightdm xfce4 xfce4-goodies

>user$ vim .xinitrc

exec startxfce4

:wq


*Gnome*

>user$ sudo pacman -S xorg gnome gdm gnome-extra gnome-tweaks  firefox

>user$ pwd

>user$ vim .xinitrc

export XDG_SESSION_TYPE=x11

export GDK_BACKEND=x11

exec gnome-session

:wq

- ativar o gdm

>user$ sudo systemctl enable gdm

>user$ sudo systemctl status gdm


23. *instalar o wayland*

>user$ sudo pacman -S xorg-server-xwayland inicializar o GDM


24. *reboot*


25. Vamos instalar agora os drivers de vídeo:


#### Intel


>root# pacman -S xf86-video-intel libgl mesa


#### Nvidia


>root# pacman -S nvidia nvidia-libgl mesa


#### AMD


>root# pacman -S mesa xf86-video-amdgpu



Caso você esteja usando o Arch em uma máquina virtual, como o Virtualbox, instale estes pacotes:


>root# pacman -S virtualbox-guest-utils virtualbox-guest-modules-arch mesa mesa-libgl


26. *Instalando o tema FLAT-REMIX*

>user$ sudo pacman -S git gnome-tweak-tool  ttf-hack gnome-shell-extensions

>user$ git clone https://github.com/daniruiz/flat-remix

>user$ git clone https://github.com/daniruiz/flat-remix-gtk

>user$ mkdir -p ~/.icons && mkdir -p ~/.themes

>user$ cp -r flat-remix ~/.icons/ && cp -r flat-remix-gtk ~/.themes/


27. *instalar o firewall*

>user$ sudo pacman -S ufw

>user$ sudo ufw enable


28.  *Como instala o AUR*


#### PAMAC

>user$ sudo pacman -S --needed base-devel

>user$ git clone https://aur.archlinux.org/pamac-aur.git

cd pamac-aur

makepkg -si


Usando pamac


O Pamac pode ser usado por meio do terminal ou da GUI. O uso da GUI do pamac é muito intuitivo.


Com o terminal, para procurar um pacote use o seguinte comando com <package> substituído pelo nome do pacote que você está procurando


pamac search <package>

Para instalar um pacote ,


pamac install <package>

Para desinstalar um pacote,


pamac remove <package>


mais informações:

ArchWiki (archlinux.org)

Arch Linux - Best distro ever? | AkitaOnRails.com

[Akitando] >root#114 - O Melhor Setup Dev com Arch e WSL2 | AkitaOnRails.com
