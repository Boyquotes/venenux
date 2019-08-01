# Debian 10 VenenuX X
===========================================================

Debian 10 es especial porque cambios fuertes.. como `calamares` el instalador, 
systemd 100% integrado, y nuevas maneras de trabajar el sistema. Este documento 
se centra en el escritorio LXDE y MATE es decir solo apariencia GTK unicamente.

* [Documentacion Debian 10 LXDE/MATE](#documentacion-debian-10-lxde-mate)
* [Caracteristicas instalacion](#caracteristicas-instalacion)
* [Configurar el Sistema](#configurar-el-sistema)
* [Programas repositorios y paquetes](#programas-repositorios-y-paquetes)
* [Escritorios LXDE y MATE](#escritorios-lxde-y-mate)
  * [Internet con VPN y comunicacion de chats](#internet-con-vpn-y-comunicacion-de-chats)
* [Gestion remota](#gestion-remota)
  * [nomachine y anydesk](#nomachine-y-anydesk)
* [Desarrollo LAMP web](#desarrollo-lamp-web)
* [Desarrollo chaintoolkit C y Go](#desarrollo-chaintoolkit-c-y-go)
* [Desarrollo chaintoolkit C y Go](#desarrollo-chaintoolkit-c-y-go)
* [Juegos y diversion](#juegos-y-diversion)
  * [Ajedrez](#ajedrez)
  * [Emuladores](#emuladores)
* [ANEXOS repos y extras](#anexos-repos-y-extras)
  * [Debian y los ppa](#debian-y-los-ppa)

# Debian 10 - descargas y disponibles

Debian 10 hay dos versiones la live y la instalar, de estas hay 
dos sabores, la que es libre y la que incluye soporte para todo, 
esta ultima es la descarga aqui señalada:

| Nombre         | Descripcion que trae y que instala            | Descarga    |
| -------------- | --------------------------------------------- | ----------- |
| Debian 10 mini | Solo base requiere internet si desea el resto | https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.0.0+nonfree/amd64/iso-cd/firmware-10.0.0-amd64-netinst.iso |
| Debian 10 LXDE | Sin internet el escritorio LXDE(combine con MATE) | https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.0.0-live+nonfree/amd64/iso-hybrid/debian-live-10.0.0-amd64-lxde+nonfree.iso |
| Debian 10 MATE | Trae el escritorio MATE (combinelo con gnome) | https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.0.0-live+nonfree/amd64/iso-hybrid/debian-live-10.0.0-amd64-mate+nonfree.iso |
| Debian 10 DVD  | Cuando instala escoges sin internet escritorios | https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.0.0+nonfree/amd64/iso-dvd/firmware-10.0.0-amd64-DVD-1.iso |
| Debian 10 LXQT | Sin internet el escritorio QT (combine con KDE5) | https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.0.0-live+nonfree/amd64/iso-hybrid/debian-live-10.0.0-amd64-lxqt+nonfree.iso |

Las LXDE, LXQT y MATE son "live" es decir, puede usarse sin instalar ni usar el discoduro 
listas para usar, incluso sin instalar

**EJEMPLOS** 

* Si instala LXQT, puede combinar instalando aplicaciones de KDE5
* Si instala LXDE, puede combinar instalando aplicaciones de MATE

## Documentacion Debian 10 LXDE/MATE

Esto es para la maquina que estaria o esta en un escritorio, asumimos LXDE o MATE 

#### Servicios que se sugieren siempre activos en su red

| servicios | puerto   | software | expuesto-ip                   | objetivo      |
| --------- | -------- | -------- | ----------------------------- | ------------- |
| ssh       | 19226    | openssh  | **SI**, usado para emergencias| para gestion remoto    |
| NX        | 4000     | nomachine  | **SI**, usado para gestion remota | para gestion remoto  grafico  |

#### Usuarios Importantes de la instalacion

| tipo          | usuario             | claves   |
| ------------- | ------------------- | -------- |
| administrador | root                | root |
| remoto        | daru                | daru |
| general       | general             | 1234 |
| sistema (op)  | systemas            | 1234 |

**NOTA** El sistema no ofrece nada a la internet y no se habilita nada hacia afuera

-------------------------------------------------------------------------------------------------------

## Caracteristicas instalacion

Se asume instalacion normal de disco Debian 10 AMD64 con firmware, 
para maquinas viejas usar i386 Debian 7 o 9 unicamente.

**Debe haber un usuario llamado `general` siempre en la instalacion**

#### discos duros y particiones recommendada

`Disk /dev/sda:  37 GB, 500105249280 bytes VIRTUAL`

| Device | Monta/objetivo  | Start  |   End  | Tamanio/Bytes | Tipo |
| ------ | --------------- | ------ | ------ | ------------- | ---- |
| part1  |  /              |     1  | 3917   |           30G | ext3 |
| part2  |                 |        |       |           2G | ext3 |
| part3  |  /home          |        |       |         lo que sobre | ext3 |

## Configurar el Sistema

**ADVERTENCIA** a partir de debian 10 todo comando administrativo debe tener ruta absoluta

#### nombre de host o maquina

* pcr : personal computer real (si v virtual, sig que es un equipo fisico)
* amd64: significa que es 64bit x86 compatible
* deb10: significa que es debian version 10
* -1 : significa que es la primera maquina instalada con este sistema operativo

```
hostname pcr-amd64-deb10-1
echo "pcr-amd64-deb10-1" > /etc/hostname
sed -i -r 's#127.0.1.1.*#127.0.1.1  pcr-amd64-deb10-1#g' /etc/hosts

/usr/sbin/telinit u
/usr/sbin/init u
```

#### escalado privilegios de su

Hay que colocar una clave a la cuenta de root, asi:

```
sudo su
passwd
cp /bin/su /bin/miluser
chmod o-rxw /bin/su
chmod g-rxw /bin/su
chmod u+rxws /bin/miluser
```

#### Independencia de sudo

Una vez con una clave valida, cambiamos sudo para proteger

```
apt-get -y install sudo
mv /usr/bin/sudo /bin/administrator
chmod u+s /bin/administrator
```

#### configuracion interfaz red

**IMPORTANTE** si ya tiene internet y es una virtual o AWS no tocar y saltar esta seccion

`ip -4 addr`

Las ip cambian cada vez se apaga y prende su computador

| tipo     | antes       | ahora      | ejemplo |
| -------- | ----------- | ---------- | ------- |
| ethernet | eth         | en         | enp2s0    |
| serial/slip | slip     | sl         | slip0s1   |
| inalambr | wlan        | wl         | wlan0s0   |
| wan inalam | wwan      | ww         | wwan0   |
| punto punto | ppp      | pp         | ppp0    |
| privada  | vpn         | vn         | vpn0    |


## Programas repositorios y paquetes

Desde buster es necesario parametro de aceptar el repo inseguro

#### ajustes repositorios

Ajustar preferencias de installacion

``` bash
cat > /etc/apt/apt.conf.d/50venenuxcustom << EOF
APT::Install-Recommends "0";
APT::Install-Suggests "0";
APT::Get::AllowUnauthenticated "true";
EOF
```

Adicionar repositorios principal, backports y multimedia

```
apt-get update
apt-get install base-files lsb-release

rm -f /etc/apt/sources.list

cat > /etc/apt/sources.list.d/50debianoficial.list << EOF
deb http://http.us.debian.org/debian $(lsb_release -s -c) main contrib non-free
deb http://ftp.de.debian.org/debian $(lsb_release -s -c) main contrib non-free
deb http://ftp.de.debian.org/debian $(lsb_release -s -c)-updates main contrib non-free
EOF

cat > /etc/apt/sources.list.d/51debianbackports.list << EOF
deb http://ftp.us.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free
deb http://ftp.de.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free
EOF

cat > /etc/apt/sources.list.d/52debianmultimedia.list << EOF
deb http://www.deb-multimedia.org $(lsb_release -s -c) main non-free
deb http://www.deb-multimedia.org $(lsb_release -s -c)-backports main non-free
deb http://debian-mirrors.sdinet.de/deb-multimedia $(lsb_release -s -c) main non-free
deb http://debian-mirrors.sdinet.de/deb-multimedia $(lsb_release -s -c)-backports main non-free
deb http://debian.ids-services.de/debian-multimedia $(lsb_release -s -c) main non-free
EOF

cat > /etc/apt/sources.list.d/53debianlamphp.list << EOF
deb https://packages.sury.org/php/ $(lsb_release -sc) main
EOF

apt-get  --allow-unauthenticated -o Acquire::AllowInsecureRepositories=true update


cat > /etc/apt/preferences.d/50debianbackports << EOF
Package: *
Pin: release n=$(lsb_release -s -c)-backports
Pin-Priority: 500
EOF

cat > /etc/apt/preferences.d/90_mailsuite_default << EOF
Package: exim4*
Pin: release *
Pin-Priority: 1

Package: courier*
Pin: release *
Pin-Priority: 700
EOF
```

#### paquetes base systema

```
apt-get update
apt-get -y --force-yes install aptitude
apt-get -y --force-yes install wget less groff bzip2 lrzip lzop lsof linux-base \
 ca-certificates curl perl perl-base perl-modules fam libfam0 \
 python python-minimal python2.7 python2.7-minimal mime-support \
 apt-transport-https lsb-release
```

#### paquetes base usuarios

```
apt-get -y --force-yes install \
 graphicsmagick-imagemagick-compat htop
```

#### Usuario de acceso general "daru"

Usuario "daru" es 444 y no 1000 ademas no tiene home y no tiene shell normal:

``` bash
apt-get -y install csh
/usr/sbin/adduser --uid 444 --home /opt/daru --shell /bin/csh daru
rm /opt/daru/*
```

## Escritorios LXDE y MATE

**ADVERTENCIA** para homoheneidad no combine estos escritorios con otros solo escoja uno aqui

#### LXDE (combinado con MATE)

Escritorio LXDE con partes mejoradas desde MATE

```
apt-get -y --force-yes install materia-gtk-theme \
 lxde usermode task-lxde-desktop notify-osd mate-polkit-bin lxlock lxsession-default-apps \
 lxhotkey-gtk lxhotkey-plugin-openbox openbox obsession lxappearance-obconf \
 mate-polkit gnome-system-tools mesa-utils hardinfo pnmixer \
 mate-power-manager atril engrampa eom mate-applet-brisk-menu \
 mate-system-monitor mate-utils pluma mate-screensaver \
 mate-desktop-environment-extras
```

Manejador de archivos con funciones completas (buscar, enviar etc)

```
apt-get -y --force-yes install sinple-scan \
 caja caja-admin caja-image-converter caja-open-terminal caja-rename caja-sendto
 pcmanfm gvfs-backends gvfs-fuse libfm-modules libfm-tools
```

#### Internet con VPN y comunicacion de chats

En debian aunque hay muchos navegadores web solo dos sabemos abriran cualqueir pagina, 
ademas tambien se puede **agregar chrome aparte de firefox y chromium**

**ADVERTENCIA** esto depende de los repositorios aqui configurados multimedia

Conectividad y VPN:

```
apt-get -y --force-yes install \
network-manager network-manager-config-connectivity-debian \
mobile-broadband-provider-info usb-modeswitch-data \
usb-modeswitch modemmanager crda pptp-linux vpnc-scripts \
network-manager-strongswan strongswan \
network-manager-pptp network-manager-pptp-gnome \
network-manager-l2tp network-manager-l2tp-gnome \
network-manager-vpnc network-manager-vpnc-gnome vpnc \
network-manager-openvpn network-manager-openvpn-gnome openvpn \
network-manager-openconnect network-manager-openconnect-gnome openconnect \
network-manager-fortisslvpn network-manager-fortisslvpn-gnome openfortivpn \
strongswan-nm
```

Navegacion internet y chat:

```
apt-get -y --force-yes install \
 chromium chromium-sandbox chromium-l10n flasplayer-chromium \
 firefox-esr libavcodec58 libcanberra0 firefox-esr-l10n-es* flasplayer-mozilla \
 webext-https-everywhere
 pidgin pidgin-plugin-pack pidgin-encryption pidgin-extprefs \
 pidgin-skype pidgin-guifications finch pidgin-mpris purple-discord
 telegram-desktop
```

#### Multimedia

**ADVERTENCIA** esto depende de los repositorios aqui configurados multimedia

```
apt-get -y --force-yes install ffmpeg ffplay ffhevc ffx264 ffxvid lame \
 mediainfo mediainfo-gui a2ps unoconv chm2pdf dos2unix e2ps odt2txt \
 mpv oss-compat gst123 mpg123 mpg321 mpv mplayer libdvdcss lsdvd aacplusenc amrenc \
 deadbeef deadbeef-infobar deadbeef-rating deadbeef-vu-meter deadbeef-waveform-seekbar \
 gstreamer1.0-libav gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly \
 gstreamer1.0-alsa gstreamer1.0-x gstreamer1.0-gl python-gst pyhton3-gst \
 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-gtk3 \
 libgstreamer-plugins-base1.0 libgstreamer-plugins-bad1.0 libgstreamer-gl1.0 \
 cmus cmus-plugin-ffmpeg moc moc-ffmpeg-plugin gstreamer1.0-tools \
 xcfa lame sox mppenc audiotools faad faac mp4v2-utils mpegtools-gtk gpac mp3fs \
 audacity mcp-plugins gespeaker mbrola mbrola-vz1 mbrola-us1 quicktime*
```

#### Ediccion audio y video

**ADVERTENCIA** esto depende de los repositorios aqui configurados multimedia

```
apt-get install smplayer qmmp udisk2 gmpt gnomad2 gtkpod \
 mediainfo mediainfo-gui a2ps unoconv chm2pdf dos2unix e2ps odt2txt \
 xcfa lame sox mppenc audiotools faad faac mp4v2-utils mpegtools-gtk gpac mp3fs \
 qwinff ffmulticonverter mixx parlatype kdenlive audacity subtitleeditor
```

## Gestion remota

En linux los mejores clientes y servicios son anydesk y nomachine, nomachine proveee todo incluso ssh.

#### nomachine y anydesk

```
apt-get install libcairo2 libgl1-mesa-glx libgl1 libgl1-mesa-dri libglu1-mesa \
  libglu1 libgtkglext1 libice6 libsm6 libx11-xcb1 libxcb-shm0 libxcb1 libxdamage1 \
  libxext6 libxfixes3 libxi6 libxmu6 libxrandr2 libxtst6 avahi-daemon udev udisks2 \
  libusb-1.0-0-dev libusb-dev gcc libtool make

wget https://download.nomachine.com/download/6.7/Linux/nomachine_6.7.6_11_amd64.deb

dpkg -i *.nomachine*.deb

wget https://download.anydesk.com/linux/anydesk_5.1.0-1_amd64.deb

dpkg -i anydesk_5.1.0-1_amd64.deb
```

#### Servicio ssh ajustes

```
apt-get -y install openssh-server openssh-client
sed -i -r 's#.*Port 22.*#Port 1922#g' /etc/ssh/sshd_config
sed -i -r 's#PermitRootLogin.*#PermitRootLogin no#g' /etc/ssh/sshd_config
/usr/sbin/service ssh restart
```

**ADVERTENCIA** ESTO HARA QUE SSH REQUIERA SIEMPRE USAR `-p 1922` PARA USAR SU MAQUINA REMOTAMENTE!

**IMPORTANTE** es una medida de seguridad.

## Desarrollo LAMP web

Esto es Linux con Apache, Mysql (ahora MariaDB) y Php, para desarrollar 
se asumira **un directorio llamado Devel en el home del usuario** y todo 
lo que este en Devel sera "servido" por apache y lighty al mismo tiempo.
(esto ultima combinacion no es posible realizarla en centos facilmente)

#### directorio de desarrollo

```
mkdir /etc/skel/Devel
for i in $(ls /home);do mkdir /home/$i/Devel;done
for i in $(ls /home);do chmod 777 /home/$i/Devel;done
for i in $(ls /home);do touch /home/$i/Devel/.keep;done
```

#### Editores y herramientas IDE, DIFF y CVS

* **geany** es un editor de texto con capacidad de IDE, git y terminal incluida embebida
* **mysqlworkbench** es un completo IDE para SQL enfocado en mysql, siver para otros
* **meld** es un editor diferencias entre archvos, mostrando en que lineas difieren
* **giggle** no necesita presentacion, es el visor y manejador de los **git** repos
* **rapidsvn** es un manejador grafico de los **svn** repos de subversion

```
apt-get install geany geany-plugins meld git subversion rapidsvn giggle mysql-workbench
```

#### web server lighttpd y apache2

**NOTA** apache ira por 8080 mientras que lighttp sera el servidor web principal

```
apt-get -y install openssl lighttpd lighttpd-mod-magnet spawn-fcgi apache2-utils
/usr/sbin/lighty-enable-mod cgi dir-listing accesslog fastcgi status userdir usertrack
/usr/sbin/service lighttpd restart

apt-get -y install openssl apache2 apache2-bin apache2-utils
sed -i -r 's|Listen 80|Listen 8080|g' /etc/apache2/ports.conf
sed -i -r "s|<VirtualHost .*:80>|<VirtualHost \*:8080>|g" /etc/apache2/sites-enabled/000-default.conf
/usr/sbin/service apache2 restart
```

#### Base de datos MariaDB (MySQL)

```
apt-get install mariadb-server mariadb-client mariadb-common
```

#### php 5 y 7 ambos en Debian 10

Repositorio

```
apt-get -y install ca-certificates apt-transport-https 
wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add -
cat > /etc/apt/sources.list.d/50debiasuryphp56.list << EOF
deb https://packages.sury.org/php/ $(lsb_release -s -c) all
EOF
apt-get update
```

Instalacion php 5.6

```
apt-get -y install php5.6-common libapache2-mod-php5.6 \
  php5.6-readline php5.6-recode  php5.6-imap \
  php5.6-odbc php5.6-sybase php5.6-bz2 php5.6-zip php5.6-xml \
  php5.6-mbstring php5.6-xmlrpc php5.6-sqlite3 php5.6-pgsql php5.6-intl \
  php5.6-xsl php5.6-curl php5.6-json php5.6-bcmath php5.6-gd \
  php5.6-cli php5.6-enchant php5.6 php5.6-cgi
```

Instalacion php 7.3

```
apt-get -y install php7.3-common libapache2-mod-php7.3 \
  php7.3-readline php7.3-recode  php7.3-imap \
  php7.3-odbc php7.3-sybase php7.3-bz2 php7.3-zip php7.3-xml \
  php7.3-mbstring php7.3-xmlrpc php7.3-sqlite3 php7.3-pgsql php7.3-intl \
  php7.3-xsl php7.3-curl php7.3-json php7.3-bcmath php7.3-gd \
  php7.3-cli php7.3-enchant php7.3 php7.3-cgi
```

Configuracion de todos los php

```
sed -s -i -r 's|.*cgi.fix_pathinfo=.*|cgi.fix_pathinfo=1|g' /etc/php*/*/php.ini
sed -s -i -r 's#expose_php =.*#expose_php = Off#g' /etc/php*/*/php.ini
sed -s -i -r 's#memory_limit =.*#memory_limit = 256M#g' /etc/php*/*/php.ini
sed -s -i -r 's#upload_max_filesize =.*#upload_max_filesize = 56M#g' /etc/php*/*/php.ini
sed -s -i -r 's#post_max_size =.*#post_max_size = 128M#g' /etc/php*/*/php.ini
sed -s -i -r 's#^file_uploads =.*#file_uploads = On#g' /etc/php*/*/php.ini
sed -s -i -r 's#^max_file_uploads =.*#max_file_uploads = 12#g' /etc/php*/*/php.ini
sed -s -i -r 's#^allow_url_fopen = .*#allow_url_fopen = On#g' /etc/php*/*/php.ini
sed -s -i -r 's#^.default_charset =.*#default_charset = "UTF-8"#g' /etc/php*/*/php.ini
sed -s -i -r 's#^.max_execution_time =.*#max_execution_time = 150#g' /etc/php*/*/php.ini
sed -s -i -r 's#^max_input_time =.*#max_input_time = 90#g' /etc/php*/*/php.ini
sed -s -i -r 's#^.date.timezone =.*#date.timezone = "America/Caracas"#g' /etc/php*/*/php.ini
```

Habilitar solo php 5.6 en ambos servidores

```
/usr/sbin/lighttpd-enable-mod fastcgi fastcgi-php
update-alternatives --set php-cgi /usr/bin/php-cgi5.6
/usr/sbin/service lighttpd restart

/usr/sbin/lighttpd-enable-mod fastcgi fastcgi-php
/usr/sbin/a2dismod php7.3
/usr/sbin/a2enmod php5.6
/usr/sbin/service apache2 restart
```

Habilitar solo php 7.3 en ambos servidores

```
/usr/sbin/lighttpd-enable-mod fastcgi fastcgi-php
update-alternatives --set php-cgi /usr/bin/php-cgi7.3
/usr/sbin/service lighttpd restart

/usr/sbin/a2dismod php5.6
/usr/sbin/a2enmod php7.3
/usr/sbin/service apache2 restart
```

#### User dir en apache y lighttpd para Devel

HAbilitar para cada usuario que el directorio `Devel` propio sea las paginas web:

```
mkdir /etc/skel/Devel
for i in $(ls /home);do mkdir /home/$i/Devel;done
for i in $(ls /home);do chmod 777 /home/$i/Devel;done
for i in $(ls /home);do touch /home/$i/Devel/.keep;done

sed -s -i -r "s|.*UserDir.*_.*|\tUserDir Devel|g" /etc/apache2/mods-available/userdir.conf
sed -s -i -r "s|.*<Directory.*home.*|\t<Directory /home/*/Devel>|g" /etc/apache2/mods-available/userdir.conf
sed -s -i -r "s|.*php_admin_value engine.*|\t#php_admin_value engine On|g" /etc/apache2/mods-available/php*.conf
/usr/sbin/a2enmod userdir usertrack lua md dir
/usr/sbin/service apache2 restart

/usr/sbin/lighty-enable-mod fastcgi-php cgi dir-listing fastcgi userdir usertrack
sed -s -i -r 's|.*userdir.path.*|userdir.path         = "Devel"|g' /etc/lighttpd/conf-enabled/10-userdir.conf
/usr/sbin/service lighttpd restart
```

**ADVERTENCIA** apache2 estara como `http://localhost:8080/` entonces todo 
lo que el usuario `general` ponga en `/home/general/Devel` se vera en `http://localhost:8080/~general/`

**IMPORTANTE** lighttpd es el servidor principal como `http://localhost`
lo que el usuario `general` ponga en `/home/general/Devel` se vera en `http://localhost/~general/`


## Desarrollo chaintoolkit C y Go

Para trabajar con dispositivos debe estar en el grupo correcto:
para mas info aqui: https://wiki.debian.org/SystemGroups#Other_System_Groups

* para serial,usb: dialout,dip,fax,plugdev,disk
* para administracion: disk,staff,netdev,plugdev,sudo
* para uso normal con acceso: lp,lpadmin,sudo,video,audio,users,cdrom

#### Activar usuarios para usar dispostivos y servicios

```
for i in $(ls /home);do \
/usr/sbin/usermod -a -G disk,staff,src,dialout,dip,fax,voice,netdev,plugdev,scanner,lp,lpadmin,sudo,video,audio,users,cdrom $i;done
```

#### Directorio de desarrollo

```
mkdir /etc/skel/Devel
for i in $(ls /home);do mkdir /home/$i/Devel;done
for i in $(ls /home);do chmod 777 /home/$i/Devel;done
for i in $(ls /home);do touch /home/$i/Devel/.keep;done
```

#### Gcc y Golang

```
apt-get install build-essential gdc gfortran gobjc libtool manpages-dev golang 
```

#### Java 8 oracle


```
echo 'deb http://download.opensuse.org/repositories/home:/vegnuli:/java8/Debian_9.0/ /' > /etc/apt/sources.list.d/home:vegnuli:java8.list
apt-get update
apt-get install oracle-java8-jdk 
```

#### Mono develop

```
apt-get install mono-complete mono-csharp-shell mono-xbuild mono-source
```

## Juegos y diversion

Es estiupido colocar aqui un apartado tan amplio ademas de que ninguna distro tiene buenos juegos
asi que solo se listas cosas pequeñas de rapido uso:

#### buscaminas maghong y hex-a-hop

```
apt-get install numptyphysics hex-a-hop gnome-mines gnome-sudoku gnome-nibbles gnome-mahjong gamine
```

#### Ajedrez

Esta seccion es minimalista, ajedrez requiere para stockfish configurar polypo

```
apt-get install gnome-chess hoichess gnuchess-book fairmax
```

#### Emuladores

Dosbox emulara MSDOS pero virtualbox es extremo pesado dado usa qt5, se recomienda qemu pero es dificil de configurar

```
apt-get install simh dosbox qemu-system-gui qemu-system-x86 qemu-user qemu-binfmt qemu-utils
```

Mednafen provee todo lo necesario y escala perfecto con muchos filtros, 
su configuracion es homogenea y tieen un interfaz grafica, quedando solo 
uno que otro mas avanzado.
Este cuadropresenta usando los paquetes Debian oficiales los sistemas emulados, 
para mejores opciones se recomienda usar "VenenuX emulation X"

| emulador/emula: | ata | pce | nes | snes | smd | sms | sgg | gba | gbc | n64 | psx | satr | ngp | ngc | nds | ws | wsc |
| --------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| blastem         |     |     |     |     |  x  |  x  |  x  |     |     |     |     |  x  |     |     |     |     |     |
| desmume         |     |     |     |     |     |     |     |     |     |     |     |     |     |     |  x  |     |     |
| mednafen        |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |     |  x  |     |  x  |  x  |     |  x  |  x  |
| pcsxr           |     |     |     |     |     |     |     |     |     |     |  x  |     |     |     |     |     |     |
| mupen64plus     |     |     |     |     |     |     |     |     |     |  x  |     |     |     |     |     |     |     |
| cen64 (DD64)    |     |     |     |     |     |     |     |     |     |  x  |     |     |     |     |     |     |     |
| yabause         |     |     |     |     |     |     |     |     |     |     |     |  x  |     |     |     |     |     |

```
apt-get install mednafe mednafen desmume pcsxr mupen64plus-ui-console cen64 yabause blastem
```

Los siguientes emuladores no son recomendados debido a los detalles mostrados o por consumir muchos recursos:

| emulador/emula:  | ata | pce | nes | snes | smd | sms | sgg | gba| gbc | n64 | psx | satr | ngp | ngc | nds | wii | ws | wsc |
| ---------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| osmose-emulator  |     |     |     |     |  x  |  x  |  x  |     |     |     |     |     |     |     |     |     |     |
| higan (*)        |     |  x  |  x  |  x  |     |  x  |  x  |  x  |  x  |     |     |     |     |     |     |     |  x  |  x  |
| mame (*)         |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |     |  x  |  x  |
| libretro (*)(**) |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |  x  |     |  x  |  x  |
| dolphin-emu (e)  |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |  x  |     |     |
| hatary (i)       |  x  |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| mgba (*)         |     |     |     |     |     |     |     |  x  |  x  |     |     |     |     |     |     |     |     |     |
| nestopia (i)     |     |     |  x  |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| stella (i)       |  x  |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |     |

* osmose es ridiculo requiriendo qt5 solo para emular consolas tan pobres
* retroarch (libretro) es muy completo y configurable pero requiere buena maquina
* higan es muy configurable pero requiere buena grafica
* (*) rendimiento pobre porque consume muchisimos recursos en maquinas potente es buena opcion
* (**) requiere modulos extras que no son iguales que su original
* (e) emulacion no completa por ser mas windosero funciona limitado
* (i) incomodo y dificil de configurar adicional no muy integrado al sistema

## ANEXOS repos y extras
 

#### Debian y los ppa

VenenuX tiene un buen documento para aquellos que deseen experimentar mexclando paquetes, 
el siguiente documento intenta orientar cuando puede mexclar repositorios de otras 
distros basadas en Debian de las mas populares..

[Debian repositorios ppa equivalentes](z-guides-packages-debian-repository-ppa-equivalent.md)

**IMPORTANTE** nunca sustituya paquetes core como `libc6`, `udev` o `dbus` !
