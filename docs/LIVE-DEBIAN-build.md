# Debian Live "live-build" - instrucciones rapida como construir
====================================================

Construira un Debian con escritorio LXDE y minimas dependencias, es interactivo y requeire internet.

Inicia una teminal consola, se asumira el directoro de comeinzo en la consola `/media/data/LIVEBUILD`.

## Instalar preliminares
------------------------

* 20Gb de espacio
* internet minimo 100Kbps
* 1GHz CPU minimo
* 1Gb swap minimo

#### 1 Instalar live-build.

``` bash
su
apt-get update
apt-get install live-build chroot rsync xorriso squashfs-tools
```

#### 2 Asegurar directorio con permisos

``` bash
mkdir /media/data/LIVEBUILD
chmod 777 /media/data/LIVEBUILD
echo "asi mas o menos: mount /media/data/ -o remount,rw,dev,suid,exec"
```

Si el directorio es parte de un disco/particion debe estar montado con permisos de ejecucion compeltos, 
ya que se va ejecutar chroot en el.

## Configurar el live build
---------------------------

#### Crear el directorio de la estructura de el disco live

``` bash
cd /media/data/LIVEBUILD
mkdir debian8live
lb clean
```

#### 4 Crear el archivo de ejecucion de script de live.build

``` bash
cat <<EOF > lb-debian8live
#!/bin/bash
lb config \
--mode debian \
--system live \
--interactive shell \
--distribution jessie \
--debian-installer live --debian-installer-gui true \
--architecture i386 \
--archive-areas "main contrib non-free" \
--security true --updates true --backports true \
--linux-flavours "586" \
--linux-packages "linux-image linux-headers" \
--apt-recommends false --apt-secure false \
--checksums none \
--iso-publisher "VENENUX" \
--binary-images iso-hybrid \
--memtest memtest86+ \
--win32-loader false \
--verbose \
--bootappend-live " boot=live config components autologin xautologin nouveau.modeset=0 radeon.modeset=0 " \
--bootappend-live-failsafe "boot=live components noapic noapm nodma nomce nolapic nomodeset nosmp nosplash vga=normal fbcon=rotate:1 acpi_enforce_reources=lax pci=noacpi,assign-busses reboot=cold,hard " \
--bootappend-install "nomce nomodeset nosplash acpi_enforce_reources=lax pci=noacpi,assign-busses noapic idle=pool reboot=cold,hard
EOF
``

**IMPORTATE NOTA 586 vs 686-pae** el instalador grafico solo esta en 486/586 ademas 
de que el modo live casi nadie lo usa sin persistencia, es ilogico usar pae aqui.

#### 5 Lanzar y configurar el entorno lb build

``` bash
chmod 700 lb-debian8live
./lb-debian8live
```

Esto descargara ya que este documento es para la version con internet y basica.

#### 6 ajustar el entorno a construir

* repositorios extras para paquetes multimedia

``` bash
cat <<EOF> config/archives/marilla.list.chroot 
deb http://www.deb-multimedia.org jessie main non-free
deb http://www.deb-multimedia.org jessie-backports main
EOF
```

* grupos del usuario live/instalador:

``` bash
mkdir -p config/includes.chroot/etc/live/config
cat <<EOF> config/includes.chroot/etc/live/config/user-setup.conf
LIVE_USER_DEFAULT_GROUPS="audio cdrom dip floppy video plugdev netdev powerdev scanner bluetooth fuse lp disk games"
EOF
```

* preconfiguracion de instalador

``` bash
cat <<EOF> config/includes.installer/preseed.cfg
d-i debian-installer/locale string es_VE
d-i localechooser/supported-locales multiselect en_US.UTF-8, ru_RU.UTF-8, es_DO.UTF-8, es_VE.UTF-8
d-i passwd/root-login boolean false
d-i passwd/user-fullname string general
d-i passwd/username string general
d-i passwd/user-password password general
d-i passwd/user-password-again password general
d-i passwd/user-uid string 1010
d-i passwd/user-default-groups string audio cdrom video netdev powerdev fuse lp dip floppy games
d-i clock-setup/ntp boolean false
d-i netcfg/enable boolean false
d-i netcfg/get_hostname string venenuxpos
d-i netcfg/get_hostname seen false
d-i partman-auto/choose_recipe select atomic
d-i partman/mount_style select uuid
d-i partman/default_filesystem string ext3
d-i base-installer/install-recommends boolean false
d-i base-installer/kernel/image string linux-image-686-pae
d-i apt-setup/use_mirror boolean false
d-i apt-setup/non-free boolean true
d-i apt-setup/contrib boolean true
d-i debian-installer/allow_unauthenticated boolean true
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i debian-installer/add-kernel-opts string nomsi acpi_enforce_resources=lax 
EOF
```

## construir el entorno 
--------------------------

#### 7 Ejecutar el buil:

`lb build`

Y_a que se especificos `--interactive shell` en el script iniciará el proceso de compilación 
descargando todo lo que necesita y juntando las cosas, eventualmente volcándolo a medio camino 
a un entorno chroot, este es el interactive shell

**ATENCION** Se abrira un shell bash donde podra manualmente installar paquetes con apt o aptitude

#### 8 Paquetes live escritorio, base, webserver e impresion

Escritorio base y cups

``` bash
apt-get install task-ssh-server task-laptop laptop-mode-tools task-print-server acpi bluetooth 
apt-get install net-tools udev wireless-tools dialog dbus
apt-get install xdg-user-dirs-gtk xdg-utils sakura
```

Escritorio LXDE

``` bash
apt-get install task-lxde-desktop openbox fbpanel obsession python-xdg
apt-get install lightdm dconf-editor atril pidgin brasero engrampa galculator system-config-printer
```

Sistema de impresion completo

``` bash
apt-get install cups cups-daemon bluetooth bluez-cups bluez-tools cups-pk-helper hplip
apt-get install system-config-printer system-config-printer-udev 
apt-get install printer-driver-cups-pdf printer-driver-*
apt-get install foomatic-db-compressed-ppds foomatic-db-engine printer-driver-all ssh-askpass-gnome xauth
```

Sistema servidor web php, desarrollo y base de datos con el navegador

``` bash
apt-get install php5 php5-mysql php5-gd php5-intl, php5-odbc, php5-mcrypt php5-curl 
apt-get install apache2 libapache2-mod-php5 php-pear
apt-get install mariadb-server mariadb-client percona-xtrabackup
apt-get install git git-core sqlite3 firefox-esr flashplayer-mozilla xul-ext-adblock-plus
apt-get install gambas3 geany geany-plugins* mysql-workbench pgadmin3 unixodbc
```


#### Ajuste de entorno live antes de terminar

* dentro de /etc/Network-manager/Network-manager.conf y cambiando managed = false a managed = true para que network-manager pueda administrar las interfaces de red. 
* configurar los parámetros de usuario predeterminados de la compilación. Para ello, copie todo el contenido del directorio de inicio de un usuario en / live / chroot / etc / skel
* instale paquetes que no se encuentran dentro de los repositorios de Debian. como por ejemplo el anydesk o el softwamker office 

#### 9 Paquete importantes

``` bash
apt-get install live-boot live-toos live-config-systemd 
apt-get install firmware-linux-nonfree firmware-ralink firmware-realtek firmware-atheros firmware-iwlwifi firmware-brcm80211 firmware-atheros
apt-get install task-ssh-server task-laptop laptop-mode-tools 
apt-get install live-tools debian-installer-laucher
apt-get install net-tools udev wireless-tools 
apt-get install live-config-systemd pciutils linux-image linux-headers
apt-get install grub2-common os-prober
apt-get install console-setup accountsservice user-setup locales
```

**IMPORTATE** si no instala algunos de los task packages de escritorio por ejemplo live-mate-desktop

#### 9 Limpiar y clave root antes terminar

Se configura el clave de root y despues se limpia y sale con `exit` asi

``` bash
passwd
rm /var/lib/dbus/machine-id && rm -rf /tmp/*
passwd
apt-get clean
exit
```

Despues que termina de instalar todos los paquetes, sale de `aptitude` o de `apt`, 
con `exit` se sale de la consola y **automaticamente al salir live-build terminara todo el proceso** final.

Acto seguiido de salir de el entorno chroot actualiza las repositorios, 
instala el instalador de las imagenes

**IMPORTANTE** Este paso tardara bastante, aqui construye el iso, pasando por 
la creacion del archivo squasfs y copia del chroot ademas de paquetes base 
para el cd root fs

una ves listo se tiene un resultado ISO que sirve tambien para pendrive

## Modificar el trabajo actual sin perder lo realizado

PAra modificar el trabajo realizado no es necesario esperar descargar todo los paquetes de nuevo.. 
solo hay que limpiar la parte binarya, y se conservara el trabajo:

`lb clean --binary`

y despues normal `lb build` y esta vez si prosigue usando lo ya realizado

## Error y se para el trabajo lb build con binary, pero ya tenemos chroot fino

si ocurre un error en tiempo de "build" y se intenta retomar, la compilacion del iso no ocurre:

``` bash
P: Begin building binary iso image...
mv: no se puede efectuar `stat' sobre «binary»: No existe el fichero o el directorio
P: Begin unmounting filesystems...
P: Saving caches...
```

esto es un bug y se resulte segun esto: https://lists.debian.org/debian-live/2014/08/msg00047.html borrando manualmente, 

`rm -rf chroot/binary`
`lb clean --binary`

y despues normal `lb build` y esta vez si prosigue

