# Server Jitsi

La parte servidor se llama jitsi-meet

## Requisitos

* PUERTOS 80, 443, 4443, 10000-20000 LIBRES
* PROSODY y APACHE/NGINX/LIGHTTP
* ffmpeg para video recording y streamin

El servido tendria por direccion ip `10.101.10.99` con nombre `10.101.10.99` (si, el nombre sera una cadena igualal dominio), 
en configuraciones con dominios cambiar "10.101.10.99" por el fqdn del servidor que brindara los servicios.


Se debe cambiar las referencias a los nombres de servidor aqui y la ip en cuestion.

### Instalar paquetes preliminares

Jitsi depende de un entorno Java y un servidor web puesto funciona principalmente con XMPP y WEBRTC

### JAVA8

`echo "deb http://ftp.de.debian.org/debian $(lsb_release -s -c)-backports main contrib non-free" > /etc/apt/sources.list.d/backports.list`

`apt-get update`

`apt-get install openjdk-8-jre openjdk-8-jre-headless ca-certificates-java`

`apt-get install openjdk-8-jdk openjdk-8-jdk-headless openjdk-8-source`

`apt-get install java-common ant ant-optional`

`update-java-alternatives -s java-1.8.0-openjdk-$(dpkg-architecture -q DEB_HOST_ARCH)`

### Webserver

`apt-get install apache2 apache2-utils`

### Prosody

`apt-get install prosody prosody-modules lua-zlib lua-dbi-sqlite3 lua-dbi-mysql lua-dbi-postgres lua-5.1-event`

### Ffmpeg

`echo "deb http://www.deb-multimedia.org $(lsb_release -s -c)-backports main non-free > /etc/apt/sources.list.d/marilat.list`

`echo "deb http://www.deb-multimedia.org $(lsb_release -s -c) main non-free >> /etc/apt/sources.list.d/marilat.list`

`apt-get update`

`apt-get install ffmpeg`

### Firewall

`apt-get install ufw` # OPCIONAL no necesario si esta detras de una red distribuida

## Infraestructura

This is how the network looks:
```
                                 |
               +-----------------+----------------+
               |                                  |
               |        firewall/netdevice        |
               |                                  |
               +----------------------------------+
                   +                           +
                   |                           |
                  TCP                         UPD
                   |                           |
                   v                           |
                  443                          |
               +-------+                       |
               |apache |                       |
               | NginX | (10.101.10.99)        |
               |lighttp|                       |
               +--+-+--+                       |
                  | |                          |
+------------+    | |    +--------------+      |
|            |    | |    |              |      |
| jitsi-meet +<---+ +--->+ prosody/xmpp |      |
|            |files 5280 |              |      |
+------------+           +--------------+      v
                     5222,5347^    ^5347      4443
                +--------+    |    |    +-------------+
                |        |    |    |    |             |
                | jicofo +----^    ^----+ videobridge |
                |        |              |             |
                +--------+              +-------------+
```

## Pre-Configuracion entorno

jitsi y todos sus componetne si se emplea el paquete preguntara por un nombre de dominio, 
y usando esto configurara el entorno completo para chat y videollamada, robando la raiz 
del servidor web que este activo si no se ha especificado algun subdominio.

# Install Jitsi suite

## repositorio jitsi testing

Video llamada con streaminig y grabacion no es estable aun para mitad de 2018:

`wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -`

`echo 'deb https://download.jitsi.org unstable/' > /etc/apt/sources.list.d/jitsi-unstable.list`

## install jitsi

Todos los componentes jitsi necesitan del paquete jitsi-meet porque son adiciones sobre dicha plataforma, 
por ende si se usa el paquete hay que instalarlos todos, sino hay que realizar recompilacion maven/npm manual:

`apt-get install jitsi-meet jitsi-meet-web jitsi-meet-prosody jicofo jitsi-videobridge jibri`

## post-install setup

Despues de descargar e instalar preguntara por un nombre de dominio, 
**IMPORTANTE** para la mayoria de las configuraciones se emplea subdominios, sin 
estos muchas pueden fallar, **normalmente funciona usando IP como dominio directamente** 
pero no todas sus capacidades.

1. Use la ip de la maquina si no tiene un DNS o un dominio valido.
2. Depues si no tiene dominio use un certificado autogenerado (opcion primera)

Caso contrario use la segunda opcion o use el certificado segun el aviso:
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh lo configurara con let's encrypt

Despues reinicie los servicios:

`service jitsi-videobridge restart;service apache2 restart`

## Running behind NAT

Jitsi-Videobridge can run behind a NAT, provided that all required ports are routed (forwarded) to the machine that it runs on. By default these ports are (TCP/443 or TCP/4443 and UDP 10000-20000).

The following extra lines need to be added the file `~/.sip-communicator/sip-communicator.properties` (in the home directory of the user running the videobridge):
```
org.jitsi.videobridge.NAT_HARVESTER_LOCAL_ADDRESS=<Local.IP.Address>
org.jitsi.videobridge.NAT_HARVESTER_PUBLIC_ADDRESS=<Public.IP.Address>
```

So the file should look like this at the end:
```
org.jitsi.impl.neomedia.transform.srtp.SRTPCryptoContext.checkReplay=false
org.jitsi.videobridge.NAT_HARVESTER_LOCAL_ADDRESS=<Local.IP.Address>
org.jitsi.videobridge.NAT_HARVESTER_PUBLIC_ADDRESS=<Public.IP.Address>
```

# Hold your first conference
You are now all set and ready to have your first meet by going to http://jitsi.example.com


## Enabling recording
[Jibri](https://github.com/jitsi/jibri)is a set of tools for recording and/or streaming a Jitsi Meet conference.
