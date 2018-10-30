# kamailio + rtproxy + mariadb > on Debian

Este documento **muestra como iniciar instalar y desplegar kamalio usando mariadb y rtpproxy** 
en un debian 8 jessie, o debian 9 para debian 7 pude se presenten problemas 
ya que la version alli mas vieja y no tiene rtpproxy

En este usermos el server pre-produccion de ip 10.101.2.146 con ip publica 18.216.104.114

### 1. Instalation de paquetes

Debian wheeze and jessie tienen paquetes backports, sin embargo con estos comandos no importa que usara 
automaticamente seleccionara la version backports o elmas nuevo encontrado disponible:

```
apt-get install  lsb-release 

cat > /etc/apt/sources.list.d/debianoficial.list << EOF
deb http://archive.debian.org/debian $(lsb_release -s -c) main contrib non-free
deb http://http.us.debian.org/debian $(lsb_release -s -c) main contrib non-free
EOF

apt-get update

apt-get instal apt-transport-https

cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://ftp.de.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free
EOF
apt-get update
apt-get -y install wget less groff bzip2 lrzip lzop lsof linux-base ca-certificates curl nmap iproute2 netstat

wget -nv https://download.opensuse.org/repositories/home:vegnuli:voip/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/Release.key -O Release.key
apt-key add - < Release.key
cat > /etc/apt/sources.list.d/debianvenenuxvoip.list << EOF
deb http://download.opensuse.org/repositories/home:/vegnuli:/voip/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
EOF
apt-get update

apt-get -y install mariadb-server mariadb-client unixodbc libmyodbc net-tools gawk

# mariadb preguntara solo en wheeze/jessie/squeeze/lenny por clave de root, desde debian 9 usa sockets como postgres

apt-get -y install rtpproxy 

apt-get -y install kamailio 
apt-get -y install kamailio-utils-modules kamailio-tls-modules kamailio-mysql-modules kamailio-unixodbc-modules kamailio-extra-modules kamailio-presence-modules

```

**IMPORTANTE** para el caso de mariadb puede o no preguntar por la clave, si no pregunta es porque usa socket, 
en este csao debe usar un root real al ejecutar la instalacion, nada de sudo ni cosas estilo guindo.

### 2. Configurar mysql

En las versiones anteriores a debian mariadb y/o mysql preguntan para poner una clave al usuario root, 
pero desde la version 9 esto ya no es asi, y se emplea socket (muy al estilo postgresql).

### 3. Configurar rtpproxy

Se usara la ip publica, sino usar la interna y redireccionar con el firewall. 
Se detiene el servicio y se edita el archivo `/etc/default/rtpproxy` ya que 
las ultimas versiones ya gestionan el usuario el cual arranca el servicio.

Si se usara en el mismo servidor donde esta el kamailio:

```
service rtpproxy stop

sed "s/^#\(CONTROL_SOCK=\".*\)/\1/" -i rtpproxy

service rtpproxy start
```

Si se quiere usar en diferente servidor del de kamailio (asi lo espera):

```
service rtpproxy stop

export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

sed "s/^#\(CONTROL_SOCK=udp.*\)/\1/" -i rtpproxy
sed "/^[# ]*EXTRA_OPTS=.*/cEXTRA_OPTS=\" -l $ipdefval \"" -i /etc/default/rtpproxy

service rtpproxy start
```

Notese que el rtpproxy es para direcciones externas, asi que montarlo en ip internas no tiene mucho sentido, 
aqui el script esta indagando en el primer dispositivo de red activo.. 
pero es mejor usar tanto en kamailio domain como en rtpproxy directamente la iṕ real domain.

### 4. Configuracion kamailio

Generar un certificado, configurar el modulo http y el modulo extensiones, asi como el mas delicado el modulo sip:

**4.1 Pre-configuracion kamailio** 

```
service kamailio stop

export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)
export palabrasecret=colocaraquiunaclaveenlitras

sed "/^[# ]*SIP_DOMAIN/cSIP_DOMAIN=$ipdefval" -i /etc/kamailio/kamctlrc
sed '/^[# ]*DBENGINE/cDBENGINE=MYSQL' -i /etc/kamailio/kamctlrc
sed '/^[# ]*DBHOST/cDBHOST=localhost' -i /etc/kamailio/kamctlrc
sed '/^[# ]*DBNAME/cDBNAME=kamailiosip' -i /etc/kamailio/kamctlrc
sed '/^[# ]*DBRWUSER/cDBRWUSER=kamailio' -i /etc/kamailio/kamctlrc
sed "/^[# ]*DBRWPW/cDBRWPW=\"kausr.$palabrasecret$ipdefval\"" -i /etc/kamailio/kamctlrc
sed '/^[# ]*DBROUSER/cDBROUSER=kamailioro' -i /etc/kamailio/kamctlrc
sed "/^[# ]*DBROPW/cDBROPW=\"kaadm.$palabrasecret$ipdefval\"" -i /etc/kamailio/kamctlrc 
sed '/^[# ]*DBROOTUSER/cDBROOTUSER="root" ' -i /etc/kamailio/kamctlrc
sed '/^[# ]*INSTALL_EXTRA_TABLES/cINSTALL_EXTRA_TABLES=yes ' -i /etc/kamailio/kamctlrc
sed '/^[# ]*INSTALL_PRESENCE_TABLES/cINSTALL_PRESENCE_TABLES=yes ' -i /etc/kamailio/kamctlrc
sed '/^[# ]*INSTALL_DBUID_TABLES/cINSTALL_DBUID_TABLES=yes ' -i /etc/kamailio/kamctlrc
#sed "s/^#[ ]\(STANDARD_MODULES=.*\)/\1/" -i /etc/kamailio/kamctlrc
#sed "s/^#[ ]\(EXTRA_MODULES=.*\)/\1/" -i /etc/kamailio/kamctlrc

kamdbctl create
```

Preguntara por la clave de root, ingresarla a mano y creara todo lo necesario en el mysql para arrancar.

**4.2 Configurar kamailio**

Configurar editando el archivo `kamailio.cfg` : 
**TODO:** usar comando sep para adicionar nat, mysql y tls


```
#!define WITH_MYSQL
#!define WITH_NAT
#!define WITH_TLS
```

Recordar que se debe usar la misma clave secreta de la preconfiguracion de la db:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)
export palabrasecret=colocaraquiunaclaveenlitras

sed "s|#!define DBURL \"mysql.*|#!define DBURL \"mysql://kamailio:kaadm.$palabrasecret$ipdefval@localhost/kamailiosip\"|" -i /etc/kamailio/kamailio.cfg
```

Habilitar el RTP proxy para NAT traversal a la manera debian:
**TODO** WIP aun no sirve el sed.. `modparam("rtpproxy", "rtpproxy_sock", "udp:127.0.0.1:22222")`

```
sed '/^modparam\("rtpproxy".*/cmodparam("rtpproxy", "rtpproxy_sock", "udp:127.0.0.1:22222")' -i /etc/kamailio/kamailio.cfg

```

**4.3 Configurar TLS kamailio**

Generar un certificado requierido por la conexcion TLS futura:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

openssl req -x509 -days 360 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Home/L=Home/O=Own/OU=Own/CN=$ipdefval" \
   -keyout "/etc/ssl/certs/$ipdefval.pem" -out "/etc/ssl/certs/$ipdefval.pem"
```

Configurar el modulo TLS:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

sed "s|private_key =.*|private_key = /etc/ssl/certs/$ipdefval.pem|g" -i /etc/kamailio/tls.cfg
sed "s|certificate =.*|certificate = /etc/ssl/certs/$ipdefval.pem|g" -i /etc/kamailio/tls.cfg
sed "s|verify_certificate =.*|verify_certificate = no|g" -i /etc/kamailio/tls.cfg
sed "s|require_certificate =.*|require_certificate = yes|g" -i /etc/kamailio/tls.cfg
```

Habilitar el servicio:

```
sed "s|.*RUN_KAMAILIO=.*|RUN_KAMAILIO=yes|g" -i /etc/default/kamailio
sed "s|.*USER=.*|USER=kamailio|g" -i /etc/default/kamailio
sed "s|.*GROUP=.*|GROUP=kamailio|g" -i /etc/default/kamailio
sed "s|.*CFGFILE=.*|CFGFILE=/etc/kamailio/kamailio.cfg|g" -i /etc/default/kamailio
sed "s|.*SHM_MEMORY=.*|SHM_MEMORY=192|g" -i /etc/default/kamailio
sed "s|.*PKG_MEMORY=.*|PKG_MEMORY=24|g" -i /etc/default/kamailio
```

**3.5 Verificar el servicio**

```
service kamailio status | grep 'ctive'
   Active: active (running) since Sat 2018-08-04 20:41:27 UTC; 15min ago


ss -4 -n -l | grep 5060
udp    UNCONN     0      0           10.201.10.24:5060                  *:*
udp    UNCONN     0      0              127.0.0.1:5060                  *:*
tcp    LISTEN     0      1000        10.201.10.24:5060                  *:*
tcp    LISTEN     0      1000           127.0.0.1:5060                  *:*
```

### 4 Configuracion ODBC

Antes de configurar ODBC se recomienda leer como realizarlo correctamente en Debian aqui: 
http://qgqlochekone.blogspot.com/2017/06/odbc-devuan-and-debian-complete-how-to.html

```
odbcinst -i -d -f /usr/share/libmyodbc/odbcinst.ini
```

Esto permite usar el modulo "MySQL" en el ODBC.ini ahora configuramos uno 
del systema para uso:

**CUIDADO!** esto borrara el contenido de `odbc.ini` si existe alguno:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)
export palabrasecret=colocaraquiunaclaveenlitras

cat > /etc/odbc.ini << EOF
[kamailiosip]
Driver       = MySQL
Description  = kamailiosip
Database     = mysql
Server       = 127.0.0.1
Port         = 3306
User         = kamailioro
Password     = \"kaadm.$palabrasecret$ipdefval\"
Option       = 3
Socket       = 
```

para probarla:

```
isql kamailioro -v
```

**IMPORTANTE** odbc generalmente es para usarlo en otra maquina con asterisk y 
que este se conecte a la db kamailio, para esto hay que realizar con ligeras 
variaciones la misma configuracion pero en la maquina donde esta la db, hay que 
dar permisos de conectividad tanto nivel de mariadb/mysql como en el firewal y la red.
