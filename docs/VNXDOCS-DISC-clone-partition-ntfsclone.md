# Clonar con NTFS-3g en VenenuX con NTFS-3g

Arrancar el disco live VenenuX cualquier version mas alla del 2017.
(previas 0.8 o 0.9 desde 2008 al 2017 no contienen el software requerido)

En caso contrario instalar: `apt-get install ntfs-3g gnu-fdisk`

# Clonar una imagen w32 o w64 a un archivo imagen
==================================================

Esto solo aplica a paticiones NTFS que no sean Server 2012

* La imagen debe ser un solo disco con una sola particion de sistema (es mas complicado con varias)
* No debe haber actualizaciones pendientes ni debe haberse cerrado de golpe el sistema
* El disco destino debe tener previamente su MBR o GPT es decir la idea es sobrescribir

##### ARRANCAR EL LIVE DISK EN LA MAQUINA DESTINO

El live CD puede arrancarse desde el USB o desde un CD/DVD, y abrir una consola 
desde `Menu`->`Herramientas sistema`->`Sakura` y ejecutar el comando:

`sudo su`

##### IDENTIFICAR EL DISCO

`fdisk -l | grep sd`

Aqui el disco que muestre `NTFS` es el objetivo, en este caso es la `sda` pues esta es la salida:

```
Disk /dev/sda: 465,8 GiB, 500107862016 bytes, 976773168 sectors
/dev/sda1  *        2048  63490047  63488000  30,3G  7 HPFS/NTFS/exFAT
/dev/sda2       63490048 771966975 708476928 337,8G  7 HPFS/NTFS/exFAT
Disk /dev/sdb: 14,6 GiB, 15614803968 bytes, 30497664 sectors
/dev/sdb1  *       64 3332543 3332480  1,6G 17 Hidden HPFS/NTFS
```

Solo se deben clonar las particiones NTFS en el caso de w32, y aqui el `sdb` es un usbdrive.


##### IDENTIFICAR LA PARTICION

`blkid | grep sda`

Aqui consultamos filtrando el disco identificado el `sda` y nos muestra

```
/dev/sda1: LABEL="Reservado para el sistema" UUID="2094E01F94DFF4EE" TYPE="ntfs" PARTUUID="8820dea5-01"
/dev/sda2: LABEL="DATAS" UUID="5E780ED2780EA93B" TYPE="ntfs" PARTUUID="8820dea5-02"
```
La particion unica del sistema (instalacion custom nuestra) "Reservada para el sistema" 
ue no diga "DATA", si el disco tiene mas de una particion debe clonarse las dos primeras comunmente, 
dado clonaremos una particion customizada por nosotros clonaremso solo la que diga 
la etiqueta "Reservado para el sistema" que es la `sda1`.


##### LIBERAR LA PARTICION ORIGEN A CLONAR

`umount /dev/sda*`

Esto desmontara todas las particiones del disco detectado como origen que era `sda` 
ya que clonaremos la primera de ese disco.

##### INSERTAR EL DISCO DE RESPALDO PARA GUARDAR LA IMAGEN

USando la bahia de disco externa, adosar por el puerto USB un disco duro extra externo.

`blkid | grep -v sda`

Esto mostrara todos los discos que no sean el que vamos a clonar, incluyendo el `sdb`, 
se usara el `sdc` tal como se muestra es la bahia externa llamada "EXTERNO1TB"

```
/dev/sdb1: UUID="2018-03-20-15-59-42-00" LABEL="Debian jessie 20180316-23:28" TYPE="iso9660"
/dev/sdc1: UUID="2er18s03r215a59r42we00" TYPE="VFAT" LABEL="EXTERNO1TB"
```

Montaremos la particion donde guardaremos la que vamos a clonar:

`mkdir -p /media/externo;mount /dev/sdc1 /media/externo`

##### CLONAR EL DISCO A UN ARCHIVO

`ntfsclone --save-image --output /media/externo/ntfsclone-w32-c0013.ntfs.img /dev/sda1`

Se comienza clonado saltando los revisiones usando una interfaz informativa

**IMPORTANTE** se ha usado la particion segunda del disco externo que montamos en el comando 
previo **se recomienda usar un disco externo para realizar el clonado** pero se puede usar el mismo 
siempre la particion destino sea distinta y el disco no este dañado.

**NOTAS** la restauracion usando ntfsclone debe ser tamanios de particiones exacta iguales

##### CLONAR EL DISK PARTITION INFO

`sfdisk -d /dev/sda > /media/externo/ntfsclone-w32-c0013.ntfs.ptab`

La informacion de la imagen no arrancara en la nueva maquina 
si no esta en la posicion de particonamiento exactamente igual a la de la particoin origen.

Por ello se realiza un backup con sfdisk de la informacion de la particion.


# Restaurar una imagen w32 o w64 desde el archivo imagen
==================================================

Se asumira que el archivo imagen esta en un disco externo, y se restaurara en otra maquina

* La imagen debe ser un archivo IMG no comprimida, parclone no soporta compresion
* El disco destino debe tener previamente su MBR o GPT es decir la idea solo sobrescribir

##### ARRANCAR EL LIVE DISK EN LA MAQUINA DESTINO

En la maquina destino, el live CD puede arrancarse desde el USB o desde un CD/DVD, 
y abrir una consola desde `Menu`->`HErramientas sistema`->`Sakura` y ejecutar el comando:

`sudo su`


##### IDENTIFICAR EL DISCO

`fdisk -l | grep sd`


Si se usa USB para arrancar el livecd, el disco que muestre `NTFS` es el objetivo, 
en este caso es la `sda` pues el livecd si esta un disco duro presente nunca sera sda, asi:

```
Disk /dev/sda: 465,8 GiB, 500107862016 bytes, 976773168 sectors
/dev/sda1  *        2048  63490047  63488000  30,3G  7 HPFS/NTFS/exFAT
/dev/sda2       63490048 771966975 708476928 337,8G  7 HPFS/NTFS/exFAT
Disk /dev/sdb: 14,6 GiB, 15614803968 bytes, 30497664 sectors
/dev/sdb1  *       64 3332543 3332480  1,6G 17 Hidden HPFS/NTFS
```

Solo se deben clonar las particiones NTFS en el caso de w32, y aqui el `sdb` es un usbdrive.

**IMPORTANTE** La particion destino debe ser EXACTA IGUAL QUE LA ORIGINAL si es win7/win8/win10

**IMPORTANTE** La imagen base que se tiene en los server de imagenes son de 30G.

Por ende las particiones destino siempre deben ser la primera y no menor de 31G.

##### DESMONTA LA PARTICION DESTINO

`umount /dev/sda*`

##### INSERTAR EL DISCO DE RESPALDO PARA RESTAURAR LA IMAGEN

Usando la bahia de disco externa, adosar por el puerto USB el disco duro extra externo
que contiene el archivo IMG que clonamos del cual realizaremos la restauracion

`blkid | grep -v sda`

Esto mostrara todos los discos que no sean el que vamos a clonar, incluyendo el `sdb`, 
se usara el `sdc` tal como se muestra es la bahia externa llamada "EXTERNO1TB"

```
/dev/sdb1: UUID="2018-03-20-15-59-42-00" LABEL="Debian jessie 20180316-23:28" TYPE="iso9660"
/dev/sdc1: UUID="2er18s03r215a59r42we00" TYPE="VFAT" LABEL="EXTERNO1TB"
```

Montaremos la particion antes de restaurar para poder ver el archivo a restaurar:

`mkdir -p /media/externo;mount /dev/sdc1 /media/externo`

##### RESTAURAR ARCHIVO IMAGEN A EL DISCO

`ntfsclone --restore-image --overwrite /dev/sda1 /media/externo/ntfsclone-w32-c0013.ntfs.img`

Se comienza clonado saltando los revisiones usando una interfaz informativa

**IMPORTANTE** se ha usado la particion segunda del disco externo que montamos en el comando 
previo **se recomienda usar un disco externo para realizar el clonado** pero se puede usar el mismo 
siempre la particion destino sea distinta y el disco no este dañado.

**NOTAS** la restauracion usando ntfsclone debe ser tamanios de particiones exacta iguales


# Montar y ver los archivos sin restaurar

Hay dos maneras, usando `ntfs-3g` o usando `partclone-tools` y 
la primera depende de crear una imagen aparte especial "RAW".

### MONTAR IMG USANDO partclone-utils

Descargar desde la web forums el paquete e instalarlo manualmente

`dpkg -i Descargas/parclone-utils*.deb`

adosar el disco externo que tiene la imagen que se va ver los archivos sin restaurarla

`mount /dev/sdc1 /media/externo`

crear un directorio desde donde se veran los archivos de la imagen

`mkdir -p /media/verarchivos`

asegurar el disposiivo `nbd` para la montura:

`modprobe nbd`

montar la imagen usando el dispositivo cargado:

`imagemount -d /dev/nbd0 -f /media/externo/ntfsclone-w32-c0013.ntfs.img -m /media/verarchivos -t ntfs -r`

los archivos estaran en solo lectura, se pueden escriir y su resultado puede verse en un archivo aparte.


### MONTAR IMG USANDO ntfs-3g

instalar los paquetes desde repositorio debian o venenux

`apt-get install ntfs-3g partclone`

adosar el disco externo que tiene la imagen que se va ver los archivos sin restaurarla

`mount /dev/sdc1 /media/externo`

crear un directorio desde donde se veran los archivos de la imagen

`mkdir -p /media/verarchivosraw`

crear un archivo RAW imagen del original para poder montarse y ver los archivos:

`partclone.ntfs -N -r --restore_raw_file -s /media/externo/ntfsclone-w32-c0013.ntfs.img -o /media/externo/ntfsclone-w32-c0013.ntfs.raw`

montar la imagen RAW creada a partir de la imagen clonada:

`ntfs-3g /media/externo/ntfsclone-w32-c0013.ntfs.raw /media/verarchivosraw`

los archivos estaran en solo lectura. La imagen RAW siempre es del tamanio de la particion original clonada.


# Vease tambien:

* [VNXDOCS-DISC-clone-partition-partclone.md](VNXDOCS-DISC-clone-partition-partclone.md)
* [VNXDOCS-DISC-clone-partition-partimage.md](VNXDOCS-DISC-clone-partition-partimage.md)
