# Banana-pi-m2-zero-Arch-Linux <img src="https://media.giphy.com/media/hvRJCLFzcasrR4ia7z/giphy.gif" width="25px"> 
#### Proyecto de Instalación y Configuración de Arch Linux en Banana Pi M2 Zero

![1200px-Archlinux-vert-dark svg (1)](https://user-images.githubusercontent.com/62630527/158080103-bbbe87f2-eb5c-402e-9fda-315ab8e79886.png) ![BANANAPI-ZERO-01 (1)](https://user-images.githubusercontent.com/62630527/128290176-105ffebd-346a-4c38-8d4a-be466738a2ef.png)

## Que hay de nuevo

Esta vez les traigo un nuevo repositorio con el fin de mostrarles como instalar Arch Linux en la Banana Pi M2 Zero así que sin decir más aquí les muestro como realizar la instalación.

Nota importante: Este tutorial solo servirá con SD de 16G en adelante dado que con SD de menor capacidad se presentan problemas de estabilidad o simplemente no cargara el sistema más adelante subiré la corrección a este problema.

## Tabla de contenidos

- [Instalación](#instalación-)
- [Primer Arranque](#primer-arranque-)
- [Configuración Ethernet](#configuración-de-ethernet-)
- [Corregir Sistema](#corregir-sistema-)
- [Habilitar Wifi](#habilitar-wifi-)
- [Links y Video](#links-)
- [Descargas e Imagen](#descargas-)
- [Referencias](#referencias)

## Instalación <img src="https://user-images.githubusercontent.com/62630527/158048706-9cb18a7c-c450-4d83-bf7d-d96cbc0ffd7d.png" width="25px"> 

#### Para comenzar tendremos que contar con un sistema Linux, al cual recomiendo actualizar antes de proseguir con esto, además que tendremos que instalar las siguientes dependencias para evitar errores o complicaciones más adelante.

	sudo apt-get update -y
	sudo apt-get upgrade -y
	sudo apt-get install -y u-boot-tools 
	sudo apt-get install -y libarchive-tools

#### Una vez se hayamos instalado las dependencias que necesitaremos procederemos a revisar la letra asignada a nuestra SD ejemplo:(sda, sdb, sdc, etc.).

	sudo fdisk -l

#### Una vez conozcamos la letra asignada a nuestra SD procederemos a la preparación de la misma en este punto tendremos que remplazar el punto de montaje (sdX) por nuestra letra ejemplo:(sda, sdb, sdc, etc.).

	sudo dd if=/dev/zero of=/dev/sdX bs=1M count=8

#### Ahora una vez terminado lo anterior procederemos a crear una nueva tabla de particiones con la siguiente secuencia de instrucciones, en esta será donde almacenaremos el sistema base de Arch Linux, así como donde realizaremos los cambios para que nuestro sistema funcione adecuada mente y recordando cambiar sdX por nuestra letra de la SD.

	sudo fdisk /dev/sdX

	-o limpiar particiones
	-p ver particiones de la SD
	-n crear nueva partición
	-p tipo de partición primaria
	-seleccionar primer partición default y ENTER
	-Seleccionar primer sector default y ENTER
	-seleccionar último sector default y ENTER
	-w escribir tabla de particiones y salir  

#### Terminada la creación de la nueva tabla de particiones tendremos que revisar si todo ha salido bien, de ser así ahora tendremos una nueva partición llamada sdX1 que será con la que trabajaremos ahora.

	sudo fdisk -l

#### Ahora le daremos el siguiente formato a nuestra nueva partición y crearemos una carpeta llamada mnt donde montaremos nuestra partición.

	sudo mkfs.ext4 -O ^metadata_csum,^64bit /dev/sdX1
	sudo mkdir mnt
	sudo mount /dev/sdX1 mnt/

#### Una vez montada nuestra partición procederemos a descargar la imagen base de Arch Linux para ARM, y seguidamente la extraeremos dentro de nuestra carpeta mnt.

	sudo wget https://archlinuxarm.org/os/ArchLinuxARM-armv7-latest.tar.gz
	sudo bsdtar -xpf ArchLinuxARM-armv7-latest.tar.gz -C mnt/

#### Seguidamente de esto procederemos con la escritura del archivo boot.scr y desmontaremos nuestra partición.

	sudo mkimage -A arm -O linux -T script -C none -a 0 -e 0 -n "BananPI boot script" -d boot.cmd mnt/boot/boot.scr
	sudo umount mnt

#### Ahora con la partición desmontada procederemos a cargar nuestro kernel en la SD.

	sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8

#### Hecho esto volveremos a montar nuestra partición en la carpeta mnt y procederemos a remplazar el siguiente archivo con el que les dejare en la carpeta dado que con el que viene por defecto no podremos conectarnos más que por TTL y con el nuevo podremos conectarnos el primer arranque por Ethernet y tener salida HDMI.

	sudo mount /dev/sdX1 mnt/
	sudo rm -R mnt/boot/dtbs/sun8i-h2-plus-bananapi-m2-zero.dtb
	sudo cp sun8i-h2-plus-bananapi-m2-zero.dtb mnt/boot/dtbs/
	sudo umount mnt/

## Primer arranque <img src="https://user-images.githubusercontent.com/62630527/158304700-ab3e3fc5-17bd-48aa-8024-45dd7afa8278.png" width="25px">

llegados a este punto procederemos a insertar nuestra SD a nuestra banana pi, así como conectarla a la alimentación.

#### consideraciones a tener en cuenta antes del primer inicio

	-Tener en cuenta que tiene que estar conectado por ethernet o con antena wifi externa.
	-Nos conectaremos por ssh con la ip que se nos ha asignado (comando: ip addr)
	-Usuario por defecto: User: alarm Pass: alarm
	-Usuario root por defecto: User: root Pass: root

Al momento de realizar el primer inicio necesitaran loguearse como usuario root, esto con el fin de poder realizar la configuración e instalación de paquetes necesarios esto se hará una vez logueado con el usuario alarm con el comando su y la contraseña: root.

#### Bueno hecho lo anterior procederemos a agregar las key necesarias para la actualización del sistema, esto podría demorar un poco.

	pacman-key --init
	pacman-key --populate archlinuxarm

#### Ahora procederemos a realizar la actualización de nuestro sistema.

	pacman -Syu

#### Ahora instalaremos los paquetes que necesitaremos para la configuración final de nuestro sistema.

	pacman -S networkmanager
	pacman -S neofetch		(opcional: solo servirá para ver info de nuestro sistema).
	pacman -S cronie
	pacman -S sudo 
	pacman -S ethtool

## Configuración de Ethernet <img src="https://user-images.githubusercontent.com/62630527/158304491-9ac89816-5eb4-4d8e-bbad-5de724a923fb.png" width="25px">

#### Una vez instalados los paquetes necesitaremos limitar la velocidad de nuestro puerto ethernet dado que si se le deja por defecto presenta inestabilidad.

	ethtool -s eth0 speed 100 duplex half autoneg off

## Corregir sistema <img src="https://user-images.githubusercontent.com/62630527/158304496-f974d309-aa27-4f32-8b0d-9a2c6b779da6.png" width="25px">

Una vez limitemos la velocidad de Ethernet el sistema nos dejara de responder, pero es normal dado que al actualizar el sistema se rompe el fichero dtb de banana pi m2 zero, por lo que ahora para solucionarlo desconectaremos la banana pi de la alimentación y retiraremos la SD para volver a montarla en muestra carpeta mnt y eliminar el archivo sun8i-h2-plus-bananapi-m2-zero.dtb y remplazarlo por el que se encuentra en los archivos que les he dejado.

	sudo mount /dev/sdX1 mnt/
	sudo rm -R mnt/boot/dtbs/sun8i-h2-plus-bananapi-m2-zero.dtb
	sudo cp sun8i-h2-plus-bananapi-m2-zero.dtb mnt/boot/dtbs/

Hecho esto volveremos a insertar nuestra SD en la banana pi y la volveremos a conectar la alimentación, y ya debería estar funcionando.

## Habilitar Wifi <img src="https://user-images.githubusercontent.com/62630527/158304169-9a11898e-c656-49e9-be23-821574de8cfd.png" width="25px">

#### Para poder utilizar el Wifi primero iniciaremos el servicio que está asociado a este, para luego proceder a conectarnos a nuestra red Wifi en este punto el uso del wifi no será permanente, pero eso lo solucionaremos en un momento.

	NetworkManager .start
	nmtui

#### Ahora para hacer permanente el uso de Wifi crearemos un nuevo servicio para que al momento de reiniciar o iniciar el sistema se habilite el wifi, así como crearemos un nuevo archivo en el siguiente directorio.

	systemctl enable --now cronie.service
	nano /etc/cron.daily/crontab 

#### En el archivo crontab que hemos creado anteriormente agregaremos la siguiente línea para que al momento de que reiniciemos o iniciemos el sistema habilite el Wifi.

	@reboot NetworkManager .start

#### Finalmente, ya solo nos quedaría dar los permisos necesarios a nuestro archivo crontab, verificarlo y reiniciar el sistema para poder utilizarlo.

	crontab -u root /etc/cron.daily/crontab
	crontab -l
	reboot

Bueno espero y este pequeño tutorial les sea de utilidad y recuerden que si tienen alguna duda la pueden comentar además que aquí les dejo mi canal de YouTube, así como un video en el que explico todo esto más detallado.

## Links <img src="https://user-images.githubusercontent.com/62630527/158304242-6865a26d-d2b7-4f70-ac2e-4037d92bbbfa.png" width="25px">

#### Video de Instalacion

	https://github.com/TuryRx/Banana-pi-m2-zero-Arch-Linux

#### Canal de Youtube 

    TuryRx
    https://www.youtube.com/channel/UCsVnls-pcXUDKCafBRPJIsg

## Descargas <img src="https://user-images.githubusercontent.com/62630527/158044106-a52b6ef1-a65d-42d1-b376-79284df8721b.png" width="25px">

#### ArchLinux Rev 1.3 Banana Pi M2Z 14-04-22 compatible con SD 4,8,16,32GB
#### Nota: Utilizar GParted para redimencionar capacidad

	https://www.mediafire.com/file/mtcf6k0fp7xpd75/ArchLinux-h2-plus-arm-devel-1.3-Banana-pi-m2-zero.rar/file


## Referencias
	https://wiki.archlinux.org/title/Banana_Pi#Compile_and_copy_U-Boot_boot_loader
	https://forum.banana-pi.org/t/archlinux-for-bpi-m2-zero/5624
	https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases
	https://gist.github.com/avafinger/4d1b654a6cdb5d6e05218d013fe94d0a
	https://github.com/xqdzn/alarm-bpi-p2z
	https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/blob/master/linux-image-4.20.0-rc3-m2z-otg-gadget_1.0-16.deb
	https://www.reddit.com/r/archlinuxarm/comments/a9sa4p/arch_linux_arm_build_system/
	https://github.com/BPI-SINOVOIP/BPI-M2Z-bsp
	https://archlinuxarm.org/platforms/armv7/allwinner/cubieboard-2#installation
	https://bitkeks.eu/blog/2017/02/arch-linux-arm-on-banana-pi-compiled-from-arch-linux.html
	https://wiki.archlinux.org/title/NetworkManager