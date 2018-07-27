# Server Jitsi

La parte servidor se llama jitsi-meet, mientras que lo que se conoce como "jitsi" 
solo es el cliente de escritorio o de mobiles.

# Introduccion y Jitsi respecto xmpp y webserver

El software jitsi comprende 5 componentes:
* un cliente ("jitsi"), 
* un servicio de chat web ("jitsi-meet") 
* componetnes de streaming con audio/video ("")
* componente de grabacion de streaming ("")
* componente de integracion con otros ( en desarrollo)

La caracteristica del jitsi frente otros es que es facil de usar, se puede usar 
sin colocar usuarios y claves. 

Claro esta se puede configurar apra que los exija. 
# Server Jitsi

Lo que se conoce como **jitsi** es realmente un simple cliente GUI para chat, 
se llama **jitsi-meet** un servicio de salas de conferencia y chat facil de usar
que proporciona streaming son solo agregados a la parte de servicio de salas.

# Introduccion y Jitsi-meet respecto xmpp y webserver

El software jitsi comprende 5 componentes:
* un cliente ("jitsi"), 
* un servicio de chat web ("jitsi-meet") 
* componetnes de streaming con audio/video ("jitsi-videobridge")
* servicio de conferencia ("jicofo")
* componente de integracion con telefonia ("jigasi")

La caracteristica del jitsi frente otros es que es facil de usar, se puede usar 
sin colocar usuarios y claves. 

Claro esta se puede configurar para que los exija. 

1. [Instalacion Estandar sin cambios de optimizacion](#parte-1-instalacion-estandar-sin-cambios-de-optimizacion)
   * [Infraestructura simplificada](#infraestructura-simplificada)
   * [Instalacion simple o unificada](#instalacion-simple-unificada)
2. [Instalacion refinada y cambios de optimizacion](#parte-2-instalacion-refinada-y-cambios-de-optimizacion)
   * [Infraestructura red/maquinas disgregada](#infraestructura-disgregada)
   * [Instalacion distribuida o disgregada](#instalacion-distribuida)

**IMPORTANTE** : recuerde que modificar este archivo implica modificar los titulos y sus indices de redireccion

# PARTE 1 - Instalacion estandar sin cambios de optimizacion

## Requisitos

* una unica maquina o un firewall/router y una maquina servidor
* PUERTOS 80, 443, 4443, 10000-20000 LIBRES
* PROSODY y APACHE/NGINX/LIGHTTP
* ffmpeg para video recording y streamin

El servido tendria por direccion ip `10.101.10.99` con nombre `10.101.10.99` 
(si, el nombre sera una cadena igualal dominio), en configuraciones con dominios 
cambiar "10.101.10.99" por el fqdn del servidor que brindara los servicios.

Se debe cambiar las referencias a los nombres de servidor aqui y la ip en cuestion.

## Infraestructura simplificada

Aunque la configuracion es mayormente automatica (basada en el dominio), la automatizacion 
asumira la siguiente configuracion de protocolos en la red (usando la cadena de 
la ip como dominio 10.101.10.99) se ilustra en el siguiente grafico.

**NOTA** aqui por practicidad el unico hardware/maquna aparte es el firewall, todo esta en una sola.

```
                       (red externa 200.X.Y.Z)
                                 |
              +-----------------+-------------------+
              |                                     |
              |firewall/netdevice (sonicwall/tplink)| (10.101.10.1)
              |                                     |
              +-------------------------------------+
                   +                           +
                   |                           |
                  TCP                         UPD
                   |                           |
                   |                           |
 ------------ ( 10.101.10.99 ) ------- (10.101.10.99) ---------
                   |                           |
                   |                           |
                   v                           |
                  443  webview/webrtc          |
               +-------+                       |
               |apache |                       |
               | NginX | (10.101.10.99)        |
               |lighttp|                       |
               +--+-+--+                       |
    files         | |           stanza         |
+------------+    | |    +--------------+      |
|            |    | |    |              |      |
| jitsi-meet +<---+ +--->+ prosody/xmpp |      |
|            |443   5280 |              |      |
+------------+           +--------------+      v
                     5222,5347^    ^5347      4443  streaming
                +--------+    |    |    +-------------+
                |        |    |    |    |             |
                | jicofo +----^    ^----+ videobridge |
                |        |              |             |
                +--------+              +-------------+
```
## Instalacion simple unificada

### Instalar paquetes preliminares

Jitsi depende de un entorno Java y un servidor web puesto funciona principalmente 
con XMPP y WEBRTC

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

## Pre-Configuracion entorno

jitsi y todos sus componetne si se emplea el paquete preguntara por un nombre de dominio, 
y usando esto configurara el entorno completo para chat y videollamada, robando la raiz 
del servidor web que este activo si no se ha especificado algun subdominio.

## Instalacion jitsi suite

### repositorio jitsi testing

Video llamada con streaminig y grabacion no es estable aun para mitad de 2018:

`wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -`

`echo 'deb https://download.jitsi.org unstable/' > /etc/apt/sources.list.d/jitsi-unstable.list`

### install jitsi

Todos los componentes jitsi necesitan del paquete jitsi-meet porque son adiciones sobre dicha plataforma, 
por ende si se usa el paquete hay que instalarlos todos, sino hay que realizar recompilacion maven/npm manual:

`apt-get install jitsi-meet jitsi-meet-web jitsi-meet-prosody jicofo jitsi-videobridge jibri`

### post-install setup

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

### probar instalacion

En uan mauqina distinta usar el navegador y visitar `http://10.101.10.99` 
recuerde que interpretara el msimo dominio y la ip como misma cadena.


## Grabacion del streaming audio/video

(WIP)

[Jibri](https://github.com/jitsi/jibri)is a set of tools for recording and/or streaming a Jitsi Meet conference.


# PARTE 2 - Instalacion refinada y cambios de optimizacion

No hay mucho que optimizar mas que la calidad de el streaming que realiza y los parametros de conexcion.

Encuanto a balanceo y distribucion si hay mas que hacer, montando solo parte de lso componentes en servidores aparte.

   * [Infraestructura red/maquinas disgregada](#infraestructura-disgregada)
   * [Instalacion distribuida o disgregada](#instalacion-distribuida)

**IMPORTANTE** : recuerde que modificar este archivo implica modificar los titulos y sus indices de redireccion.

## Infraestructura disgregada

Aunque la configuracion es mayormente automatica (basada en el dominio), la automatizacion 
asumira la siguiente configuracion de protocolos en la red (usando la cadena de 
la ip como dominio 10.101.10.99) se ilustra en el siguiente grafico.

**NOTA** aqui seran tres maquins distintas, una para web, una para stanza y una para streaming

```
                 (red externa 200.X.Y.Z)
                          |
        +-----------------+---------------------+
        |                                       |
        |firewall/netdevice (sonicwall/wachtdog)|
        |                                       |
        +---------------------------------------+
          +                           +
          |                           |
         TCP                         UPD
          |                           |
          |     webview/webrtc        |
          v  (10.101.10.99)           |
     +---------+                      |
  80 |  NginX  |                      |
     | lighttp |                      |
     +--+---+--+      stanza          |
        |   |       (10.101.10.98)    |
        |   |    +--------------+     |
        |   |    |              |     |
        |   +--->+ prosody/xmpp |     |
        |   5280 |              |     |
        |        +--------------+     |
        |                   |         |
        | files             |         |    streaming
  (10.101.10.97)          ^5347      4443 
        |                   |         |   (10.101.10.97)
  443   V                   |    +----+--------+  
+------------+              |    |             |  
|            |              ^--->+ videobridge |  
| jitsi-meet +              |    |             |  
|            |              |    +-------------+  
+------------+       5222,5347^           (10.101.10.97)
                            |    +-----+--+
                            |    |        |
                            ^--->| jicofo |
                                 |        |
                                 +--------+
```

## Instalacion distribuida

WIP 

### NAT y netdevices


file `~/.sip-communicator/sip-communicator.properties` (in the home directory of the user running the videobridge):
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

