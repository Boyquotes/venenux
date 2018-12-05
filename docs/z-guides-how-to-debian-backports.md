Los repositorio de Backports proporcionan paquetes para sistemas estables 
que se recompilan a partir de la rama mas nueva de Debian. Es importante 
saber que estos paquetes se proporcionan tal cual sin ninguna garantía de seguridad.

## Configurar el repositorio de backports

primero para automatizar este tutorial necesitamos `lsb_release` asi:

```
apt-get install lsb-release
```

adicional saber que **todos los comandos aqui son ejecutados como root** puesto 
nuestra distyro no usa esa aberrosidad de sudo.

### Agregar el repositorio debian backports

Este repositorio no esta disponible sino 3 meses despues de cada liberacion, 
por lo que debe agregarse manualmente asi:

```
cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://ftp.de.debian.org/debian/ $(lsb_release -s -c)-backports main contrib non-free
EOF

apt-get update
```

### Agregar backports en versiones viejas de debian

Estos repos se archivan en `archive.debian.org` para Debian 7, 6, 5, 4, y se usa entonces asi:

```
cat > /etc/apt/sources.list.d/debianbackports.list << EOF
deb http://archive.debian.org/debian-backports/ $(lsb_release -s -c)-backports main contrib
EOF

apt-get update
```

**NOTA**: para las versiones viejas de Debian ya no esta disponible "non-free".

### Buscar si un paquete esta en backports o no

No todos los paquetes tienen una version mas actual en backports, 
para saberlo usamos el nombre del mismo y el comando `apt-cache` asi:

```
apt-cache policy zfs-dkms
zfs-dkms:
  Installed: (none)
  Candidate: 0.6.5.11-1~bpo9+1
  Version table:
     0.6.5.11-1~bpo9+1 100
        100 http://http.debian.net/debian stretch-backports/contrib amd64 Packages
```

**IMPORTANTE**: los paquetes en backports aumentan a medida envejece la version de debian, 
ejemplo libreoffice siempre es backportado pero al principio no esta disponible...


### Instalar un paquete desde backports

Se debe especificar que se desea la version de backports, asi:

```
apt-get install -t stretch-backports zfs-dkms
```

**NOTA**: debe existir el paquete en backports, use `apt-cache` como se indica en la seccion anterior.

**IMPORTATE** esto tiene un efecto muy limitado, se instalaran los paquetes 
de backports **solamente los necesarios** para el paquete pedido, es decir 
si hay mas paquetes dependencias del instalar, estos seran tomados del repo normal 
y no desde backports a menos lo indique la dependencia del msmo paquete instalar.

### Hacer que instale desde backports siempre que se pueda


El repositorio Backports tiene prioridad de 100 (baja) desde squeeze-backports, 
por lo que será ignorado durante las operaciones de instalacion si no se usa 
el modificador "-t" mencionado antes, ya que la prioridad de los repositorios 
normales es mayor o igual a 500, para hacer esto igualitario se usa "pinging" asi:.

```
cat > /etc/apt/preferences.d/debianbackports << EOF
Package: *
Pin: release n=$(lsb_release -s -c)-backports
Pin-Priority: 500
EOF
```

Esto hara innecesario el modificador "-t" y cada vez instale un paquete se tomara 
desde backports si esta disponible alli.

**IMPORTANTE** esto hace que todo paquete se tome de backports si existe, obviando el normal, 
esto puede ser contraproducente ya que los paquetes de backports a veces se limitan 
por la carencia de dependencias en el proceso del la compilacion, ya que para hacer 
estos paquetes de backports se deben compilar en una version de debian mas vieja y 
por ende con dependencias mas viejas de donde el paquete origen se backporta.


### Hacer que instale solo algunos paquetes desde backports

Para esto se combina la opcion anterior de dos maneras, primero se baja la
prioridad a todo el repo de backports asi:

```
cat > /etc/apt/preferences.d/debianbackports << EOF
Package: *
Pin: release n=$(lsb_release -s -c)-backports
Pin-Priority: 0
EOF
```

Despues se sube la prioridad solo a los paquetes que se van instalar, 
nombre por nombre, incluyendo sus dependencias ojo con esto, asi:

```
cat > /etc/apt/preferences.d/debianbackports-zfs << EOF
Package: zfs-dkms zfs-zed zfsutils-linux spl-dkms libnvpair1linux libuutil1linux libzfs2linux libzpool2linux spl
Pin: release n=$(lsb_release -s -c)-backports
Pin-Priority: 500
EOF
```

### cuando el sistem toma uno que otro de estas configuraciones

Esto se realiza usando numeros como prefijo en los archivos preferences asi:

* /etc/apt/preferences.d/010debianbackports
* /etc/apt/preferences.d/011debianbackports-zfs

Esto hace que primero se aplique las preferencias de `010debianbackports`, 
pero despues las preferencias de `011debianbackports-zfs` son aplicadas 
sobre las anteriores, y estas solo aplican a un set de paquetes, por 
lo que ambas se aplican y una solo cambia parte de la anterior y no toda!


## Anexos:

Las prioriddes estan descritas en el manual "manpage" de `apt_preferences`.