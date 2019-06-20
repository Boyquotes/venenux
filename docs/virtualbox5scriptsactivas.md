## Convertir imagenes entre formatos

Convertir entre imagenes requeire siempre dos pasos, 
el primero de la original a RAW y el segundo de la raw a la destino, 
el segundo paso es usando la utilidad de el software destino, 
aqui un ejemplo de conversion entre VDI hacia QCOW de qemu imagenes:

**REcomendaciones:**

* usar discos duros distintos para archivos origen y destino, facilita por la escritura I/O
* no usar discos externos via USB ya que son lentos o discos compartidos o de red.

Esto es porque siempre estas imagenes de discos duros son archivos grandes.

## documentacion:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Para montar particiones creadas pro VMDK/VDI

### requisitos instalar

`apt-get install qemu-utils`

### Paso 1 vmdk->qcow: From vmdk to raw

`qemu-img convert -p -f vmdk -O raw original_image.vmdk new_image.raw`

### Paso 2 vmdk->qcow: From raw image to qcow2

`qemu-img convert -f vmdk -O qcow2 new_image.raw new_image.qcow2`


## Reconectar la conexcion de la virtual cuando la real se pierde

Cuando su host pierde la conexión de red y sus rutas cambian (la conexión vpn se pierde y despues vuelve a conectar) la única manera de volver a conectar la red dentro de la máquina invitada/virtualizada es encender y apagar la tarjeta de red de invitado/virtual. El siguiente script hace esto:

``` bash
#!/bin/bash
echo "$0: connecting and reconnecting network of all virtual machines ... "
VBoxManage list runningvms | awk '{system("echo " $0 " |  cut -d'{' -f1 | sed s/\ $//g ;")}' | awk '{system("VBoxManage controlvm \""$0"\"  setlinkstate1 off; sleep 2; VBoxManage controlvm \""$0"\"  setlinkstate1 on;")}'
sleep 5
VBoxManage list runningvms | awk '{system("echo " $0 " |  cut -d'{' -f1 | sed s/\ $//g ;")}' | awk '{system("VBoxManage controlvm \""$0"\"  setlinkstate1 off; sleep 2; VBoxManage controlvm \""$0"\"  setlinkstate1 on;")}'
echo "$0: done."
```

## apagado automatico muy rapido

En versiones modernas de los paquetes Virtualbox, los script apagan las maquinas virtuales 
(al menos en el reciente libvirt-bin package), pero solo 30 segundo, 
lo cual es muy poco si las virtuales son pesadas:

En las modernas editar `/etc/init/libvirt-bin.override` asi:

``` bash
$ vim /etc/init/libvirt-bin.override
#extend wait time for vms to shut down to 4 minutes
env libvirtd_shutdown_timeout=240
```

## Migration from VirtualBox to KVM

This boils down to

    having a lot of time
    having a lot of free harddisk space
    creating a clone of the vbox-machine with VBoxManage clonehd (this can take a looooong time!). Kloning is the easiest way of getting rid of snapshots of an existing virtual machine.
    converting the images from vdi to qcow-format with qemu-img convert
    creating and configuring a new kvm-guest
    adding some fou to NAT with a qemu-hook (see next section)

    http://serverfault.com/questions/249944/how-to-export-a-specific-virtualbox-snapshot-as-a-raw-disk-image/249947#249947

To clone an image - on the same machine - you have to STOP kvm and start vboxdr (see above). Also be aware, that the raw-images take up a lot of space!

```
# The conversion can take some time. Other virtual machines are not accessible in this time
VBoxManage clonehd -format RAW myOldVM.vdi /home/vm-exports/myNewVM.raw
0%...

cd /home/vm-exports/
qemu-img convert -f raw myNewVM.raw -O qcow2 myNewVM.qcow
```

#### Cloning a Snapshot:

```
# for a snapshot do (not tested)
cd /to/the/SnapShot/dir
VBoxManage clonehd -format RAW "SNAPSHOT_UUID" /home/vm-exports/myNewVM.raw
```


When you start the virtual machine with plain kvm-qemu you will certainly get a BSOD. This can be circumvented starting the machine according to the following article:

    https://bbs.archlinux.org/viewtopic.php?id=194350

... with something like

```
qemu-system-x86_64
...
-machine q35,accel=kvm
-drive if=none,file=your.image
-device ahci,id=ahci
-device ide-drive,bus=ahci.0,drive=d1
```

I by myself found out, using these settings in virt-manager works also very well. I also set

    audio to ac97 and
    network to the e1000 driver

Then I start the machine in safe mode with networking and install the virtio drivers for networking from linux-kvm.org and the spice video guest drivers and qxl drivers from spice-space.org. 

# Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.
