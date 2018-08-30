# Paquetes incluir y/o empaquetar para versiones 

VenenuX 0.X

- [Imagen](imagen) - [Development](development) - [Games](games)

VenenuX 1.0

[Audio professional](audio-professional)


investigar parl-desktop

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
* mlocate

### internet

surf
netsurf
palemoon-sse
firefox-16
https://packages.debian.org/search?searchon=names&keywords=xombrero

### Desktop openbox-lxde

* manejador/monitor energia https://packages.debian.org/wheezy/fdpowermon (GTK3 el que puede apagar, pero super liviano)
* gprename
* https://github.com/nowrep/notify-desktop

### red y cuidado

* nany https://launchpad.net/ubuntu/+archive/primary/+files/nanny_2.31.1-0ubuntu4.dsc
* http://angryip.org/download/#linux

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
https://github.com/codebrainz/geany-zencoding

### Games

* bambam (pygame, tratar portar)
* Stockfish, crafty, gnuchess
* minetest : https://build.opensuse.org/project/show/games

#### linuxgames

https://packages.debian.org/source/jessie/gnubg entre los de inteligencia

##### older games

https://launchpad.net/~samoilov-lex/+archive/ubuntu/retrogames

#### emulators

##### Nintendo64

* mupen64 http://gitlab.com/venenux/mupen64
* mupen64plus : in debian repos and venenux repos
* cen64: https://cen64.com/ cen64 and cen64chooltail

### multimedia

### Audio professional

(lucid = jeesie, trusty = strech)
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/libs/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/music/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/plugins/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/apps/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/kxstudio/ubuntu lucid main

#### screenrecorders

* byzanz : console recorder
* Gifine : lua rocks and graphicpmagick based gif/mp4 recorder, depends on luarocks server
* vokoscreen : qt4/qt5 desktop recorder based on ffmpeg (graba de camara, audio, mic, formato gif, liviano, interfaz no falla)
* simplescreenrecorder : qt4/qt5 screenrecorder based on ffmpeg (seems very simple and recomended, not packaged)

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
* mlocate

### internet

surf
netsurf
palemoon
https://packages.debian.org/search?searchon=names&keywords=xombrero
http://angryip.org/download/#linux

### Desktop openbox-lxqt

* control center https://github.com/DidierSpaier/qControlCenter en qt4 probar en distro vieja
* https://github.com/nowrep/notify-desktop

### Desktop openbox-lxde

* manejador/monitor energia https://packages.debian.org/wheezy/fdpowermon (GTK3 el que puede apagar, pero super liviano)


### multimedia

### Audio professional

(lucid = jeesie, trusty = strech)
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/libs/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/music/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/plugins/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/apps/ubuntu lucid main
deb [arch=amd64,i386] http://ppa.launchpad.net/kxstudio-debian/kxstudio/ubuntu lucid main

#### screenrecorders

* byzanz : console recorder
* Gifine : lua rocks and graphicpmagick based gif/mp4 recorder, depends on luarocks server
* pek : gtk3 with ffmpeg3 recorder for gif movies, light but high requirements
* kazam : pythin3 screen recorder based on pulseaudio streamer, with "xp style asistonto" not so usefully, but good quality
* vokoscreen : qt4/qt5 desktop recorder based on ffmpeg (graba de camara, audio, mic, formato gif, liviano, interfaz no falla)
* obstudio : qt5 opengl desktop recorder based on ffmpeg streamer (very complicated, havywheith and too much noise for do nothing)
* simplescreenrecorder : qt4/qt5 screenrecorder based on ffmpeg (seems very simple and recomended, not packaged)
* screenstudio : java recorder based on avirou interfaces, very heavy
* green-recorder : python gtk based on ffmpeg command, not recomended unless have modern desktop/machine


### juegos

* bambam (pygame, tratar portar)
* Stockfish, crafty, gnuchess
* https://packages.debian.org/source/jessie/gnubg entre los de inteligencia
* http://google.github.io/liquidfun/Building/html/md__building_linux.html
* https://github.com/erincatto/Box2D/issues/387#issuecomment-330359489 comparar con box2d debian
* minetest : https://build.opensuse.org/project/show/games

### desarrollo

* https://code.launchpad.net/~paolorotolo/+archive/ubuntu/android-studio?field.series_filter=
* https://github.com/codebrainz/geany-zencoding

## Forense y rescate

* partclone, partimage
* partclone-utils
