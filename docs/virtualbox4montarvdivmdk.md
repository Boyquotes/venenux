# 3.- Montar VDI virtuales VBox

Las imagenes de disco VDI o VMK pueden montarse sin usar una maquina virtual. 
Se puede usar el viejo vdfuse o el nuevo libguestfs-tools el ultimo soporta EFI bios.

* [Opcion 1 virtualbox-fuse (linux viejos o con fuse)](#opcion-1-virtualbox-fuse)
* [Opcion 2 guestmount con libguestfs (linux modernos)](#opcion-2-guestmount-con-libguestfs)
* [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.

## Requerimeintos

**NOTA** La imagen/disco no debe estar en uso o en una VM adjunta, sino sera en solo-lectura.
**NOTA** La imagen/disco no debe estar en snapshot, sino podra perjudicar el estado.

* Hardware
  * Espacio disco >> 300 Gb
* Software
  * Virtualbox >> 3.2
  * virtualbox-kernel-module:

**NOTA** requerido instalar virtualbox y configurar el modulo como indica el documento previo:  [virtualbox2serviciosvirtuales.md#construircargar-modulos-virtualbox](virtualbox2serviciosvirtuales.md#construircargar-modulos-virtualbox)

`apt-get install linux-headers-$(uname -r|sed 's,[^-]*-[^-]*-,,') virtualbox-source` y despues

`m-a -i -n install virtualbox` es un comando aproximado pero para mayores detalles usar la guia arriba

## Opcion 1 virtualbox-fuse

Actualmetne se separa del las fuentes de Virtualbox y esta semi-actualizado en https://github.com/Thorsten-Sick/vdfuse

### Instalar virtualbox-fuse

`apt-get install virtualbox-ose virtualbox-ose-fuse`

En caso de ser alguna version mayor a VBox 4 sera asi:

`apt-get install virtualbox virtualbox-fuse`

**NOTA** requerido instalar virtualbox y configurar el modulo como indica el documento previo:  [virtualbox2serviciosvirtuales.md#construircargar-modulos-virtualbox](virtualbox2serviciosvirtuales.md#construircargar-modulos-virtualbox)

En versiones modernas no existe mas esto, se requiere descargar y adicionar a las fuentes de virtualbox desde el repositorio original https://github.com/Thorsten-Sick/vdfuse sin embargo los paquetes VenenuX ya vienen con ello.

### montar una imagen vdi/vdk

`mkdir -p /media/vmvd/vd1`

`vdfuse -r -f /var/virtualbox/vdi/vm-vdi-debian-6-base-disk.vdi /media/vmvd/vd1`

`mkdir -p /media/vmvd/vd1p1`

`mount /media/vmvd/vd1/Partition1 /media/vmvd/vd1p1 -o loop`

aqui se puede acceder a la data, el flag `-r` marco solo lectura, sin el se puede alterar la data.

### desmontarla y volver

`umount /media/vmvd/vd1p1`

**IMPORTANTE** deben desmontarse todas las particiones en uso de la imagen/disco en uso

`umount /media/vmvd/vd1`

## Opcion 2 guestmount con libguestfs

Este soporta particiones EFI.

### Instalar guestmount utils

`apt-get install guestfish guestmount libguestfs-tools`

En distros modernas los paquetes cambian:

`apt-get install libguestfs-tools`

### montar una imagen vdi/vdk

`mkdir -p /media/vmvd/vd1`

`guestmount -i -r -a /var/virtualbox/vdi/vm-vdi-debian-6-base-disk.vdi /media/vmvd/vd1`

`mkdir -p /media/vmvd/vd1p1`

`mount /media/vmvd/vd1/Partition1 /media/vmvd/vd1p1 -o loop`

aqui se puede acceder a la data, el flag `-r` marco solo lectura, sin el se puede alterar la data.

### desmontarla y volver

`umount /media/vmvd/vd1p1`

**IMPORTANTE** deben desmontarse todas las particiones en uso de la imagen/disco en uso

`umount /media/vmvd/vd1`

# Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.
