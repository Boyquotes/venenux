# 1.- Servicios de VirtualBox

Coloca en un usuario unico todas las ejecuciones de maquinas virtuales, 
y los administra al inicio y al final. 

Maquinas virtuales solo ofrecen la ventaja de isolacion de entorno 
si configuran sus interfaces de red en NAT modo.

### Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.

### Requerimeintos

* Hardware
  1. CPU con VT-X o AMD-V
  2. Espacio disco >> 300 Gb
* Software
  1. Virtualbox >> 3.2
  2. gcc >> 4.1
  3. autotools
  4. linux-headers

### instalar virtualbox

`apt-get install virtualbox libguestfs-tools virtualbox-source build-essential module-assistant dpkg-dev dkms`

En caso de ser muy viejo el linux o antigua la distro:

`apt-get install virtualbox-ose virtualbox-ose-fuse virtualbox-ose-source build-essential module-assistant dpkg-dev`

### construir/cargar modulos virtualbox

hay que instalar las cabeceras o headers de el kernel que se esta ejecutando, se determina asi:

``` bash
apt-get install linux-headers-$(uname -r|sed 's,[^-]*-[^-]*-,,') virtualbox-source
```

el modulo se construye **usando module-assistant, el mismo paquete construido 
del modulo se usa en otra instalacion**, sin embargo `dkms` hace esto dinamicamente 
en distros modernas. (en distro viejas usar `virtualbox-ose` en vez de `virtualbox` solo)

`m-a prepare`

`m-a build virtualbox`

`m-a install virtualbox`

El documento detallado de como realizarlo esta en la documentacion previa 
[virtualbox1essentials.md#preparacion-instalacion-base](virtualbox1essentials.md#preparacion-instalacion-base)

`modprobe vboxdrv`

`modprobe vboxnetflt`

# 2.- Crear maquinas virtuales

Para no tener que usar graficos ni el paquete de virtualbox extendido privativo, 
se tiene una imagen VDI de venenux Debian squeeze en el drive de vegnuli, 
sin embargo se puede instalar con cualqueir interfaz en dhcp sin importar que ip tenga la virtual, 
esto porque se configurara uan regla nat que permite usar el puerto 22 en el 20X22 tal como se 
muestra mas adelante.

### Crear usuario

`adduser --system --group --shell /bin/dash --no-create-home --home /var/virtualbox --disabled-login --disabled-password  virtualmachines`

### configurar home y descargar vdis base

```
mkdir -p /var/virtualbox/vdi
mkdir -p /var/virtualbox/scripts/
cd /var/virtualbox/vdi && wget http://10.10.34.22/admins/virtualmachines/vm-vdi-debian-6-base-disk.vdi.bz2
gunzip -x vm-vdi-debian-6-base-disk.vdi.bz2
chown -R virtualmachines:virtualmachines /var/virtualbox/
```

### crear maquinas virtuales

`su -s /bin/bash --preserve-environment virtualmachines`

`VBoxManage createvm --name vmERP1server --ostype Debian --register --basefolder /var/virtualbox/vm`

### Configurar maquina virtual: ram, os, audio, usb, bios

`VBoxManage modifyvm vmERP1server --name vmERP1server --ostype Debian --memory 1024 --audio none --acpi on`

`VBoxManage modifyvm vmERP1server --mouse usb --keyboard ps2 --usb on --usbehci off --ioapic off`

El soporte ECHI requiere usar software extra, por ello se desabilita; se asume servidor, sin embargo si se trabajara audio cambiar `none` por `alsa` ya que usar `pulseaudio` significara mayor consumo de recursos.

### Configurar maquina virtual: activar virtualizacion por hardware

`VBoxManage modifyvm vmERP1server --nestedpaging off --hwvirtex on --vtxvpid on`

El procesador debe ser superior al anio 2009 con VT-X/AMD-V disponible, y activarlo, esto es en el bios

El **vtxvpid** es solo en procesadores que sean intel unicamente, acelera las transacciones de buffer en la memoria (TLBs) entre la virtual y la real.

El nested paging solo se puede habilitar en procesadores mas alla del año 2010, esta "nested paging" implementa manejo de memoria por hardware.

### Configurar maquina virtual: video

`VBoxManage modifyvm vmERP1server --name vmERP1server --vram 24 --accelerate2dvideo off --accelerate3d off`

`VBoxManage setextradata vmERP1server GUI/Fullscreen/LegacyMode 1`

### Configurar una interfaz de red y acceso ssh

`VBoxManage modifyvm vmERP1server --nic1 none`

`VBoxManage modifyvm vmERP1server --nic1 nat --nictype1 82545EM --cableconnected1 on --macaddress1 080027E03936`

`VBoxManage modifyvm vmERP1server --nic2 none`

VirtualBox permite interfaces con su propio MAC pero isolado en un entorno separado, usando NAT con una ip cerrada:
Para acceder a sus puertos y servicios se redireccionan a otros por encima de 1024, se usara formato 20XXX asi:

`VBoxManage modifyvm vmERP1server --natpf1 "vmERP1serverssh,tcp,,20022,,22"`

### Configurar una interfaz de red real puente

`VBoxManage modifyvm vmERP1server --nic1 none`

`VBoxManage modifyvm vmERP1server --nic1 bridged --nictype1 82540EM --cableconnected1 on --bridgeadapter1 eth0`

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/0/LUN#0/Config/RestrictAccess 0`

`VBoxManage modifyvm vmERP1server --macaddress1 080027401131 `

**importante** la maquina virtual no ofrece inmediato la ip de la emulada, 
PARA SABER LA IP CON EL COMANDO `VBoxManage --nologo guestproperty get "name" /VirtualBox/GuestInfo/Net/1/V4/IP` 
SE REQUIERE GUEST ADDITIONS INSTALADO EN EL SISTEMA EMULADO.

### configurar con dos interfaces de red

Una estara a la conexcion real de la maquina y el OS real, en puente, obteniendo una ip real, 
la otra estara virtual, en modo `NAT` oculta, para esto debemos deshacer cualquier configuracion 
de extradata asi:

`VBoxManage modifyvm vmERP1server --nic1 none`

`VBoxManage modifyvm vmERP1server --nic2 none`

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/0/LUN#0/Config/RestrictAccess`

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/1/LUN#0/Config/RestrictAccess`

`VBoxManage modifyvm vmERP1server --nic2 bridged --nictype2 82540EM --cableconnected2 on --bridgeadapter2 eth0`

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/1/LUN#0/Config/RestrictAccess 0`

`VBoxManage modifyvm vmERP1server --nic1 nat --nictype1 82545EM --cableconnected2 on`

`VBoxManage modifyvm vmERP1server --natpf1 "vmERP1serverssh,tcp,,20022,,22"`

**NOTA**: la imagen VDI `vmERP1server` descargada del DRIVE venenux tiene dos interfaces "registradas", 
las mac son estas, sin embargo con el ssh en 22 no importa si es dhcp o que ip tenga, 
se puede obtener conexion usando el puerto 20022 en el ssh, la ip remota sera del tipo `10.0.X.Y` 
sin importar que ip tenga.

`VBoxManage modifyvm vmERP1server --macaddress1 080027E03936 --macaddress2 080027401131 `

**importante** la maquina virtual no ofrece inmediato la ip de la emulada, 
PARA SABER LA IP CON EL COMANDO `VBoxManage --nologo guestproperty get "name" /VirtualBox/GuestInfo/Net/1/V4/IP` 
SE REQUIERE GUEST ADDITIONS INSTALADO EN EL SISTEMA EMULADO.

### modificar disco y controlador disco

`VBoxManage storagectl vmERP1server --name vmERP1serverSATA1 --add sata --hostiocache on`

`VBoxManage storageattach vmERP1server --storagectl vmERP1serverSATA1 --port 0 --device 0 --type hdd --medium /var/virtualbox/vdi/vm-vdi-debian-6-base-disk.vdi`

`VBoxManage storagectl vmERP1server --name vmERP1serverSATA1 --sataportcount 1`

**ALERTA**: porcount/sataportcount es 1 siempre para sata, de 1 a 4 para ide y hasta 30 para scsi.

**CUIDADO**: el hostiocaching es porque MSDOS y linux antiguos no soportan SATA ni AHCI, 
por ende al usar IDE debe dejarse hostiocache abilitado. Sin embargo se puede desactivar para SATA 
si el sistema operativo real que tiene el virtualbox es muy estable y confiable en terminos 
de configuracion.

**NOTA:** el disco vdi ya estaba creado sin embargo al crear uno hay que iniciar graficamente la VM, 
esto es imposible sin entorno grafico asi que la imagen vdi puede crearse 
y la instalacion realizarse afuera de el lugar donde ejecutan las VM, 
y despues descargarlas en el lugar donde se ejecutaran.

### Configuracion bios y arranque/prueba

`VBoxManage modifyvm vmERP1server --boot1 disk  --firmware bios`

`VBoxManage startvm vmERP1server --type headless`

### entrar y probar ssh por NAT

`ssh root@localhost -p 20022`

Esto ejecutandose desde la misma maquina de donde se arranco. 
Se recuerda que se agrego a el `nic2` que estaba en NAT el redireccionamiento de el puerto de ssh.

Cambie `localhost` por la ip de la maquina que esta ejecutando el comando `VBoxManage` 
es decir por la ip de la maquina real que tiene la maquina virtual.

### apagar la maquina virtual desde manejador

`VBoxManage controlvm vmERP1server acpipowerbutton`


# 3. Crear Virtuales Guindows detectar con vnc

1. la tarjeta de red debe ser PCFASTNETIII https://forums.virtualbox.org/viewtopic.php?f=8&t=50627
2. el remote desktop debe estar activado y el usuario a conectar debe tener clave
3. el puerto (si usamos NAT) no se puede alterar, asi que se empleara otro soft remoto, como VNC para detectar
4. basicamente la parte del ssh se realiza pero con el vnc, una vez adentro se ve la ip del interfaz puente y listo.

el IO APIC debe estar activado..
Controladora: IDE, el disco duro va de primero, cdrom de segundo
interfaz1 NAT PCFASTNETIII
interfaz2 PUENTE PCFASTNETIII
redireccionar puerto 5900 a 35900 (pensar despues un numero de formato)
sin audio

VBoxManage setextradata des-wnt5-sdektop5 GUI/Fullscreen/LegacyMode 1

# 4. Crear un disco duro virtual DESDE UNO REAL

Se debe tener requisito esto:

1. pertenecer al grupo `disk` (con `usermod -a -G disk virtualmachines` siendo `virtualmachines` el usuario
2. no tener montado ninguna particion o tner ningun acceso al disco, el acceso es exclusivo


Se crea con el comando 

`VBoxManage internalcommands createrawvmdk -rawdisk /dev/sdb /var/virtualbox/vdi/rawdisk-sdb.vmdk`

# 5. Convertir en servico las VM del sistema

Esto sera cubierto en un documento aparte.

# Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.
