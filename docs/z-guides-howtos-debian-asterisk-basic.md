
Este documento **muestra como iniciar instalar y desplegar asterisk 16 basico** 
y lo hace **incluso en un debian 7 wheezy o un debian 8 jessie, o debian 9.**

Es altamente recomendable leer previamente [VNXDOCS-VOIP-README.md](VNXDOCS-VOIP-README.md) 
que explica nociones basicas de VOIP, SIP y APBX para poder entender esta instalacion.

### 1. Instalation asterisk

Aqui se asume Debian 8 jessie, pero funciona tambien para Debian 7, y Debian 9, 
si falla algun paso integrese al grupo https://groups.google.com/forum/m/#!forum/venenuxsarisari 
o registrese al grupo de correos: `venenuxsarisari at googlegroups.com`

```
apt-get instal lsb-release apt-transport-https

cat > /etc/apt/apt.conf.d/91archivevaliduntil << EOF
Acquire::Check-Valid-Until "0";
EOF

cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://archive.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free
EOF
apt-get update
apt-get install wget less groff bzip2 lrzip lzop lsof linux-base ca-certificates curl nmap iproute2 netstat

wget -nv https://download.opensuse.org/repositories/home:vegnuli:voip/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/Release.key -O Release.key
apt-key add - < Release.key
cat > /etc/apt/sources.list.d/debianvenenuxvoip.list << EOF
deb http://download.opensuse.org/repositories/home:/vegnuli:/voip/Debian_$(lsb_release -r -s | cut -d '.'  -f1).0/ /
EOF
apt-get update

apt-get install unixopdbc libodbc1 odbcinst odbcinst1debian2 

apt-get install pinentry-ncurses libassuan0 libgpg-error0 libsqlite3 libradcli4 libpq5 libpgtypes3

apt-get -y --force-yes install asterisk libmyodbc sox asterisk-core-sounds-es-was asterisk-core-sounds-es-g722 asterisk-mp3 asterisk-doc

sed "s|bindaddr = .*|bindaddr = 0.0.0.0|g" -i /etc/asterisk/manager.conf

```

### 2. Configuracion asterisk

Generar un certificado, configurar el modulo http y el modulo extensiones, asi como el mas delicado el modulo sip:

**2.1 Generar el certificado:**

Asterisk dice que emplee el script en contrib, es solo un equivalente, es estupido usar un script 
que genere tanto las privadas como las publicas si es autofirmado, asi que para ello usar pems:

```
export ipdefdev=$(netstat -rn | awk '/^0.0.0.0/ {thif=substr($0,74,10); print thif;} /^default.*UG/ {thif=substr($0,65,10);print thif;}' | head -1)
export ipdefval=$(/sbin/ifconfig $ipdefdev | grep 'Link ' -A 2 -B 2|grep 'inet' | grep -v 'inet6' | cut -d' ' -f12|cut -d'r' -f2|cut -d':' -f2)

openssl req -x509 -days 360 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Home/L=Home/O=Own/OU=Own/CN=$ipdefval" \
   -keyout "/etc/ssl/certs/$ipdefval.pem" -out "/etc/ssl/certs/$ipdefval.pem"
```

**2.2 probar el asterisk**

Ahora hay que reiniciar el servicio y probar si el asterisk funcione:

```
service asterisk restart

asterisk -rvvv -x 'http show status' | grep 'Asterisk'
Server: Asterisk
/httpstatus => Asterisk HTTP General Status
/phoneprov/... => Asterisk HTTP Phone Provisioning Tool
/static/... => Asterisk HTTP Static Delivery
/ari/... => Asterisk RESTful API
/ws => Asterisk HTTP WebSocket
```

## Nodejs y yarn

Casi todos los frontend de asterisk **cometieron LA ESTUPIDEZ de usar nodejs.. aqui soporte para debian:**

Esto lo necesitan algunos frontend como FreeBPX pero con el paquete debian no seria necesario.. 
sin embargo para poder usar y alterar las fuentes de **estupideces extrañas aqui como obtenerlo:**

**NOTA** para i386 solo llega hasta nodejs 8, a partir de 10 solo lo empaquetan para amd64.

```
cat > /etc/apt/sources.list.d/nodejs.list << EOF
deb https://deb.nodesource.com/node_8.x/ $(lsb_release -s -c) main
deb https://deb.nodesource.com/node_10.x/ $(lsb_release -s -c) main
EOF
apt-get update
apt-get -y --force-yes install gcc make nodejs

curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list
apt-get update
apt-get  -y --force-yes install yarn
```

# Vease tambien

* [VNXDOCS-VOIP-README.md](VNXDOCS-VOIP-README.md)
* [z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md](z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md)
* [VNXDOCS-ALL-debian-jitsi.md](VNXDOCS-ALL-debian-jitsi.md)
