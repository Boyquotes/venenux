 - [README inicial](README.md) - []

# Debian Live "live-build" - instrucciones rapida como construir
====================================================

Construira un Debian con escritorio MATE y minimas dependencias, es interactivo y requeire internet.

UEFI esta soportado oficialmente en 64bits y corregido solamente en debian 8 pero esta info podria ayudar en debians menores: https://askubuntu.com/questions/395879/how-to-create-uefi-only-bootable-usb-live-media

Inicia una teminal consola, se asumira el directoro de comeinzo en la consola `/LIVEBUILD`.

Para debian 7 como base leer el [VNXDOCS-live-howto-livebuild-deb08.md](VNXDOCS-live-howto-livebuild-deb07.md) que aun soprota hardware viejo.

Para debian 8 como base leer el [VNXDOCS-live-howto-livebuild-deb08.md](VNXDOCS-live-howto-livebuild-deb08.md) que aun soprota hardware viejo y algunos nuevos.

Para debian 9 como base leer el [VNXDOCS-live-howto-livebuild-deb09.md](VNXDOCS-live-howto-livebuild-deb09.md) que solo soportara harware nuevo.

**EL proceso aqui es generico leer los documentos citados antes para mas detallados con este repo**


## Instalar preliminares
------------------------

* 20Gb de espacio
* internet minimo 100Kbps
* 1GHz CPU minimo
* 1Gb swap minimo

#### 1 Preparar el entorno instalacion

``` bash
apt-get update
apt-get install lsb-release apt-transport-https

cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://archive.debian.org/debian $(lsb_release -sc)-backports main contrib non-free
deb http://archive.debian.org/debian $(lsb_release -sc)-backports-sloppy main contrib non-free
EOF

cat > /etc/apt/apt.conf.d/01norecommend << EOF
APT::Install-Recommends "0";
APT::Install-Suggests "0";
EOF

cat > /etc/apt/apt.conf.d/91archivelist << EOF
Acquire::Check-Valid-Until "0";
APT::Get::AllowUnauthenticated "true";
EOF

cat > /etc/apt/preferences.d/debian << EOF
Package: *
Pin: origin security.debian.org
Pin-Priority: 500

Package: *
Pin: origin archive.debian.org
Pin-Priority: 500

Package: *
Pin: release l=Debian-Security
Pin-Priority: 500
EOF

cat > /etc/apt/preferences.d/debianbackports << EOF
Package: *
Pin: release l="Debian Backports"
Pin-Priority: 500

Package: linux-image*
Pin: release l="Debian Backports"
Pin-Priority: -1

Package: linux-header*
Pin: release l="Debian Backports"
Pin-Priority: -1

Package: linux-compiler-gcc-4*
Pin: release l="Debian Backports"
Pin-Priority: -1

Package: live-boot*
Pin: release l="Debian Backports"
Pin-Priority: -1
EOF
```

#### 2 Instalar y preparar el live build

``` bash
apt-get update
apt-get -y --force-yes install live-build chroot rsync xorriso squashfs-tools

```

Si el directorio es parte de un disco/particion debe estar montado con permisos de ejecucion compeltos, 
ya que se va ejecutar chroot en el.

## Configurar live build
---------------------------

#### Crear el directorio de la estructura de el disco live

``` bash
mkdir /LIVEBUILD
chmod 777 /LIVEBUILD
echo "si es montura, asegurar mas o menos: mount /LIVEBUILD -o remount,rw,dev,suid,exec"
cd /LIVEBUILD
lb clean
```

#### 4 Crear el archivo de ejecucion de script de live.build

``` bash
cat <<EOF > lb-debianlive
#!/bin/bash
lb config \
--mode debian \
--system live \
--interactive shell \
--distribution jessie \
--debian-installer live --debian-installer-gui true \
--architecture i386 \
--archive-areas "main contrib non-free" \
--security true --updates false --backports true \
--parent-mirror-bootstrap http://archive.debian.org/debian \
--parent-mirror-binary http://archive.debian.org/debian \
--mirror-bootstrap http://archive.debian.org/debian \
--debootstrap-options "--include=apt-transport-https,ca-certificates,openssl" \
--firmware-chroot true  --firmware-binary true \
--linux-flavours "586" \
--linux-packages "linux-image linux-headers" \
--apt-recommends false --apt-secure false \
--apt-options "--yes -oAcquire::Check-Valid-Until=false --allow-unauthenticated" \
--checksums none \
--iso-publisher "VENENUX" \
--binary-images iso-hybrid \
--memtest memtest86+ \
--win32-loader false \
--verbose \
--bootappend-live " boot=live config components autologin xautologin nouveau.modeset=0 radeon.modeset=0 " \
--bootappend-live-failsafe "boot=live components noapic noapm nodma nomce nolapic nomodeset nosmp nosplash vga=normal fbcon=rotate:1 acpi_enforce_reources=lax pci=noacpi,assign-busses reboot=cold,hard " \
--bootappend-install "nomce nomodeset nosplash acpi_enforce_reources=lax pci=noacpi,assign-busses noapic idle=pool reboot=cold,hard"
EOF
```

**IMPORTANTE NOTA 586 vs 686-pae** el instalador grafico solo esta en 486/586 ademas 
de que el modo live casi nadie lo usa sin persistencia, es ilogico usar pae aqui.

**IMPORTANTE NOTA 64bits vs 32bits** no emplear 64bits porque gastara mas RAM, esta 
solo es necesario si tiene UEFI o programara con androit. Consume muucho mas.

**NOTA**: los directorios debianlive[8/9/7] contiene una estructura mas refinada de este manual

#### 5 Lanzar y configurar el entorno lb build

``` bash
chmod 700 lb-debian8live
./lb-debian8live
```

#### 6 adicionar y configurar repositorios live chroot

* 6.0 permitir cualquier repo

``` bash
lb config --apt-options "--yes -oAcquire::Check-Valid-Until=false"

cat > config/apt/apt.conf << EOF
Acquire::Check-Valid-Until "0";
APT::Get::AllowUnauthenticated "1";
APT::Install-Recommends "0";
APT::Install-Suggests "0";
EOF
```

* 6.1 VenenuX repositories para voip 

``` bash
cat > config/archives/venenuxvoip.list.chroot << EOF
deb http://download.opensuse.org/repositories/home:/vegnuli:/voip/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/golang/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/java8/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/minetest/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/emulators/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/devel/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/internet/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
deb http://download.opensuse.org/repositories/home:/vegnuli:/databases/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
EOF
```

* 6.2 debian multimedia marillat

``` bash
cat <<EOF> config/archives/marillat.list.chroot 
deb http://www.deb-multimedia.org jessie main non-free
deb http://www.deb-multimedia.org jessie-backports main
EOF

cat <<EOF> config/archives/marillat.pref.chroot
# we put all backports and normal debian to 500 so then here 501
Package: *
Pin: origin www.deb-multimedia.org
Pin-Priority: 501

# the rest of packages excetp those must have bad conflicting files, only happened with jessie
Package: libva*
Pin: origin www.deb-multimedia.org
Pin-Priority: -1

Package: vdpau-va*
Pin: origin www.deb-multimedia.org
Pin-Priority: -1

Package: va-driver-all
Pin: origin www.deb-multimedia.org
Pin-Priority: -1

Package: vainfo
Pin: origin www.deb-multimedia.org
Pin-Priority: -1
EOF
```

* 6.3 Nodejs 8 y 10

```
cat > config/archives/nodejs.list.chroot  << EOF
deb https://deb.nodesource.com/node_8.x/ $(lsb_release -s -c) main
deb https://deb.nodesource.com/node_10.x/ $(lsb_release -s -c) main
EOF

cat <<EOF> config/archives/nodejs.pref.chroot
Package: *
Pin: origin deb.nodesource.com
Pin-Priority: 501
EOF
```


#### 7 configurar entorno del live construir

* grupos del usuario live/instalador:

``` bash
mkdir -p config/includes.chroot/etc/live/config
cat <<EOF> config/includes.chroot/etc/live/config/user-setup.conf
LIVE_USER_DEFAULT_GROUPS="audio cdrom dip floppy video plugdev netdev powerdev scanner bluetooth fuse lp disk"
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

Cabe destacar que hemos refinado la configuracion, pero el directorio [debianlive8](debianlive8) 
contiene una estructura mas refinada de este manual, puede copiarlo y revisarlo antes de seguir.


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
apt-get install xcowsay cowsay fortunes fortunes-bofh-excuses fortunes-es fortunes-es-off fortunes-ru
```

Escritorio mate

``` bash
apt-get install task-mate-desktop mate-desktop-environment-core caja caja-gksu caja-open-terminal
apt-get install lightdm dconf-editor atril pidgin brasero engrampa galculator system-config-printer
```

Escritorio Xfce

``` bash
apt-get install task-xfce-desktop xfce4
```


Sistema de impresion completo

``` bash
apt-get install cups cups-daemon bluetooth bluez-cups bluez-tools cups-pk-helper hplip
apt-get install system-config-printer system-config-printer-udev 
apt-get install printer-driver-cups-pdf printer-driver-*
apt-get install foomatic-db-compressed-ppds foomatic-db-engine printer-driver-all ssh-askpass-gnome xauth
```

Sistema servidor web php y base de datos con el navegador

``` bash
apt-get install php5 php5-mysql php5-gd php5-intl, php5-odbc, php5-mcrypt php5-curl 
apt-get install apache2 libapache2-mod-php5 php-pear
apt-get install mariadb-server mariadb-client percona-xtrabackup
apt-get install git git-core sqlite3 firefox-esr flashplayer-mozilla xul-ext-adblock-plus
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

**TEMA DE ESCRITORIO CON MATE/XFCE**

Desde GTK3 y Gnome no se puede usar archivos de configuracion ,asi que se dbe en jessie usar dconf


```
apt-get install gtk3-engines-oxygen gtk2-engines-oxygen oxygen-icon-theme dconf-cli

cat <<EOF> config/includes.chroot/etc/skel/.config/autostart/matetheme.desktop 
[Desktop Entry]
Type=Application
Exec=dconf write /org/mate/desktop/interface/gtk-theme "'THEME_NAME'"
Hidden=false
X-MATE-Autostart-enabled=true
Name=osposweb
EOF

cat <<EOF> config/includes.chroot/etc/skel/.config/autostart/matefonts.desktop 
[Desktop Entry]
Type=Application
Exec=gsettings set org.mate.font-rendering antialiasing none
Hidden=false
X-MATE-Autostart-enabled=true
Name=osposweb
EOF
```

Los comando que cambian el tema para todo el escritorio son:

* dconf write /org/mate/marco/general/theme "'THEME_NAME'"
* dconf write /org/mate/desktop/interface/gtk-theme "'THEME_NAME'"
* dconf write /org/mate/desktop/interface/icon-theme "'THEME_NAME'"
* dconf write /org/mate/desktop/peripherals/mouse/cursor-theme "'THEME_NAME'"
* gsettings set org.mate.font-rendering antialiasing none

I found this using the "dconf watch /" command that lists what changes are made by mate-appearance-properties when changing a theme.

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

## Error cuando esta en build, pero ya tenemos chroot fino

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
