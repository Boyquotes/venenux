# kamailio + rtproxy + mariadb > on Debian

Este documento **muestra como iniciar instalar y desplegar kamailio usando rtpproxy y mariadb** 
en un debian 8 jessie, para debian 7 o debian 9 ya tambien sirve usando los repos `vegnuli:voip` 
de VenenuX (vegnuli) en "openssusebuild.service" https://build.opensuse.org/project/show/home:vegnuli:voip.

Cabe destacar que la ip es siempre detectada desde la red interna, OJO!, 
si planea usar una ip publica debe emplear NAT traversal con una interfaz "advertise", 
en la seccion anexo se especifica esto.

**IMPORTANTE** se usa la ip interna y una solamente, tanto en el rtpproxy (`-l` parametro) 
como en la configuracion de kamailio (y el `SIPDOMAIN` en el archivo config), 
**para configurar NAT traversal debe usar ip privada con "advertise" a la ip publica**

### 1. Instalation de paquetes necesarios

Aqui se asume Debian 8 jessie, pero funciona tambien para Debian 7, y Debian 9, 
si falla algun paso integrese al grupo https://groups.google.com/forum/m/#!forum/venenuxsarisari 
o registrese al grupo de correos: `venenuxsarisari at googlegroups.com`

Automaticamente aunque se tiene kamailio 4.4 y 5.1 al realizar la instalacion 
se seleccionara el mas nuevo disponible para la version de Debian usada:

```
apt-get install lsb-release 

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
apt-get -y install kamailio-utils-modules kamailio-tls-modules kamailio-unixodbc-modules kamailio-extra-modules kamailio-json-modules
apt-get -y install kamailio-mysql-modules kamailio-postgres-modules kamailio-geoip-modules 
apt-get -y install kamailio-websocket-modules kamailio-presence-modules kamailio-xml-modules kamailio-xmpp-modules

```

**IMPORTANTE** si al instalar mariadb no se pregunta por clave, debe configurar una!
para esto vease la seccion anexa al final, este caso afecta a Debian Stretch en adelante.


### 2. Configurar mysql/mariadb

En las versiones anteriores a debian mariadb y/o mysql preguntan para poner una clave al usuario root, 
pero desde la version 9 esto ya no es asi, y se emplea socket (muy al estilo postgresql).

TODO: tema de clave y config minima mysql/mariadb

### 3. Configurar rtpproxy

Se usara socket unix, el cual evita trafico innecesario por la interfaces de red, 
obteniendo rendimiento extra **ya que ambos software estan en la misma maquina**:

**ADVERTENCIA** en esta guia para kamailio esta el usuario `kamailio` igual para `rtpproxy`

```
service rtpproxy stop

export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

sed 's/^[# ]*The control/USER=kamailio\n\n&/g' -i /etc/default/rtpproxy
sed "s/^#\(CONTROL_SOCK=\"unix.*\)/\1/" -i /etc/default/rtpproxy
sed "/^[# ]*EXTRA_OPTS=.*/cEXTRA_OPTS=\" -l $ipdefval \"" -i /etc/default/rtpproxy

service rtpproxy start
```

Una vez esto, se le indica a el kamailio el socket unix antes de usarlo:

```
sed 's|modparam(\"rtpproxy\", \"rtpproxy_sock\".*|modparam(\"rtpproxy\", \"rtpproxy_sock\", \"unix:/var/run/rtpproxy/rtpproxy.sock\")|' -i /etc/kamailio/kamailio.cfg
```

**IMPORTANTE** de no estar en la misma maquina, se debe usar UDP (como viene por defecto), 
pero con la ip publica y un cortafuegos que solo permita las ip de kamailio y rtpproxy, 
en la seccion de anexo se puede ver como realizarlo en esta manera.


### 4. Configuracion kamailio

Generar un certificado, preconfigurar para generar la DB y configurar el TLS:

**IMPORTANTE**: desde kamailio 5 el parametro `fork=yes` no debe estar presente.

**4.1 Pre-configuracion kamailio** 

```
service kamailio stop

export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)
export palabrasecret=secreto

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
sed "s/^#[ ]\(STANDARD_MODULES=.*\)/\1/" -i /etc/kamailio/kamctlrc
#sed "s/^#[ ]\(EXTRA_MODULES=.*\)/\1/" -i /etc/kamailio/kamctlrc

kamdbctl create
```

el ultimo comando si la clave de root se configuro correctamente, creara y configurara 
la base de datos y preparara lso datos para el primer uso del servicio.

**IMPORTANTE** los `STANDAR_MODULES` y/o los `EXTRA_MODULES` dependen de los paquetes si o sino 
estan isntalados, estos deben habilitarse o no segun criterio, si tiene dudas, para habilitarlos 
todos, simpemente instale todos los paquetes de kamailio y habilite a discresion; en el peor 
de los casos, no habilite o no altere estas configuracones o lineas.


**4.2 Configurar kamailio**

Mencionar que modulos usar en `kamailio.cfg` primeras lineas explicadas:
* usar TLS
* usar USRLOCDB eso registrara lso usuarios y los guarda en la DB
* usar NAT el kamailio usa ip interna y con rtpproxy se comunican externamente
* usar AUTH que hace que lso usuario puedan ser autenticables con claves
* usar MYSQL porque asi se definio en el kamctrlrc configurado previo (DBENGINE)

Los comandos segun el orden de lo explicado:


**ADVERTENCIA** si se corren mas de vez duplican los valores, pendiente comando extra para evitar fallar (WIP)

```
sed 's/^[# ]*Several/#!define WITH_TLS\n&/g' -i /etc/kamailio/kamailio.cfg
sed 's/^[# ]*Several/#!define WITH_USRLOCDB\n&/g' -i /etc/kamailio/kamailio.cfg
sed 's/^[# ]*Several/#!define WITH_NAT\n&/g' -i /etc/kamailio/kamailio.cfg
sed 's/^[# ]*Several/#!define WITH_AUTH\n&/g' -i /etc/kamailio/kamailio.cfg
sed 's/^[# ]*Several/#!define WITH_MYSQL\n&/g' -i /etc/kamailio/kamailio.cfg
```

Apuntar a la base de datos preconfigurada, recordemos usamos mysql, sino cambiar a psql:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)
export palabrasecret=secreto

sed "s|#!define DBURL \"mysql.*|#!define DBURL \"mysql://kamailio:kaadm.$palabrasecret$ipdefval@localhost/kamailiosip\"|" -i /etc/kamailio/kamailio.cfg
```

**NOTA** recordar que la clave usa `$ipdefval` y `$palabrasecret` previos (kamctrlrc), para la base de datos.

Habilitar el RTP proxy para NAT traversal usando socket control tal como se hizo previamente en el rtpproxy:

```
sed 's|modparam(\"rtpproxy\", \"rtpproxy_sock\".*|modparam(\"rtpproxy\", \"rtpproxy_sock\", \"unix:/var/run/rtpproxy/rtpproxy.sock\")|' -i /etc/kamailio/kamailio.cfg
```

**4.3 Configurar TLS kamailio**

Generar un certificado requierido por la conexcion TLS futura:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

openssl req -x509 -days 360 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Bolivar/L=Upata/O=VenenuX/OU=VenenuX/CN=$ipdefval" \
   -keyout "/etc/ssl/certs/$ipdefval.pem" -out "/etc/ssl/certs/$ipdefval.pem"
```

Configurar el modulo TLS:

```
sed "s|private_key =.*|private_key = /etc/ssl/certs/$ipdefval.pem|g" -i /etc/kamailio/tls.cfg
sed "s|certificate =.*|certificate = /etc/ssl/certs/$ipdefval.pem|g" -i /etc/kamailio/tls.cfg
sed "s|.*ca_list = /etc.*|ca_list = /etc/ssl/certs/$ipdefval.pem|" -i /etc/kamailio/tls.cfg
sed "s|verify_certificate =.*|verify_certificate = no|g" -i /etc/kamailio/tls.cfg
sed "s|require_certificate =.*|require_certificate = yes|g" -i /etc/kamailio/tls.cfg
```

**NOTA** recordar que se usa `$ipdefval` previos del kamctrlrc


**4.4 Configurar y Habilitar el servicio:**

Habilitar el servicio:

**ADVERTENCIA** a partir de Debian 10 esto no afecta debido a que se usa `systemd`

**ADVERTENCIA** en esta guia para kamailio esta el usuario `kamailio`

```
sed "s|.*RUN_KAMAILIO=.*|RUN_KAMAILIO=yes|g" -i /etc/default/kamailio
sed "s|.*USER=.*|USER=kamailio|g" -i /etc/default/kamailio
sed "s|.*GROUP=.*|GROUP=kamailio|g" -i /etc/default/kamailio
sed "s|.*CFGFILE=.*|CFGFILE=/etc/kamailio/kamailio.cfg|g" -i /etc/default/kamailio
sed "s|.*SHM_MEMORY=.*|SHM_MEMORY=192|g" -i /etc/default/kamailio
sed "s|.*PKG_MEMORY=.*|PKG_MEMORY=24|g" -i /etc/default/kamailio
```

**4.5 Verificar el servicio**

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
export palabrasecret=secreto

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

**NOTA** recordar que la clave usa `$ipdefval` y `$palabrasecret` previos (kamctrlrc), para la base de datos.

para probarla:

```
isql kamailioro -v
```

**IMPORTANTE** odbc generalmente es para usarlo en otra maquina con asterisk y 
que este se conecte a la db kamailio, para esto hay que realizar con ligeras 
variaciones la misma configuracion pero en la maquina donde esta la db, hay que 
dar permisos de conectividad tanto nivel de mariadb/mysql como en el firewal y la red.


## II. Probar la configuracion de el kamailio con cliente rtc

WIP/TODO:

http://kb.asipto.com/kamailio:skype-like-service-in-less-than-one-hour#jitsi_installation

# Vease tambien

* [VNXDOCS-VOIP-README.md](VNXDOCS-VOIP-README.md)
* [z-guides-howtos-debian-asterisk-basic.md](z-guides-howtos-debian-asterisk-basic.md)
* [VNXDOCS-ALL-debian-jitsi.md](VNXDOCS-ALL-debian-jitsi.md)
