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

# 3 Configuracion vsftp+pam

### 3.1 vsftpd+pam+users reales

paquetes (asumiendo estan solo los basicos)

```
apt-get install mysql-client mysql-server vsftpd
```


### 3.1 vsftpd+pam+users virtuales

paquetes (asumiendo estan solo los basicos)

```
apt-get install mysql-client mysql-server openssl vsftpd
```

WIP:

```
htpasswd -c /etc/vsftpd_passwd
openssl passwd -1
cat /etc/vsftp_passwd
username1:hashed_password1
username2:hashed_password2
nano /etc/pam.d/vsftpd

auth required pam_pwdfile.so pwdfile /etc/vsftpd_passwd
account required pam_permit.so

useradd -d /srv/ftp ftpusers
chown ftpusers:ftpusers /srv/ftp

```

# 3 Configuracion vsftpd+Mysql

### 3.1 vsftpd+mysql+realuser

### 3.2 vsftpd+mysql+virtualuser

paquetes (asumiendo estan solo los basicos)

```
apt-get install mysql-client mysql-server mysql-client libpam-mysql vsftpd
```

```
mysql -u root -D mysql
CREATE USER 'elftp'@'localhost' IDENTIFIED BY '***';
GRANT USAGE ON * . * TO 'elftp'@'localhost' IDENTIFIED BY '***' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;
CREATE DATABASE IF NOT EXISTS `elftp` ;
USE elftp;
DROP TABLE IF EXISTS `ftp_users`;
CREATE TABLE IF NOT EXISTS `ftp_users` (
  `usuario` varchar(40) NOT NULL,
  `clave` varchar(80) NOT NULL,
  `estado` varchar(40) NOT NULL DEFAULT 'ACTIVO',
  PRIMARY KEY (`usuario`))
COMMENT='usuarios para el ftp son solo usuario grupales';
GRANT ALL PRIVILEGES ON `elftp` . * TO 'elftp'@'localhost';
```

crear el servicio pam para mysql, para envriptar colocar `crypt=3` y usara md5, aqui se colocar en 0 para fines didacticos:

```
cat /etc/pam.d/vsftpdmysql << EOF
auth required pam_mysql.so user=elftp passwd=elftppass host=localhost db=elftp table=ftp_users usercolumn=usuario passwdcolumn=clave crypt=0
account required pam_mysql.so user=elftp passwd=elftppass host=localhost db=elftp table=ftp_users usercolumn=usuario passwdcolumn=clave crypt=0
EOF
```

editar el vsftpd.conf, nuestro archivo se divide (para seguir a Debian) en dos partes, 
las opciones que ya tenia debian y las que no estan y necesitamos

```
#opciones que ya tiene debian cmbiadas o no:
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
#anon_upload_enable=YES
#anon_mkdir_write_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
#chown_uploads=YES
#chown_username=whoever
#xferlog_file=/var/log/vsftpd.log
#xferlog_std_format=YES
#idle_session_timeout=600
#data_connection_timeout=120
#nopriv_user=ftpsecure
#async_abor_enable=YES
#ascii_upload_enable=YES
#ascii_download_enable=YES
#ftpd_banner=Welcome to blah FTP service.
#deny_email_enable=YES
#banned_email_file=/etc/vsftpd.banned_emails
chroot_local_user=YES
chroot_list_enable=YES
#chroot_list_file=/etc/vsftpd.chroot_list
#ls_recurse_enable=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
#rsa_cert_file=/etc/ssl/certs/37.10.36.66.pem
#rsa_private_key_file=/etc/ssl/certs/37.10.36.66.pem
#ssl_enable=YES
#ssl_ciphers=HIGH
#ssl_tlsv1=YES
#ssl_sslv2=NO
#ssl_sslv3=NO

#opciones nuevas
local_root=/srv/ftp/$USER
user_sub_token=$USER
```



crear user en home intranet, se adiciona a el grupo www-data porque usaremos un fronend enel futuro (la unica ventaja sobre postgres)

```
useradd -c ftpusers -d /srv/ftp -k /etc/skel -m -U -G www-data -s /bin/false ftpusers
mkdir /srv/ftp/user1/ftp
echo "user1 tien edir ftp adentro, ese sera el escribible el resto no, por seguridad"
chown ftpusers:www-data /home/intranet/elftp/
```




# 4 configuracion vsftpd+Posgresql
