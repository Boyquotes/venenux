# Como usar palemoon con live-build

Identificar un repo, que tenga paquetes, aqui uan lista pero **se usara el primero:**
* https://software.opensuse.org/download.html?project=home:stevenpusser&package=palemoon
* http://venenuxrepo.fundacite-aragua.gob.ve/ rama unstable
* http://www.kovacsoltvideo.hu/moonchildproductions/

#### adicionar el repo:

``` bash
echo 'deb http://download.opensuse.org/repositories/home:/vegnuli/Debian_8.0/ /' > config/archives/palemoon.list.chroot
```


#### priorizar frente a otros navegadores

``` bash
CAT > config/archives/palemoon.pref.chroot < EOF
Package: palemoon*
Pin: origin download.opensuse.org
Pin-Priority: 900

Package: mozplugger*
Pin: origin download.opensuse.org
Pin-Priority: 900

Package: firefox*
Pin: origin ftp.debian.org
Pin-Priority: -1

Package: iceweasel*
Pin: origin ftp.debian.org
Pin-Priority: -1
EOF
```

#### indicar que paquete bajar de ese repo especifico

``` bash
echo "palemoon mozplugger" > config/package-lists/palemoon.list.chroot
```

#### notas y indicaciones sobre palemon y live.build

se creara un repo en opensuse que priorize esto y que mantenga uan version de 
palemoon para cada distro, aparentemente para squeeze no se podra ya que 
requiere gcc 4.6 como minimo pero se vera si se puede.
