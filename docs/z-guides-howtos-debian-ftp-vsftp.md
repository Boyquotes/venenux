# FTP - vsftp VenenuX Debian

Este documento **muestra como iniciar instalar y desplegar vsftp con Debian** en un debian cualquiera, 
su unica diferencia radica en la configuracion de la gestion de base de datos, asumiremos los tres casos, 
tanto mysql como postgres asi como sqlite o pam+users del sistema.

## 1. Instalation vsftp

Debian wheeze and jessie tienen paquetes backports, sin embargo con estos comandos no importa que usara 
automaticamente seleccionara la version backports o el mas nuevo encontrado disponible:

```
apt-get instal lsb-release apt-transport-https

cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://ftp.de.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free
deb http://archive.debian.org/debian-backports $(lsb_release -s -c)-backports main contrib non-free
EOF
apt-get update
apt-get install wget less groff bzip2 lrzip lzop lsof linux-base ca-certificates curl nmap iproute2 netstat

wget -nv https://download.opensuse.org/repositories/home:/mckaygerhard/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/Release.key -O Release.key
apt-key add - < Release.key
rm -f Release.key
cat > /etc/apt/sources.list.d/debianvenenuxvoip.list << EOF
deb https://download.opensuse.org/repositories/home:/mckaygerhard/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
EOF
apt-get update

apt-get -y install vsftp
```

Esto deja el ftp sin ningun acceso pero activo, hay varias maneras de configurarlo pero difieren en dos ramas, 
con usuarios virtuales o con usuarios reales, las siguientes secciones usaran usuarios reales y despues virtuales.

## 2 Configuracion vsftp

### 2.1 Configuracion de cualquier caso pero con SSL/TLS

Debemos generar un certificado, en este caso sera autofirmado, sino genere uno `let's encrypt`, 
la unica diferencia es que tanto la llave privada (privkey.pem) como la publica (fullchain.pem) 
en el caso de autofirmado son el mismo archivo (o estan contenidas en el mismo archivo).. 

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

openssl req -x509 -days 360 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Bolivar/L=Upata/O=VenenuX-server/OU=VenenuX-server/CN=$ipdefval" \
   -keyout "/etc/ssl/certs/$ipdefval.pem" -out "/etc/ssl/certs/$ipdefval.pem"
```

Ahora configurar en el vsftp que use este certificado y habilitar el uso de SSL/TLS en el ftp:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

sed "s|.*rsa_cert_file=.*|rsa_cert_file=/etc/ssl/certs/$ipdefval.pem|g" -i /etc/vsftp.conf
sed "s|.*rsa_private_key_file=.*|rsa_private_key_file=/etc/ssl/certs/$ipdefval.pem|g" -i /etc/vsftp.conf
sed "s|.*ssl_enable=.*|ssl_enable=YES|g" -i /etc/vsftp.conf
sed "s|.*.*||g" -i /etc/vsftp.conf
sed "s|.*.*||g" -i /etc/vsftp.conf
```

