# Read more latter

- [Arch Linux with BTRFS Installation ](https://www.nishantnadkarni.tech/posts/arch_installation/)

# Guia de instalación de Archlinux

---

![init-page](./assets/01-init-02.png)

## Primeros pasos

---

## Configuración de teclado

Lo primero lo primero que deberías hacer es configurar el teclado ya que, en
caso de tenerlo mal configurado tendrías problemas a la hora de querer ingresar
algún símbolo. Para ello en caso de que la distribución de teclado sea en inglés
no hace falta hacer nada dado que es que está configurado por defecto, aún así
se podes ingresar el siguiente comando para estar seguro.

```shell
# loadkeys us
```

En caso de que la distribución de teclado sea en español podes usar uno de los
siguientes comandos

Si el teclado tiene distribución de España ejecuta:

```shell
# loadkeys es
```

En caso de tener distribución de Latinoamérica ejecuta:

```shell
# loadkeys la-latin1
```

Para consultar la lista completa de distribuciones de teclado se puede usar el
siguiente comando

```shell
# ls /usr/share/kbd/keymaps/**/*.map.gz | more
```

O también se puede usar

```shell
# localectl list-keymaps | more
```

## Verificar modalidad de arranque

Para verificar el modo de boot, se puede usar el siguiente comando.

```shell
# ls /sys/firmware/efi/efivars
```

Si el comando muestra muestra algo como lo siguiente, el sistema bootea en modo
UEFI.

!(efivar)[./assets/efi-efivars.png]

Si el directorio no existe, entonces el sistema puede bootear en BIOS.

## Conexión a internet

Validar la conexión a internet

```shell
# ping -c 5 archlinux.org
```

!(ping)[./assets/ping.png]

Si obtuviste una salida como esta, quiere decir que tenes conexión a Internet,
por lo tanto, podes continuar.

## Mirror list

---

En archlinux la lista de servidores de réplicas se encuentra en un fichero
llamado mirrorlist.

Tenerlo bien configurado ayuda a que al momento de realizar descargas,
actualizacoines o similares, lo hagamos con la mejor velocidad.

Desdes la página [archlinux-mirrorlist](https://archlinux.org/mirrorlist/) se
pude generar la lista de servidores eligiendo los parametros que se quieran.

Una vez que se tenga el fichero lo siguiente es tener copiarlo dentro del
fichero de mirrorlist que usa Archlinux.

```shell
sudo pacman -S reflector
```

Reflector es un script que es capaz de obtener la lista más reciente de mirrors
desde la página MirrorStatus, filtrar los mirrors más actualizados, ordenarlos
en base a su velocidad y sobrescribir el archivo /etc/pacman.d/mirrorlist

Pero antes de hacer cualquier cosa, lo ideal es realizar una copia de seguridad
de la lista que se tenga actualmente.

```shell
# cp -vf /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

Una vez hecho esto se puede ejecutar lo siguiente

```shell
# sudo reflector --verbose -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```

```shell
# sudo vim /etc/pacman.d/mirrorlist
```

Una vez hecha la copia de seguridad ya puedes ejecutar reflector. Y por ultimo
sincronizamos, de haber actualizaciones las aceptamos.

```shell
# sudo pacman -Syy
```

## Esquema de particiones

---

Para iniciar con el particionado del disco, Archlinux provee algunas
herramientas para hacerlo, esta herramienta es `cfdisk`

```shell
# cfdisk
```

Donde una vez que lo ejecutes, se va a mostrar lo siguiente

![cfdisk](./assets/02-cfdisk.png)

Conociendo el modo de booteo, lo siguiente a hacer es determinar el tipo de
etiqueta. En caso de que tengas _UEFI_ tenes que seleccionar _¨gpt¨_. Pero en
caso de que tengas _BIOS_ tenes que elegir _¨dos¨_.

Una vez que selecciones la etiqueta te saldrá la siguiente pantalla

![partitions](./assets/02-partitions.png)

El esquema de particiones que elijas variar según las necesidades que tengas, se
puede contar con una partición _/home_, _/boot_ - _/_ (raíz) o _Swap_.

La ventaja de crear particiones para ciertos directorios, es la de poder volver
a montarlos, como en el caso del directorio /home en otra instalación sin tener
que realizar modificaciones sobre este, conservando los directorios de los
usuarios. También podríamos eliminarlos y volverlos a crearlos de forma
individual, como en el caso del directorio /boot; si nuestro inicio de sistema
se corrompe, es posible trabajar directamente sobre el de forma individual, sin
tocar el resto de los directorios del sistema, eliminando la partición donde se
encuentra montando el directorio /boot y creando una partición nueva donde
montaremos y reinstalaremos el nuevo /boot.

Pero como mínimo, para sistemas UEFI, deberías tener las siguientes particiones

- efi boot
- root /

En sistemas BIOS, por otro lado, se cuenta con una partición de booteo y otra
para la partición root.

- booteable /boot
- root /

Con respecto a la partición _swap_, existen diferentes caminos con los que se
puede optar. En mi caso yo opto por usar un fichero Swap, pero mostraré como
crear una partición swap así como una partición _home_.

Un esquema de particiones completo sería:

![partitions-scheme](./assets/partitions-escheme.png)

Para conseguir esto, sobre el espacio que dice _Free space_ seleccionas la
opción _new_ e indicas el espacio que ocupa.

Luego, hay que seleccionar el tipo. Esto se puede hacer desde la opción _type_
quedándote algo similar a esto.

- 512M efi # tamaño recomendado
- 4G linux swap # tamaño recomendado equivalente a la RAM
- 25G root /
- 20,5G home

Pero de este esquema podes obviar tanto de la partición _Home_ como de la _Swap_
y en sistemas BIOS, puedes obviar _efi_

Cuando termines de hacer tu esquema seleccionas la opción _Write_ y luego
escribes _yes_

## Formateo de las particiones

---

Para formatear una partición, se debe tener en claro el orden que tiene cada
una, es decir, saber si, por ejemplo, la partición root esta en **/dev/sda1/** o
en **/dev/sda2/**, que esto varia según como se hayas realizado el paso
anterior.

Para conocer el orden en de cada partición, se puede hacer uso del siguiente
comando.

```shell
# fdisk -l
```

!(fdisk-l)[./assets/fdisk-l.png]

Una vez confirmado el orden de cada partición, se puede continuar con el
formateo

Para dar formato a una partición se hace uso del comando `mkfs`, donde la
primera partición a formatear debe ser la partición de booteo

### Formato partición /boot

#### UEFI

```shell
# mkfs.vfat -F32 /dev/sda1
```

o

```shell
# mkfs.fat -F 32 /dev/sda1
```

!(mkfs.vfat32)[./assets/mkfs-vfat.png]

#### Sin UEFI

```shell
# mkfs.ext2 /dev/sda1
```

### Formato partición root

```shell
# mkfs.ext4 /dev/sda2
```

!(mkfs-ext4)[./assets/mkfs-ext4.png]

## Formato y activación de la partición SWAP

En caso de contar con una partición _swap_, tenes que seguir los siguientes
pasos para poder formatearla y activarla

### Formateo de la swap

```shell
# mkswap /dev/sdaN
```

Reemplaza la _'N'_ por el número de partición correspondiente.

### Activación

```shell
# swapon /dev/sdaN
```

## Montado de particiones

---

### Montado de partición root

Lo primero a montar es la partición root para ello se ejecuta el siguiente
comando

```shell
# mount /dev/sda2 /mnt
```

### Montado de partición boot

#### Con UEFI

Seguido a esto, es necesario que crees el siguiente directorio en caso de tener
un sistema UEFI

```shell
# mkdir -p /mnt/boot/efi
```

#### Sin UEFI

```shell
# mkdir /dev/sda1/ /mnt/boot
```

Una vez creado el directorio, hay que hacer el montado

#### Con UEFI

```shell
# mount /dev/sda1 /mnt/boot/efi
```

#### Sin UEFI

```shell
# mount /dev/sda1 /mnt/boot
```

### Montado partición Home

En caso de contar con una partición _home_ lo que se debe hacer es lo siguiente

Primero hay que crear su respectivo directorio

```shell
# mkdir /mnt/home
```

Una vez creado el directorio, se continua con el montado

```shell
# mount /dev/sdaN /mnt/home
```

## Instalación del sistema base

---

Para continuar con este paso, es necesario verificar que se tiene conexión a
internet

```shell
# ping archlinux.org
```

## Paquetes a instalar

### [base](https://archlinux.org/packages/core/any/base/)

Consiste en un grupo de paquetes que contienen todas las dependencias necesarias
para instalar Archlinux.

### `gvfs`

Paquete que permite montar USBs, Micro SD y demás particiones de disco.

### `gvfs-mtp`

Paquete que permite montar un android

### `netctl` `wpa_supplicant` `dialog`

Paquetes necesarios para el uso del wi-fi

### `xf86-input-synaptics`

Controlador para el touchpad

### `ntfs-3g`

También agreguemos el paquete ntfs-3g para que detecta la partición de Windows y
así poder interactuar con sus archivos.

### `nvim`

Editor de texto a usar. También, si queres algo más sencillo, podés optar por
usar `nano`

Para realizar la instalación se hará uso del script `pacstrap` y los mismos se
harán sobre el directorio **/mnt/**.

```shell
# pacstrap /mnt base base-devel efibootmgr networkmanager gvfs gvfs-mtp xdg-user-dirs netctl wpa_supplicant dialog xf86-input-synaptics nvim
```

## Generar FSTAB

---

El fichero /etc/fstab contiene información del sistema de archivos. Esto define
como se almacena el sistema de particiones que ha sido montado.

```shell
# genfstab -pU /mnt/ >> /mnt/etc/fstab
```

## Configuración del sistema base

---

Para continuar con la instalación es necesario que ingreses al sistema base
instalado. Para ello, se usa el siguiente comando.

```shell
# arch-chroot /mnt
```

Fíjate bien y el prompt cambió, ahora está entre corchetes [root@archiso /]

## Swap-file

La swap o espacio de memoria de intercambio (también conocido como memoria
virtual), es la que se sirve del espacio en el HDD en lugar de un módulo de
memoria. La swap entra en uso cuando la memoria real se está agotando, el
sistema copia parte del contenido de la RAM en la swap, a fin de poder realizar
otras tareas.

Una de las principales desventajas de utilizar este sistema es que el sistema se
volverá más lento, ya que la velocidad de transferencia de datos entre la RAM y
el HDD es abismalmente diferente y todo depende de tu hardware.

[que-es-la-swap] [que-es-la-swap-file]

Vamos a crear el archivo de swap utilizando el comando `dd`, el archivo que voy
a crear será de 2GB, es decir 2 bloques de un 1GB cada uno.

```shell
# dd if=/dev/zero of=/swapfile bs=1G count=2 status=progress
```

Ahora hay que cambiar los permisos del archivo

```shell
# chmod 600 /swapfile
```

Cambia el tipo de archivo a _swap_

```shell
# mkswap /swapfile
```

Habilita el archivo con

```shell
# swapon /swapfile
```

Agrega el archivo al sistema de archivos, es decir, al archivo _fstab_ (Yo voy a
utilizar `nvim`, si instalaste `nano`, utiliza nano)

```shell
# nvim /etc/fstab
```

Al final del archivo tenes que agregarlo siguiente

```
/swapfile none swap default 0 0
```

## Configuración de zona horaria

Es importante establecer la zona horaria. Acá hay una lista de algunas zonas
horarias que se pueden usar.

### Argentina

```shell
# ln -sf /usr/share/zoneinfo/America/Buenos_Aires /etc/localtime
```

### España

```shell
# ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```

Lo que se esta haciendo en este comando es establecer un enlace simbólico en
_/etc/localtime_

Para ver la lista completa de países se puede usar lo siguiente

```shell
# ls /usr/share/zoneinfo/
```

!(zoneinfo)[./assets/zoneinfo.png]

Ahora ya podemos sincronizar el reloj del sistema con el reloj de hardware

```shell
hwclock --systohc
```

## Configurar idioma del sistema

Para configurar el idioma principal de nuestro sistema operativo vamos a editar
el archivo local.gen

```shell
# nvim /etc/locale.gen
```

Una vez dentro, tenes que buscar el nombre de tu país o por lo menos de la del
idioma que quieras tener tu sistema, descomentarlo y guardar el archivo.

Por ejemplo

```shell
LANG=es_AR.UTF-8
```

Y para confirmar este cambio, es necesario hacer lo siguiente

```shell
# echo LANG=es_AR.UTF-8 >> /etc/locale.conf
```

Ahora generamos el archivo `locale.gen` con el siguiente comando

```shell
# locale-gen
```

## Configuración del reloj de hardware

Cuando GNU/Linux arranca, el sistema está configurado para leer el reloj interno
del equipo, después el reloj del sistema, que es independiente. Usaremos el
comando hwclock -w para ajustar el reloj interno.

```
# hwclock -w
```

## Configuración de la distribución teclado

Ahora, para configurar la distribución de nuestro teclado y dejarlo con el
idioma que escogimos al principio de toda la instalación, cuando usamos el
comando loadkeys (pero siempre tendremos que escoger nuestra distribución de
teclado en las configuraciones de Teclado del entorno gráfico que escojas):

```shell
# echo KEYMAP=la-latin1 > /etc/vconsole.conf
```

## Configuración de red - hostname

El hostname representa el nombre del equipo con el cual nuestro equipo será
visible en la red.

Para crear el mismo es necesario que agregues un fichero dentro de la ruta /etc

Para ello, si tenes vim, nvim o nano podes alguno de estos seguido del nombre
del archivo

```shell
# echo Archlinux >> /etc/hostname
```

```shell
# nvim /etc/hosts
```

Una vez dentro, tendrias que agregar lo siguiente

```
127.0.0.1 localhost
::1       localhost
127.0.1.1 HOSTNAME.localdomain HOSTNAME
```

Tenes que reemplazar 'HOSTNAME' con el nombre que vos hayas elegido

## Configuración de DNS

Configuramos DNS para nuestra red (en caso de que ya nuestro DHCP no lo haya
hecho).

```shell
nvim /etc/resolv.conf
```

Una vez dentro del archivo, tenes que agregar lo siguiente

```
nameserver 1.1.1.1
nameserver 8.8.8.8
```

Y para estar seguro, podes probar la conexión haciendo un `ping`

```shell
# ping archlinux.org
```

## Crear contraseña del Administrador (root)

El siguiente paso de la instalación es darle un password a nuestro usuario de
root.

Como ya estamos en nuestra instalación, para cambiar el password solo debemos
teclear:

```shell
# passwd
```

Escribes la contraseña que desees. No te mostrará nada al escribir, así que
fíjate muy bien cuál pondrás de root. Te pedirá nuevamente que confirme la
contraseña que acabas de ingresar y una vez confirma, pasemos a lo siguiente.

### Añadir usuario y grupo

- **-m**: indica que se cree el directorio, home y todo lo que este contiene
  correspondientes al usuario dentro del directorio root

- **-g**: para indicarle el nombre de un grupo al cual se agregará al usuario

- **-s**: para indicarle que sea el usuario por defecto de la shell que se
  indique. En caso de ser el único usuario esto se puede omitir.

```
useradd -mG GROUPNAME -s /bin/zsh USERNAME
useradd -mG GROUPNAME -s /bin/bash USERNAME
```

El nombre del usuario debe ser en minúsculas.

## Instala los paquetes finales

Estos son algunos de los paquetes que recomiendo instalar, pero podes agregar el
que vos quieras.

[base-devel](https://archlinux.org/groups/x86_64/base-devel/)

Consiste en un grupo de paquetes que contienen todas las dependencias necesarias
para instalar Archlinux.

### `grub`

Gestor de arranque que es necesario para cuando se haya terminado la instalación

### `efibootmgr`

Paquete necesario para la instalación con UEFI

### `networkmanager`

Paquete que proporciona al sistema la detección y configuración automática para
conexión a la red. De no tenerlo, no se tendrá conexión al usar el OS

### `network-manager-applet`

### `dialog`

### `os-prober`

El paquete os-prober para que detecte otros sistemas operativos en nuestro disco
duro. Si lo estás instalando junto a windows te sera útil.

### `mtools`

### `dosfstools`

### `reflector`

### `openshh`

### `git`

### `xdg-user-dirs`

Paquete que permitirá crear las carpetas por defecto de del usuario de forma
automática.

```shell
# pacman -S base-devel linux-headers grub efibootmgr networkmanager network-manager-applet dialog os-prober mtools dosfstools  reflector openssh git xdg-utils xdg-user-dirs virtualbox-guest-utils pulse-audio bluez bluez-utils
```

## BootLoader - GRUB

---

Ahora procedamos a instalar el grub y usar el comando correcto, dependiendo si
usas UEFI o No.

Sin UEFI

```shell
# grub-install /dev/sda
```

Con UEFI

```shell
# grub-install --efi-directory=/boot/efi --bootloader-id='Arch Linux' --target=x86_64-efi
```

## Actualizar grub

Creamos el archivo `grub.cfg`

```shell
# grub-mkconfig -o /boot/grub/grub.cfg
```

## Dual boot

Si estás haciendo un dualboot con windows y cuando estabas configurando el grub,
no te apareción windows u otro sistema operativo en la lista, usa el comando
`os-prober` Y te tendría que aparecer Windows.

Y ahora vuelve a ejecutar el comando

```shell
# grub-mkconfig -o /boot/grub/grub.cfg
```

## Saliendo de chroot

---

Ya terminado esto, salimos de chroot.

```shell
# exit
```

## Desmontado de particiones y reinicio del sistema

---

Es importante desmontar las particiones en el siguiente orden:

1. /boot o /boot/efi
2. /home
3. /

O todo junto con el siguiente comando

```shell
# umount -R /mnt
```

### Desmontado partición **boot**

#### Con UEFI

```shell
# umount /mnt/boot/efi
```

#### Sin UEFI

```shell
# umount /mnt/boot/
```

### Desmontado partición home

```shell
umount /mnt/home
```

### Desmontado partición root

```shell
umount /mnt
```

## Reinicio del sistema

Una vez concluida esta etapa de la guía reiniciamos el sistema con el comando
reboot, para ya entrar en nuestro archlinux recién salido del horno.

## Inicio del sistemas y configuraciones

---

Para este punto se puede decir que ya se ha instalado Archlinux pero para
concluir con la instalación es necesario realizar algunas configuraciones.

!(imagen)[]

Para continuar con los siguientes es necesario ingresar con usuario **root**

Entonces, cuando pida ingresar el nombre de usuario le pones 'root' y en la
parte de contraseña, la que creamos para este usuario.

## Añadir permisos root

Para asignar permisos root a un usuario, dentro de Archlinux, una los caminos a
seguir es el de agregarlo dentro del grupo wheel. Haciendo esto se indica que
este usuario tendrá permisos root. Otro alternativa, es la de hacer que el grupo
donde se encuentra el usuario tenga permisos root. Tanto para uno como para el
otro método se debe editar el fichero _sudoers_

Si se tiene el editor vim se puede usar el siguiente comando

```shell
visudo
```

Por el contrario, si se tiene el editor nano se lo hace de la siguiente manera.

```shell
nano /etc/sudoers
```

A continuación, si se opto por agregarlo dentro del usuario wheel con
descomentar la siguiente linea ya estaría.

```text
%wheel ALL=(ALL) NOPASSWD:ALL
```

Caso contrario, se debe agregar lo siguiente y por conveniencia al final del
fichero

```text
GROUPNAME ALL=(ALL) NOPASSWD:ALL
```

## Habilitar Network Manager

Paso súper importante para tener acceso a Internet. Lo que vamos a hacer es que,
a través del paquete `networkmanager` se creará un demonio que por defecto se
encuentra apagado.

```shell
# systemctl start NetworkManager.service
```

Y después...

```shell
systemctl enable NetworkManager.service
```

Una vez que hiciste esto, ya no es necesario seguir en con el usuario root.

Ahora ejecuta el comando `exit`, y luego pone tu usuario y contraseña

## Habilitar SSH

Ahora continuamos con la habilitación del SSH

```shell
# systemctl enable sshd
```

## Conectarse a Internet

### WiFi

Si nos queremos conectar a una red wifi, ya no lo haremos por medio de
wifi-menu, sino que con el siguiente comando

```shell
# sudo nmcli dev wifi connect SSID password PASSWORD
```

Donde SSID es el nombre de tu red y PASSWORD es la que tiene tu modem o la que
le has puesto, si es de seguridad WPA2. Un ejemplo sería

### Cableada

Si usas un cable ethernet, pues no harás nada más que habilitar Network Manager,
cosa que ya hicimos anteriormente cuando ejecutamos

```shell
# systemctl enable NetworkManager.service
```

## Actualización de sistema

---

Una vez tengamos Intenet corriendo en nuestro sistema, es imprescindible antes
de continuar, realizar una actualización de nuestro sistema. De esta forma
tendríamos todos nuestros paquetes a la ultima versión y nuestra base de datos
de paquetes sincronizada con la de los repositorios. Basta solo con ejecutar la
siguiente linea de comando:

```shell
# sudo pacman -Syyu
```

Una vez finalizado este paso, ya podes decir que tenés Archlinux instalado

## Instalar complementos gráficos

---

Hasta esta instancia se podría decir que ya tenemos Archlinux completamente
instalado, pero no muy intuitivo, para que sea un sistema completamente
intuitivo para el usuario necesitamos instalar un entorno de escritorio. Veamos
paso a paso como llevar a cabo este procedimiento:

## Servidor gráfico

El servidor gráfico es una de las capas mas bajas de la interfaz gráfica, es el
responsable de las operaciones gráficas que dibujan iconos, fondos, ventanas,
etc que ejecutan las aplicaciones Para instalarlo por completo ejecutamos…

```shell
# sudo pacman -S xorg-server xorg-xinit
```

## Instalar mesa

Básicamente, se puede definir como un conjunto de software para el procesamiento
de gráficos avanzados, teniendo como objetivo funcionar tanto sobre GPU
dedicadas como en los aceleradores gráficos integrados en las CPU (como las IGP
de Intel y las APU de AMD). Incluye drivers e implementaciones de distintas API,
entre las cuales están OpenGL, OpenGL ES, OpenCL, OpenMAX, VDPAU, VA API, XvMC y
Vulkan. Sin embargo, Mesa no compone todo el stack gráfico de GNU/Linux, ya que
los drivers para las GPU están incluidos en el kernel. Para instalar mesa por
completo ejecutamos:

```shell
# sudo pacman -S mesa mesa-demos
```

## Controladores de vídeo

Los controladores de video van a ser instalados según el tipo de tarjeta gráfica
que estemos utilizando, ya sea onboard o externa. En primera instancia debemos
identificar nuestra tarjeta de video. Para saber el tipo de controlador que
necesitas, escribe la siguiente linea de comando:

```shell
# lspci | grep VGA
```

Donde identificaremos el tipo de tarjeta de video que estamos ocupando.

### Según el fabricante

#### AMD

```shell
# sudo pacman -S xf86-video-amdgpu amd-ucode
```

#### Nvidia open source

```shell
# sudo pacman -S xf86-video-nouveau
```

O

```shell
xf86-video-nv
```

Desde los repositorios de AUR

#### Nvidia privativo

```shell
# sudo pacman -S nvidia nvidia-utils
# sudo pacman -S nvidia-340xx
```

#### ATI Radeom open source

```shell
# sudo pacman -S xf86-video-ati
```

#### ATI Radeom propietario

```shell
# catalyst-dkms
```

Disponible solo en los Repositorio AUR

#### Intel código abierto

```shell
# sudo pacman -S xf86-video-intel intel-ucode
```

### Desconocido

En caso de no encontrar ningún controlador, el controlador vesa es el más
genérico, aunque no ofrece soporte 3D ni aceleración por hardware.

```shell
# sudo pacman  -S xf86-video-vesa
```
