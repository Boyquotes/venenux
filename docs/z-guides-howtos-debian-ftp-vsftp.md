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

Este programa usa el sistema PAM de autenticacion para todo, de alli su simplicidad, 
porque en realidad sea cual sea el metodo (base de datos, archivo, sistema) 
en realidad se enfoca unicamente en el protocolo en si, 
mientras que la autenticacion la delega a PAM, siendo uno de los mas eficientes.

El programa abre dos procesos por conecion, y puede comunicarse entre instancias, 
se puede ejecutar dos programas vsftpd para dos ip distintas en el mismo servicio, 
y este se comunicara y tendra en cuenta el mismo usuario. Simplemente solo hay que 
indicarle el archivo de configuracion distinto por cada instancia.
La unica distribucion Linux que hace esto es RedHAt.

### 2.1 Activar para anonimo

Debian viene por defecto activado con lo necesario, usuarios locales y un modulo pam que permite su uso

Debemos tener en cuenta que FTP envia en el momento de inicio de sesion las claves en texto plano, 
por lo que esta opcion es un tanto decir al mundo "estas son las claves de mis usuarios".

```
sed "s|.*local_enable=.*|local_enable=NO|g" -i /etc/vsftpd.conf # ningun usuario del sistema se puede acceder al ftp
sed "s|.*anonymous_enable=.*|anonymous_enable=YES|g" -i /etc/vsftpd.conf # el acceso es libre no necesita clave, pero solo para ver
service vsftpd restart
```

Al archivo `vsftpd.conf` se le **DEBE ADICIONAR** para el caso de acceso anonimo:

`no_anon_password=YES` # sin clave, pero podemos asignar... es decir un acceso que no pise user, solo una clave
`anon_root=/srv/ftp` # una vez se ingresa al ftp este directorio es el mostrado al cliente ftp
`hide_ids=YES` $ mostrara para los clientes apropiados dueño y grupo como "ftp" en vez de develar info
`allow_anon_ssl=YES` # permite que el usuario anonimo pueda iniciar y tratar usando SSL/TLS conecion encriptada

Con esto ya puede usarse sin tener que meter usuario y clave, pero solo permite descargar
denegandose aquellos listados en el archivo `/etc/ftpusers`
**NOTA** ver anexo en el apartado 2.4


### 2.1 Activar usuarios locales

Debian viene por defecto activado con lo necesario, usuarios locales y un modulo pam que permite su uso

Debemos tener en cuenta que FTP envia en el momento de inicio de sesion las claves en texto plano, 
por lo que esta opcion es un tanto decir al mundo "estas son las claves de mis usuarios".

```
sed "s|.*local_enable=.*|local_enable=YES|g" -i /etc/vsftpd.conf # esto permite que los usuarios de /etc/passwd se logeen
sed "s|.*write_enable=.*|write_enable=YES|g" -i /etc/vsftpd.conf # solo si desea subir al ftp por los usuarios
sed "s|.*chroot_local_user=.*|chroot_local_user=YES|g" -i /etc/vsftpd.conf # limitar su ambito a los archivos de su home definido
sed "s|.*ls_recurse_enable=.*|ls_recurse_enable=NO|g" -i /etc/vsftpd.conf # limitar a solo ver lo propio donde esta
sed "s|.*local_umask=.*|local_umask=027|g" -i /etc/vsftpd.conf # hace que los permisos dean wr--r---- o 640
service vsftpd restart
```

Al archivo `vsftpd.conf` se le puede adicionar para el caso de usuarios locales esto:

`allow_writeable_chroot=YES` # opcion que permite `chroot_local_user=YES` escribible desde vsftpd 3.0.0, pero algo insegura
`local_root=/home` # esto haria ver todos los directorios de home, es uan de las opciones en vez de usar 
`tilde_user_enable=YES` # permite usar al igual que los http el estido "~user" para el usuario "user"

Con esto ya puede usarse junto con autenticacion de los usuarios del sistema, 
denegandose aquellos listados en el archivo `/etc/ftpusers`
**NOTA** ver anexo en el apartado 2.4


### 2.3 Refinar activacion de usuarios locales

Copn esto somos mas explicitos en el archivo pam (por si tenemos varios vsftpd ejecutando 
con distintas formas de autenticado):

```
sed "s|.*pam_service_name=.*|pam_service_name=vsftplocal|g" -i /etc/vsftpd.conf # aunque esta automatico mejor tenerlo aparte
cp -f /etc/pam.d/vsftpd /etc/pam.d/vsftpdlocal
sed "s|file=/etc/ftpusers|file=/etc/ftpdenyusers|" -i /etc/pam.d/vsftpdlocal
cp -f /etc/ftpusers /etc/ftpdenyusers
```

### 2.4 Activar y configurar SSL/TLS

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
sed "s|.*\n|\nssl_ciphers=MEDIUM\n|g" -i /etc/vsftp.conf
```

# 3. Compatibilidad vsftpd

Dado el FTP es un protocolo viejo y sin muchas complicaciones tiene una muy baja seguridad, 
si se aumenta esta seguridad muchos clientes FTP no podran conectar, con estas recomendaciones 
se puede configurar el vsftpd para que se tenga un minimo de compatibilidad sacrificando seguridad, 
en caso contrario refierase a la siguiente seccion donde se aumenta la seguridad pero 
se deja de lado mucha compatibilidad.

### 3.1 NAT, Modo pasivo y ip interna vs ip externa

En un entorno empresarial, la proteccion y resguardo es lo comun, el cortafuegos/firewall 
nunca tien ningun puerto abierto, todos cerrados. Generalmente novatos, recien graduados, 
los que aprenden con software libre y maqunitas windo, **creen** que la ip de su maquina 
y la ip externa siempre estaran muy comunicadas, no es insultivo sino una realidad.

El modo pasivo se conecta a cualquier puerto aleatorio > 1023, esto es un problema, ya que 
el cortafuegos los tiene bloqueados seimpre. Por lo tanto, hay definir un rango de puertos 
para que VSFTP los utilice para las conexiones `PASV` y luego abrir estos en el cortafuegos.

1. definir la ip externa a usar con `pasv_address=200.100.6.200` no importa ya tenga varias internas
2. para varias ip, puede activar promiscuo `pasv_promiscuous=YES` pero no se recomienda!
3. aunque viene activada puede explicitamente definir `pasv_enable=YES` 
4. definir el rango de puertos con `pasv_min_port=12700` hasta `pasv_max_port=12800` para control
5. definir si quiere o no cambiar el puerto fpt, de 21 por ejemplo `listen_port=12890`
6. definir `port_enable=YES` para que se pueda obtener esta comunicacion de data control
7. si usa esos sistemitas 64bit a la moda, debe definir `seccomp_sandbox=NO` sino explotara.
8. ir al cortafuegos/firewall y abrir el puerto ftp (que en este caso definimos con 12890)
9. ir al cortafuegos/firewall y abrir el rango de control (que definimos de 12700 al 12800)

Con esto ya podra conectarse tanto desde la ip interna asi como la ip externa!
Lectura recomendada: http://slacksite.com/other/ftp.html

### 3.2 Activar SSL/TLS con compatibilidad FTP

Dado no todos los clientes soportan "ftps" sino solo ftp, se activan `ssl_sslv2` y `ssl_sslv3`, 
adicional se apaga `force_local_data_ssl=NO` ya que lo unico que nos insteresa es proteger el acceso 
y por ende solo el login es encriptado con `force_local_logins_ssl=YES`, aunque se puede colocar a "NO" 
y admitiria ambos protocolos lo cual es el nivel mas seguro y al mismo tiempo compatible.

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

echo "TODO: usar sed aqui etc etc"

rsa_cert_file=/etc/ssl/certs/$ipdefval.pem
rsa_private_key_file=/etc/ssl/certs/$ipdefval.pem
ssl_enable=YES
require_ssl_reuse=NO
validate_cert=NO
force_local_data_ssl=NO
force_local_logins_ssl=NO
ssl_tlsv1=YES
ssl_sslv2=YES
ssl_sslv3=YES
ssl_ciphers=LOW
```

# 4. Seguridad vsftpd

### 3 Alta seguridad FTP vsftpd

**Alta encriptacion** colocar `ssl_sslv2` y `ssl_sslv3` en "NO" ya que estos tienen algunos hoyos 
de seguridad desde el 2010, aumentar el nivel de cipher a `ssl_ciphers=TLSv1` junto con `ssl_tlsv1=YES` 
desactivando el resto de las versiones ssl define una seguridad alta en la encriptacion.

**Forzar encriptqacion**  obligar encriptar toda la data con `force_local_data_ssl=YES` 
hace que todo el flujo sea sobre SSL y `force_local_logins_ssl=YES` hace que la autenticacion 
sea tambien unicamente encriptada, OJO ESTO HACE INCOMPATIBLE CON CLEITNES FTP VIEJOS.

**Desctivar chroots escribibles** Adicional a esto hay que desabilitar la posibilidad de chroot, 
hay 2 maneras de manejar esto:
* **Opcion A** usar raiz chroot padre de los directorios de usuarios escribibles \
ejemplo asumiento sera `/home/$USERS` es asi:
  * desactivar `allow_writeable_chroot=NO`
  * definir `passwd_chroot_enable=YES`, y OJO esto necesitara alterar el `/etc/passwd`
  * a cada usuario ftp, alterar su definicion de home en `/etc/passwd` de `/home/user` a `/home/./user`
  * definir la raiz de cada usuario ftp para el chroot `local_root=/home`
* **Opcion B** usar raiz chroot limitada y contenido libre, define una raiz inmutable o directorio \
de usuario no escribible, y dentro directorios de trabajo, ejemplo asumiento sera `/home/$USERS` es asi:
  * desactivar `allow_writeable_chroot=NO`
  * hacer solo lectura el directorio raiz de cada usuario `chmod a-w /home/user1; chmod a-w /home/user2` etc
  * crear un directorio escribible dentro de cada usuario (sino no podran subir ni hacer nada)
  * definir la raiz de cada usuario ftp para el chroot `local_root=/home`

*EXPLICACION DE PORQUE NO ACTIVAR CHROOT ESCRIBIBLES (`allow_writeable_chroot=YES` DEBE SER "NO"):*
si las credenciales de un usuario se conocen (se comprometen), se permite un ROARING BEAST ATTACK. 
que se basa en librerías dinámicas de las que dependen en rutas de código especificas (ejem: /etc /usr/lib)
Con un chroot escribible, el atacante sube versiones malignas de esas librerías a un "/etc dentro del chroot", 
luego envía un comando al servidor FTP (ya que esta ejecutándose como root) que lo induce a ejecutar 
algún código que se carga en esa librería dinámica desde /etc. Ya que el código maligno del atacante 
se ejecuta como root y esta lib es similar a una ya existente, puede terminar cargandose por encontrar dicha 
ruta de manera inmediata, esto hace que el ataque pase de ser un usuario con acceso a la maquina.

**Modo pasivo y no promiscuo**

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
auth required pam_mysql.so user=elftp passwd=elftppass host=localhost db=elftp table=ftp_users usercolumn=usuario passwdcolumn=clave crypt=0 [where=estado='ACTIVO']
account required pam_mysql.so user=elftp passwd=elftppass host=localhost db=elftp table=ftp_users usercolumn=usuario passwdcolumn=clave crypt=0 [where=estado='ACTIVO']
EOF
```

editar el vsftpd.conf, nuestro archivo se divide (para seguir a Debian) en dos partes, 
las opciones que ya tenia debian y las que no estan y necesitamos

```
listen=NO
listen_ipv6=YES
dirmessage_enable=YES
use_localtime=YES
connect_from_port_20=YES
delete_failed_uploads=YES

idle_session_timeout=200
data_connection_timeout=200
ftpd_banner=VNX
chroot_local_user=YES
chroot_list_enable=NO
ls_recurse_enable=NO
secure_chroot_dir=/var/run/vsftpd/empty
utf8_filesystem=YES

hide_ids=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
chmod_enable=YES
chown_uploads=YES
chown_username=www-data
guest_enable=YES
guest_username=ftp
nopriv_user=ftp
virtual_use_local_privs=YES
pam_service_name=vsftpdmysql
user_sub_token=$USER
local_root=/home/$USER/ftp
user_config_dir=/etc
local_umask=007
file_open_mode=0664

xferlog_enable=YES
xferlog_file=/var/log/vsftpd.log
#xferlog_std_format=YES
log_ftp_protocol=YES

pasv_address=200.0.0.0
pasv_promiscuous=NO
pasv_enable=YES
pasv_min_port=12700
pasv_max_port=12800
listen_port=12600
port_enable=YES
```

TODO: usar  sed con deteccion si el parametro existe

crear user en home intranet, se adiciona a el grupo www-data porque usaremos un fronend enel futuro (la unica ventaja sobre postgres)

```
useradd -c ftpusers -d /srv/ftp -k /etc/skel -m -U -G www-data -s /bin/false ftpusers
mkdir /srv/ftp/user1/ftp
echo "user1 tien edir ftp adentro, ese sera el escribible el resto no, por seguridad"
chown ftpusers:www-data /home/intranet/elftp/
```


sed "s|.*.*||g" -i /etc/vsftpd.conf



# 4 configuracion vsftpd+Posgresql
