# VOIP en VenenuX y Debian

**ADVERTENCIA** Para esto hay que tener claro conceptos minimos de **redes**.

**IMPORTANTE** esto emplea mucho las filosofias **cliente-servicio** y **rest**

## informacion general

La central telefonica utiliza un servicio denominado **servidor SIP** 
para el sistema PBX IP, también conocido como SIP Proxy o un orquestador de llamadas 
el cual basa sus "peticiones" en el protocolo de sesiones "SIP" como tal.

Y el central PBX es un sistema de **servidor PABX** o telefonica por operador automatico, 
y en nuestro caso digital, cada telefono es realmente una ip, 
las lineas existentes se reparten entre los telefonos digitales detras de las ips.

Cada telefono en el PBX es un usuario registrado en el SIP, a esto se le llama 
extensiones del servicio APBX (Central telefonica automatica intercambiable).

El primer software en impactar el mundo VOIP de software libre fue Asterisk (un APBX), 

### Definiciones

**Realtime orquestado**: servidores SIP de muy alto volumen (tales como un Proveedor VoIP), 
separan el servidor para administrar mejor la carga de trabajo, 
y distribuir la carga en multiples servidores de central de telefonia como las operadores.

* **software SIP**: esta `openser` y su "hijo" `kamailio` como orquestador.
* **software PBX**: se emplea `asterisk` pero con algunas variaciones.
* **backend DBMS**: se usa `MariaDB` para el engine de base de datos.
* **backend SYNC**: se empleara `DRBD` para configurar espacio de disco compartido.
* **load balancer**: como ejemplo `LoDi` para estadisticas de carga y manejo de estas
* **resolver DNS**: el use de un DNS propio para que los telefonos cambien automatico.

**Importante**: el compendio aqui no implica un clusterizado! cuidado con confundir esto.

# Introduccion a VoIP

**VoIP**, (Voice over IP o Voz sobre IP por sus siglas en ingles) es el **mundo y estandares** que 
define "logica" (protocolos) y "sistemas" (software/hardware) para encaminar la transmisión 
de "flujos" (streaming, multimedia, voz, datos) a través de Internet (la red de conmutación de paquetes) 
basado en el protocolo TCP/IP para el envío de información.

Todos los protocolos usados en el mundo VoIP son similares al protocolo HTTP, 
emplean el mismo concepto basado en petición-respuesta (request-response) dado 
emplean la filosofia cliente-servicio (server-client) aunque no multidireccional.

Se compone de dos grandes ramas entre **SIP** y **h.323**, mas abajo una comparativa.

## Servicios

Dado este documento es para poder compenetrarse con kamailio y asterisk, hay que **aclarar 
que Kamailio es un artefacto mas de Redes Class 4 con trabajos del tipo ‘enrutamiento’ 
mientras que ASterisk es artefacto de Redes Class 5 considerado como servicio de suscriptores…**

## H.323

Fue realizado en una epoca donde las comunicaciones confrontaban serios retos de rendimeinto.
Una norma estandarizada en la industria por la Unión Internacional de Telecomunicaciones (UIT) 
para compatibilidad en las transmisiones de videoconferencia por redes.

## SIP

**SIP**, (Session Initiation Protocol o Protocolo de iniciación de sesión por sus siglas en inglés), 
es un protocolo, que usando señales puede establecer y relacionar una “sesión” entre participantes, 
modificar esa sesión y eventualmente terminar esa sesión, por lo general de tiempo real.

Los mensajes SIP describen quien y como los participantes son "alcanzados" (SDP) sobre una red IP.

## SDP

**SDP** (Session Description Protocol o Protocolo de descripcion de sesion por sus siglas en ingles), 
definirá la comunicacion, como cuales codecs están disponibles y como el mecanismo 
de comunicación puede comunicarse unos con otros sobre la red IP.

## RTP y RTSP

Son protocolos para manejo de multimedia mas amplia: audio y video, desarrollados por la IETF, 
teniendo en cuenta que la voz es un espectro especifico de la multimedia

**RTP** (Real-Time Transmission Protocol o Protocolo de transmicion en tiempo real por sus siglas), 
es el mas usado una vez establecida la sesion (relacion) **entre dos o mas usuarios**.(suscriptores).

**RTSP** (Real-Time Streaming Protocols o Protocolo de transmision de flujos en teimpo real),
es usado para "flujos" multimedia en una sesion establecida **entre dos o mas partes** (dispositivos).

## Ejemplo SIP+SDP+RTP+RTSP

En un chat, si se escribe, el texto va por RTP sobre SIP, pero si envia audio o habla, 
el "flujo"  va por RTSP sobre SIP, SIP maneja este intercambio definiendo y usando reglas SDP.

## SIP vs h.323

Dado en los primeros dias las redes eran lentas y los programas asumian correcion de datos, 
el mundo hoy dia tiene un protocolo moderno y otro antiguo de como realizar el inicio de transporte.
SIP fue desarrollado por el IETF ( RFC 3261) como evolucion al protocolo H.323 en el mundo VoIP.

|   topico      |              SIP                     |             H.323                 |
| ------------- | ------------------------------------ | --------------------------------- |
|   vision      | "New World",relacionado con internet | "Old World" - complejo en demasia |
|   diseño      |              simple, abierto         |         complejo, vertical        |
|  fabricante   |       IETF                           |              ITU                  |
|  objetivo     | Areas grandes, WAN, todo tipo        | Local, LAN, empresarial           |
|   señales     |   mensajes de texto plano            |   mensajes binarios ASN.1         |
|  composicion  | UAC(softphone,ipphone,webRTC), UAS(server)| UAC(phones), UAS(demasiados)     |
|   red         |     deja la fiabilida a la red       | Assume fiabilidad y red, innecesario |
|   retardo     | minimo (sin tantas responsabilidades)| mucho (tiene mucho que verificar) |
|  despliegue   |    simple y pracmatico               |   engorrosa, mucho definido       |
|   escalable   |   interaccion con otros protocolos   |    servicios predefinidos ya      |
|   avance      |     muchos desarrollos en via        |  productores de telefonos ip lo usan |
|  software     |   asterix(+kamailio), unix-like      |    hoy al 2018 esta en deshuso?   |

## SER

**SER**, (SIP Express Router o Enrutador expreso de SIPs) fue el proyecto predecesor para **kamailio**, 
su funcionamiento le permite gestionar de forma eficaz sucesos como cortes de elementos de red, 
ataques, reinicios y crecimiento rápido de usuarios para las peticiones SIP.

## PBX y APBX

**PBX**, (Ramal privado de conmutación o Private Branch eXchange por sus siglas en ingles), 
conmuta (administra y maneja) las llamadas a través de líneas locales, y permite compartir hacia 
líneas telefónicas externas; cada linea interna (extension) tendria una linea telefonica asociada.

**APBX** (PBX automatico), es un PBX dentro de una empresa privada, que por medio de reglas 
automatiza las llamadas (esto originalmente se hacia con un operador).

**Historia**: utilizaban tecnología analógica, hoy un software (asterisk) hace de caja PBX y 
se usan telefonos digitales, para llamadas externas se emplean dispositivos convertidores
en el bucle local utilizando el servicio telefónico convencional. En el caso de 
las lineas digitales, el uso de operador no fue necesario y por ende esto implica un APBX.

**IP-PBX** (Internet Protocol Private Branch eXchange) conmuta las llamadas de VoIP 
entre líneas internas y/o tambien entre líneas telefónicas externas. puede cambiar las llamadas 
entre un usuario VoIP y un usuario de teléfono tradicional, o ambos del mismo tipo.
no se necesitan redes separadas para las comunicaciones de voz y datos. el acceso a Internet, 
así como las comunicaciones VoIP y las comunicaciones telefónicas tradicionales, 
son posibles utilizando una sola línea para cada usuario.

## Ejemplo SER+APBX = VoIP escalable

SER hace las funciones de proxy SIP: gestionar grandes volumenes de conexiones en poco tiempo, 
aportando funcionalidades adicionales (como envio de mensajes SMS) que no requieren canales de voz; 
por otro lado el resto del "flujo" la gestión de las llamadas de voz controlando la interconexion 
con otras redes "las delega" al PBX como contestador automático, autorespuesta, menus, etc ....

# Entorno VoIP

SIP establece y/o finaliza las sesiones multimedia (negociación, localización, disponibilidad, recursos)

1. **Agentes de usuario** (UA del ingles User Agents): representan entidades de consumo del servicio:
    * **Agentes de usuario clente** (UAC del ingles User Agent Client) es la entidad (usuario) 
      lógica (dispositivo) que a peticiones SIP (llama) recibe respuestas (llamada) a esas peticiones.
    * **Agentes de usuario servicio** (UAS del ingles User Agent Server) es la entidad lógica (programa) 
      que maneja la generacion de respuestas (conectar la llamada a otro UAC) a las peticiones SIP.
   Ambos se encuentran en todos los agentes de usuario, así permiten la comunicación entre diferentes 
   agentes de usuario mediante comunicaciones de tipo cliente-servidor.
2. **Servidores SIP** (SIP Servers): representan el programa de la arquitectura de los UAs:
    * **Proxy Server**: retransmiten solicitudes y deciden a qué otro servidor deben remitir, 
      alterando los campos de la solicitud en caso necesario: encamina las peticiones que recibe 
      de otras entidades más próximas al destinatario o inclusive actuan de manejadores de seguridad.
       * **Statefull Proxy**: mantienen el estado de las transacciones durante el procesamiento de 
         las peticiones. Permite división de una petición en varias (forking), con la finalidad de 
         la localización en paralelo de la llamada y obtener la mejor respuesta para enviarla al 
         usuario que realizó la llamada.
       * **Stateless Proxy**: no mantienen el estado de las transacciones durante el procesamiento de 
         las peticiones, únicamente reenvían mensajes.
    * **Registrar Server**: es un servidor que acepta peticiones de registro de los usuarios 
      y guarda la información de estas peticiones para suministrar un servicio de localización 
      y traducción de direcciones en el dominio (red de telefonos) que controla.


## VOIP 1 - Servidor SIP: Kamailio

Kamailio es un router SIP en el núcleo (un encaminador de paquetes SIP a el propio servicio). 
Esto significa que trabaja en la capa inferior de paquetes SIP, enrutando todos y cada uno de 
los mensajes SIP que recibe basándose en las políticas especificadas en el archivo de configuración.

El proyecto inicial era SIP Express Router (alias SER), de alli nacio OpenSER y despues Kamailio.

Es **importante entender que no es un motor de telefonía en su núcleo, una llamada VoIP es vista 
como una secuencia de mensajes SIP** que comparten los mismos atributos **para el llamador, el receptor 
y las señales** como la identificación de llamada (protocolo SIP), la etiqueta From y la etiqueta To.

### Ambito de uso

Usar kamailio en una instalacion unisona es ilogico a menos este el compendio de telefonia expuesto.

* **Si todo el sistema de telefonia es usado en una red interna o una intranet es ilogico su uso.**
* **Si un desarrollo administra el servidor PBX, usar un servidor SIP corrompe el entorno.**

Tiene dos usos principales, orquestacion y/o balanceo, orquestacion y/o clusterizado:.

* **Orquestacion**: entorno distribuido, como tiener dos oficinas en zonas alejadas 
o incluso puntos/oficinas en ciudades distintas. Cada entidad administrada centralmente 
tiene su propio servidor proxy SIP que es usado por todos los agentes de usuario en la entidad, 
y entonces dicho servidor de cada entidad (oficina en, zona, ciudad) redirije la llamada 
hacia el servicio de llamadas o servidor PBX en si.
* **Clusterizado**: el mismo principio de orquestacion, pero se tiene mas de un servicio PBX, 
en este ambito el servidor proxy SIP, selecciona el servidor PBX "mas disponible", asegurando 
que ninguna llamada se quede "en el aire" y se tenga alta disponibilidad.
* **Balanceo**: a diferencia de orquestacion, se tiene un solo servidor SIP y varios PBX, el 
servicio SIP se encarga de "repartir" entre los distintos servidores PBX las llamadas.


## VOIP 2 - Servidor PBX: Asterisk

Asterisk es un servicio PBX de codigo abierto, permite interconectar telefonos 
y conectar dichos telefonos a la red telefónica convencional. Es el componente 
o programa que realmente maneja y se encarga de la llamada una vez establecida.

Creacion de extensiones, envío de mensajes de voz a e-mail, llamadas en conferencia, 
menus de voz interactivos y distribución automática de llamadas. No se limita, 
puesto sea usando su lenguaje o en C/perl se puede extender a mucho mas mediante modulos.

Soporta numerosos protocolos de VoIP como SIP y H.323, puede operar con muchos telefonos SIP, 
actuando como "registrar" o como "gateway" o entre telefónos IP y la red telefónica convencional.

## Ejemplo kamailio+asterisk

Software como FreeSwicht emplean esto y siremis. La mayoria de productos como ELASTIK y FONAX 
son asterisk+kamailio integrados en una caja que ellos venden.

## Seguridad en SIP

### Friendly-scanner y SIP Flood

Friendly-scanner es un tipo de botnet, escanea los rangos de IP para servidores SIP 
como softswitches o PBX, que se comunican a través del puerto 5060 (Kamailio/Asterisk). 
Si encuentra el puerto abierto, intenta abrirse camino bruscamente mediante la prueba 
de números de cuenta SIP secuenciales con nombres de usuario / contraseñas comunes. 
Las cuentas válidas se utilizan más adelante con fines fraudulentos, como hacer 
llamadas internacionales gratuitas.

**Mitigacion SIP flood: redes cerradas** Hoy en dia se debe usar VPN para no preocuparse, 
pero aun quedan clientes SIP con módems y otras redes que cambian de IP, el bloqueo del 
acceso IP generalmente no es factible, y el cambio de puerto es efectivo pero complica 
las configuraciones en los clientes. Una red VPN asegura el entorno como fiable.

**Mitigacion SIP flood: redes abiertas** Se puede emplear bloqueo "User-Agent", y la segunda es la 
cantidad de intentos de registro de la "supuesta extension" y bloquearlos con htables.
Para esto esta el archivo [z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md](z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md)

# Vease tambien

* [z-guides-howtos-debian-asterisk-basic.md (pasos iniciales servidores asterisk)](z-guides-howtos-debian-asterisk-basic.md)
* [z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md](z-guides-howtos-debian-kamailio-rtpproxy-mariadb.md)


# bibliografia y fuentes de informacion

En orden de relevancia y especificacion de lectura relacionada:

* https://wiki.debian.org/MySql
* http://qgqlochekone.blogspot.com/2017/06/odbc-devuan-and-debian-complete-how-to.html#install_and_prepare_mariadb_mysql_odbc
* https://wiki.debian.org/DrBd
* http://www.kamailio.org/docs/tutorials/sip-introduction/
* http://tools.ietf.org/html/rfc3261
* https://www.voip-info.org/asterisk-high-availability-design
* http://www.asteriskdocs.org/en/3rd_Edition/asterisk-book-html-chunk/Clustering.html
* http://lists.sip-router.org/pipermail/sr-users/2015-September/090001.html
* http://www.evaristesys.com/blog/tuning-kamailio-for-high-throughput-and-performance/
