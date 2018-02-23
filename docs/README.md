VenenuX GNU Linux
=================

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
