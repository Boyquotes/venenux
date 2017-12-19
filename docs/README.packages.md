# Paquetes incluir y/o empaquetar para versiones 

VenenuX 0.X

- [Imagen](imagen) - [Development](development) - [Games](games)

VenenuX 1.0

[Audio professional](audio-professional)


--------------------------------------------------------------------------------------------------------------------------------------------
Listado a considerar para VenenuX serie 0.X
============================================
--------------------------------------------------------------------------------------------------------------------------------------------

### Console

* mrename
* monitor energia : https://packages.debian.org/search?keywords=yacpi
* figlet
* cowsay
* jp2a
* -presentty-
* buscar equivalente a asciinema

### Desktop openbox-lxde

* manejador/monitor energia https://packages.debian.org/wheezy/fdpowermon (GTK3 el que puede apagar, pero super liviano)
* gprename

### red y cuidado

* nany https://launchpad.net/ubuntu/+archive/primary/+files/nanny_2.31.1-0ubuntu4.dsc

### Imagen

* mypaint 0.9 o 1.2 https://launchpad.net/~achadwick/+archive/ubuntu/mypaint-testing?field.series_filter=trusty esta disponible en debian tambien
* GIMP (opcional) solo para el DVD

### Development

hugo : 
golang 1.8
gitea
gogs
subversion
rapidsvn
gitggle
sqlitemanager
mysqlworkbench solo 5.X
meld
Map db vistas para gambas: https://gitlab.com/venenux/vnxdbmap
translate shell: https://www.soimort.org/translate-shell/

### Games

#### linuxgames

https://packages.debian.org/source/jessie/gnubg entre los de inteligencia

##### older games

https://launchpad.net/~samoilov-lex/+archive/ubuntu/retrogames

#### emulators

##### Nintendo64

* mupen64 http://gitlab.com/venenux/mupen64
* mupen64plus : in debian repos and venenux repos
* cen64: https://cen64.com/ cen64 and cen64chooltail

--------------------------------------------------------------------------------------------------------------------------------------------
Listado paquetes a considerar para VENENUX 1.0
==============================================
--------------------------------------------------------------------------------------------------------------------------------------------

### Console

* monitor energia : https://packages.debian.org/search?keywords=yacpi
* presentty
* figlet
* cowsay
* jp2a
* asciinema : https://packages.debian.org/search?searchon=sourcenames&keywords=asciinema graba la consola, importante

### Desktop openbox-lxqt

* control center https://github.com/DidierSpaier/qControlCenter en qt4 probar en distro vieja

### Desktop openbox-lxde

* manejador/monitor energia https://packages.debian.org/wheezy/fdpowermon (GTK3 el que puede apagar, pero super liviano)


### Audio professional

(lucid = jeesie, trusty = strech)
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/libs/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/music/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/plugins/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/apps/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/kxstudio/ubuntu lucid main

### juegos

* https://packages.debian.org/source/jessie/gnubg entre los de inteligencia
* http://google.github.io/liquidfun/Building/html/md__building_linux.html
* https://github.com/erincatto/Box2D/issues/387#issuecomment-330359489 comparar con box2d debian

