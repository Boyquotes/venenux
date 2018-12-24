## requisitos:

1. disco duro donde se va instalar
2. una maquina donde se tenga otr disco con linux ejecutandose y:
   * virtualbox
   * espacio al menos 8G
   * iso de linux a instalar (usaremos debian venenux)
   * el usuario que ejecuta virtualbox debe estar en el grupo disk y adm
3. alguna manera de que el disco donde se instalara linux se vea y acceda directo en la maquina

## Preparacion:

Se debe tener requisito esto:

* poder o tener el disco accedible externamente o internamente desde la maquina donde esta virtualbox
* pertenecer al grupo disk (con `usermod -a -G disk virtualmachines` siendo `virtualmachines` el usuario
* no tener montado ninguna particion o tner ningun acceso al disco, el acceso es exclusivo

Se crea el acceso para virtualbox con el comando

`VBoxManage internalcommands createrawvmdk -rawdisk /dev/sdb /var/virtualbox/vdi/rawdisk-sdb.vmdk`

Despues configurar an maquina virtual que carge el vmdk creado OJO el disco debe ser desmontado

En la maquina virtual cargar la iso del linux que se va instalar en el disco

## instalacion

Arrancar la maquina virtual y realizar la instalcion normal.. 

OJO al estar finalizando cuidar que:

* No usar el x86 virtual tilidades, algunos instaladores ofrecen instalar el paquete `virtualbox-guest-utils` 
este paquete no se debe usar porque la virtual es solo para saltarse el usar cdrom o usb
* Debe escoger drivers genericos al finalizar instalacion de grub loader, si escoge dedicado no podra arrancar 
en la maquina destino porque requerde que el disco cree que el virtualbox es la maquina real..

