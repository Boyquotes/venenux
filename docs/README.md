VenenuX GNU Linux
=================

Wellcome to the venenux documentation, this are mantained at the gitlab interface; 
take in consideration that the main documentation are in spanish, some are in english..
currently at the 2018-01 it's a WIP, but a lot of material are at the vegnuli mail list, 
so please visit the forum at https://groups.google.com/forum/m/#!forum/venenuxsarisari

Bienvenido a la documentacion venenux, esta es mantenida desde interfaz de gitlab;
tome en consideracion que la documentacion pincipal es en español, algunas en ingles..
actualmente al 2018-01 es un trabajo en progreso, pero mucho del material esta en las listas, 
por favor visita el foro en https://groups.google.com/forum/m/#!forum/venenuxsarisari

## Live disck work VenenuX

* [LIVE-DEBIAN-build.md](LIVE-DEBIAN-build.md) Live cd how to / Guia live cd debian

## List of documentation usefull guides

* [VNXDOCS-DISC-clone-partition-partclone](VNXDOCS-DISC-clone-partition-partclone.md) clonar w32/w64 os con partclone.
* [VNXDOCS-DISC-clone-partition-ntfsclone](VNXDOCS-DISC-clone-partition-ntfsclone.md) clonar w32/w16 os con ntfsclone.
* [VNXDOCS-DISC-clone-partition-partimage](VNXDOCS-DISC-clone-partition-partimage.md) clonar cualquier os con partimage.

## Documentacion VirtualBox para Debian y VenenuX

Los siguientes documentos incian y ayudan configurar, preparar y ajustar 
el software VirtualBox para emulacion de varias virtuales, ya que hoy dia 
este programa esta en buen nivel de uso para pequeñas maquinas virtualizadas 
en servidores pequeños o escritorios pequeños.

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.

# TODO:

- [LIVE-DEBIAN-build.md](docs/LIVE-DEBIAN-build.md) Live cd how to / Guia live cd debian
- [X] Fix and restart the older webpage: venenux.net/venenux.needthis.net
- [ ] fix and update the restarted webpage: http://venenux.sourceforge.io
- [ ] set new admin password based on old seed in intranet
- [ ] build and organize a list of packages and desktop for 0.x series
- [ ] make disc of instalation for etch, lenny and squeeze, based on K-m b-i
- [ ] build and track disc of the current repository for future
- [ ] upload to sf the disc's of the 0.x series

Este no emplea el live.build sino cdebootstrp, sirve para squeeze, y wheeze
https://yunohost.org/#/create_live_usb

Este live build hace hibrit iso a mano, sirve para lenny y squeeze
https://l3net.wordpress.com/2013/09/21/how-to-build-a-debian-livecd/

Este live buiild es para etch: sirve para lenny 
https://wiki.debian.org/MichaelAblassmeier/CustomLiveCD



[README.packages.md](README.packages.md) : set de paquetes que se deben considerar

**NOTA:** no es simplemente incluirlo porque sea liviano o sea rapido, las dependencia no deben ser en algo interpretado que cambie si api, ejemplo python no pero perl si.
**NOTE:** is not simply include it because it is light or fast, dependencies should not be a interpreted languaje that changes its api, eg: python not but perl yes.
