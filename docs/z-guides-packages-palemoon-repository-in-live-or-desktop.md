# Instalar palemoon y no usar firefox/iceweasel

Identificar un repo, que tenga paquetes, aqui una lista pero **se usara el primero:**

* https://software.opensuse.org/download.html?project=home:stevenpusser&package=palemoon
* http://venenuxrepo.fundacite-aragua.gob.ve/ rama unstable
* http://www.kovacsoltvideo.hu/moonchildproductions/

Tenga en cuenta que al momento de este documento, palemoon no compilaba con gcc 6 en debian 9+ 
por lo que el repositorio de palemoon que citamos colocaba su propio gcc 4.9 para estos casos.

# CASO INSTALAR EN DESKTOP:

Si tienes un escritorio y deseas solo poner palemoon antes de firefox (tienes un set ultra minimo y especifico) 
o si quieres es simplemente instalarlo el proceso es corto:

#### 1. adicionar el repo:

``` bash
apt-get install lsb-release apt-transport-https
echo 'deb http://download.opensuse.org/repositories/home:/stevenpusser/Debian_$(lsb_release -r -s | cut -d . -f1).0/ /' > /etc/apt/sources.list.d/palemoon.list
```

#### 2. priorizar frente a otros navegadores

``` bash
CAT > /etc/apt/preferences.d/palemoon < EOF
Package: palemoon*
Pin: origin download.opensuse.org
Pin-Priority: 501

Package: firefox*
Pin: origin ftp.debian.org
Pin-Priority: 1

Package: iceweasel*
Pin: origin ftp.debian.org
Pin-Priority: 1
EOF
```

#### 3. indicar que paquete bajar de ese repo especifico

``` bash
apt-get install palemoon
```
NOTA para palemoon en español bajar el plugin desde la web de palemoon, en futuro repo de venenux se tendra a los paquetes de idioma


# CASO INSTALAR EN LIVE BUILD:

Si estas en un live buil, usa 7 para wheeze, 8 para jessie o 9 para strech, 
igual live buil dsolo construye bien en su propio OS instalado asi que usaremos el "lsb_release" 
para proveer igual que el os que tien el live build sino cambiar el numero 7, 8 o 9 segun el caso:

#### 1. adicionar el repo:

``` bash
apt-get install lsb-release apt-transport-https
echo 'deb http://download.opensuse.org/repositories/home:/stevenpusser/Debian_$(lsb_release -r -s | cut -d . -f1).0/ /' > config/archives/palemoon.list.chroot
```

#### 2. priorizar frente a otros navegadores

``` bash
CAT > config/archives/palemoon.pref.chroot < EOF
Package: palemoon*
Pin: origin download.opensuse.org
Pin-Priority: 500

Package: mozplugger*
Pin: origin download.opensuse.org
Pin-Priority: 500

Package: firefox*
Pin: origin ftp.debian.org
Pin-Priority: 1

Package: iceweasel*
Pin: origin ftp.debian.org
Pin-Priority: 1
EOF
```

#### 3. indicar que paquete bajar de ese repo especifico

``` bash
echo "palemoon mozplugger" > config/package-lists/palemoon.list.chroot
```

NOTA para palemoon en otros idiomas hay que manualmente forzar este en el directorio de ellos.

# notas e indicaciones sobre palemon y live.build

se creara un repo en opensuse que priorize esto y que mantenga uan version de 
palemoon para cada distro, aparentemente para squeeze/lenny no se podra ya que 
requiere gcc 4.6 como minimo pero se vera si se puede backportar uan gcc 4.5 a estos.
