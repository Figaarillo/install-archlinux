
<h1 align='center'> Guía de instalación de Archlinux</h1>

## Primeros pasos

### Configuración de teclado

```shell
# loadkey us
```

### Configuración fecha y hora

```shell
# timedatectl set-ntp true
```

### Conectar a internet

Ver redes wifi

```shell
ip link
```

-> `wlan0` por ejemplo

levantarlo si esta `DOWN`

```shell
# ip link set wlan0 up
```

```shell
# iwctl
# station wlan0 connect   WIFI_NAME
```

### Descargas paralelas

```shell
# vim /etc/pacman.conf
```

```text
ParallelDownloads = 5
```

### Mirrorlist

```shell
# pacman -Sy reflector
```

```shell
# cp -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

```shell
# reflector --verbose -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```

```shell
# pacman -Syy
```

## Cración partiones

```shell
# cfdisk
```

Seleccionar _gtp_

Esquema:

| Size    | Type             |
| ------- | ---------------- |
| 512M    | EFI System       |
| default | Linux filesystem |

## Formateo de particinoes

Para ver el particionado realizado

```shell
# fdisk -l
```

### Formateo partición EFI

```shell
# mkfs.fat -F32 /dev/sda1
```

### Formateo partición root

Con btrfs

```shell
# mkfs.btrfs /dev/sda2
```

## Montado de particiones

### Partición root

```shell
# mount /dev/sda2 /mnt
```

#### Creación de subvolumenes

```shell
# btrfs su cr /mnt/@
# btrfs su cr /mnt/@home
# btrfs su cr /mnt/@var
# btrfs su cr /mnt/@opt
# btrfs su cr /mnt/@tmp
# btrfs su cr /mnt/@.snapshots
# umount /mnt
```

Ver volumenes creados

```shell
# ls /mnt/
```

o

```shell
# btrfs subvolume list /mnt
```

#### Montando los subvolumenes

```shell
# mount -o noatime,commit=120,compress=zstd,space_cache,subvol=@ /dev/sda2 /mnt

# mkdir -p /mnt/{boot/efi,home,var,opt,tmp,.snapshots}

# mount -o noatime,commit=120,compress=zstd,space_cache,subvol=@home /dev/sda2 /mnt/home

# mount -o noatime,commit=120,compress=zstd,space_cache,subvol=@opt /dev/sda2 /mnt/opt

# mount -o noatime,commit=120,compress=zstd,space_cache,subvol=@tmp /dev/sda2 /mnt/tmp

# mount -o noatime,commit=120,compress=zstd,space_cache,subvol=@.snapshots /dev/sda2 /mnt/.snapshots

# mount -o subvol=@var /dev/sda2 /mnt/var

# mount /dev/sda1 /mnt/boot/efi
```

## Instalado de sistema base

```shell
# pacstrap /mnt base linux linux-firmaware vim btrfs-progs reflector sudo
```

## fstab

```shell
# genfstab -pU /mnt/ >> /mnt/etc/fstab
```

## Configuración del sistema

```shell
# arch-chroot /mnt
```

### Zona horaria

```shell
ln -s /usr/share/zoneinfo/America/Cordoba /etc/localtime
```

```shell
hwclock --systohc
```

### Idioma del sistema

```shell
vim /etc/locale.gen
```

Descomentar

```txt
en_GB.UTF-8 UTF-8
```

Ahora generar la configuracion

```shell
locale-gen
```

Setea el lenguaje

```shell
echo LANG=en_GB.UTF-8 >> /etc/locale.conf
```

### Configuración del teclado

```shell
echo "KEYMAP=us" >> /etc/vconsole.conf
```

## Configuracion de la red

Set hostname

```shell
echo Archlinux >> /etc/hostname
```

Set hostfiles

```shell
vim /etc/hosts
```

```txt
127.0.0.1   localhost
::1         localhost
127.0.1.1   Archlinux.localdomain Archlinux
```

## Usuarios y contraseñas

Password usuario root

```shell
passwd
```

Crear usuario y grupo

```shell
groupadd figarillo
useradd -mG wheel -s /bin/zsh figarillo
useradd -m -g users -G figarillo -s /bin/zsh figarillo
useradd -m -g users -G figarillo -s /bin/shell figarillo
passwd figarillo
```

Agregar a sudo al usuario

```shell
visudo
```

si no funciona

```shell
nano /etc/sudoers
```

Agregar al final del archivo

```shell
figarillo ALL=(ALL) NOPASSWD:ALL
```

## Configuraciones extras

```shell
vim /etc/pacman.conf
```

Buscar la linea "#VerbosePkgLists" y y escribir debajo " ParallelDownloads = 5
".

Escrolear y descomentar:

- " #[multilib] "
- " #Include = /etc/pacman.d/mirrorlist "

Guardar y salir

```shell
# cp -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

```shell
# reflector --verbose -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```

```shell
# pacman -Sy
```

## Instalación de paquetes finales

```shell
# pacman -Syy base-devel linux-headers linux-zen linux-zen-headers grub grub-btrfs efibootmgr networkmanager network-manager-applet vim neovim kitty netctl dialog wpa_supplicant wireless_tools mtools dosfstools bluez bluez-utils xdg-utils xdg-user-dirs ntfs-3g gvfs gvfs-mtp file-roller alsa-utils alsa-lib pulseaudio pulseaudio-alsa pavucontrol xf86-input-synaptics openssh git
```

## Agregar módulo btrfs a mkinitcpi

```shell
vim /etc/mkinitcpio.conf
```

Ir a `MODULES=()` y modificarlo

-> `MODULES=(btrfs)`

Ahora recrear la imagen

```shell
mkinitcpio -p linux-zen
```

_linux-zen puede ser reemplazado por `linux` o el kernel intalado_

## Instalación y configuraciones del GRUB

```shell
vim /etc/default/grub
```

```txt
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
GRUB_DISABLE_OS_PROBER=false
```

Instalar grub

```shell
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Archlinux
```

Generar el archivo de configuracion

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

## Salir

```shell
exit
```

```shell
# umount -R /mnt
```

Si se esta en maquina virtual, apagar y sacar el disco de Archlinux

```shell
# poweroff
```

Si esta en maquina normal, reiniciar

```shell
# reboot
```

## Inicio del sistema

Hacer el primer login como root

Verificar /etc/sudoers

### Habilitación de servicios

```shell
# systemctl start NetworkManager.service
# systemctl enable NetworkManager.service
# systemctl start wpa_supplicant.service
# systemctl enable wpa_supplicant.service
# systemctl enable sshd
# systemctl enable bluetooth
# systemctl enable --now fstrim.timer
```

Salir del usuario root

## Actualizar sistema

```shell
# sudo pacman -Syyu
```

## Servidor gráfico

```shell
# sudo pacman -S xorg xorg-server xorg-xinit
# sudo pacman -S mesa mesa-demos
```

### Gráficos intel

```shell
# sudo pacman -S xf86-video-intel intel-ucode
```

### Genérico

```shell
# sudo pacman -S xf86-video-vesa
```

## Instalando Yay

```shell
cd /tmp
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si ## Se compilará e instalará
cd ~ ## Volvemos a nuestro home
rm -rf /tmp/yay-git # Ya no necesitamos esto, yay es capaz de actualizarse a él mismo
```
