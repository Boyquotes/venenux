El error "*Guru meditation state*" ocurre cuando se tiene mala configuracino de CPU, RAM o NET, 
por ejemplo la maquina con virtualbos esta en 32bit y el os a emular es 64bit, 
o se configuro mucha ram o casi mas de la mitad respecto la real. 

Casos especiales en CPU Intel antiguas que no implementan ejecución sin restricciones en VT-x.

### Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.

### Mensages tipicos o en el archivo log:

* `Changing the VM state from 'RUNNING' to 'GURU_MEDITATION'`
* `Guru Meditation -1618 (VERR_PGM_PHYS_PAGE_RESERVED`
* `Guru Meditation -37 (VERR_NOT_SUPPORTED)`

Pero encontrar como arreglarlo es dificil ya que no se sabe la causa real 
puesto es un problema de configuracion del xml/vdm y sucede mucho cuando 
se crean VM por linea de comandos, asi que pongo los casos mas comunes que lo pueden solventar:

## CASO1: memoria mal asignada o no accedible

es muy comun si usas varias VM y estas son de usuarios distintos, aqui 
es facil si tienes 2G de ram y ya tienes una VM con 1G asignada, 
si tratas de arrancar otra VM con tambien 1G asignada esta fallara con ese error

la solucion es disminuir ambas a 512 ya que 
**no puede arrancarse varias VM y estas abarquen mas de la mitad de la ram** 
del sistema real. Menos si son desde ditintos usuarios en la misma maquina real.

## CASO 2: mala configuracion de discos/ide/sata/scsi

Es **incorrecto numero de discos en un controlador de disco**, 
el problema es que por consola el comando no hace esto automatico, 
y es porque **si es IDE se puede llevar hasta 4, si es SCSI se lleva hasta 30 
pero si es SATA hay que poner siempre 1**, 
porque obviamente un puerto SATA tiene un solo disco y un solo cable sata, 
este es el error relacionado a controlador de discos.

`VBoxManage storagectl vmERP1server --name SATA1 --sataportcount 1`

En las versiones mayores a 4 es asi:

`VBoxManage storagectl vmERP1server --name SATA1 --portcount 1`

**OJO** hay que especificar el nombre del controlador, si es por los graficos 
siempre se pone "Sata controller" aqui el hardware sata se llama "SATA1" 
y el caso es sata que siemrpe debe ser "1"

## CASO 3: mala configuracion de red

tenemos dos tipos que son los mas usados de los 4, el NAT y el BRIDGE solamente 
cuando es NAT se puede redireccionar y acomodar los puertos internos hacia los reales 
del sistema operativo real, si se hace esto en un BRIDGE dara el error

esto es dificil porque pedir detalle de la maquina no lo muestra, 
solo ocurre en versiones viejas de VBox, 
ejemplo de como borrar redirecion en nateo puerto 2:

`VBoxManage modifyvm deskdeb4vm1 --natpf2 delete  deskdeb4vm1ssh2`

esto borrara una redireccion de la interna VM 22 a la real en 22022 
que se coloco incorrectamente despues de configurar ese nic2 como puente o BRIDGE 
resultado de este comando:

`VBoxManage modifyvm deskdeb4vm1 --natpf2 "deskdeb4vm1ssh2,tcp,,22022,,22"`

Esto ocurre porque el configurador inicialmente (y aun hoy pero otra extension) emplea archivos XML, 
el configurador no ve si se puede o no simplemente adiciona la seccion configurada en donde debe, 
que es en el nic2 pero este no es nat por ende esta mal configurado

## CASO 4: distinto arquitectura:

ejemplo instalar un OS 64bit en un VM 32bit genera ese error

## CASO 5: arrancar VM distintas desde usuarios distintos

por defecto hay restriccion que imposibilita **arrancar puente en la misma interfaz 
con distintos usuarios arrancando varias vm**.. es un caso muy especial 
pero se puede dar en una maquina que arranque varias VM cada una con un usuario distinto:

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/1/LUN#0/Config/RestrictAccess 0`

aqui es "1" porque es la segunda nic, pero en el primer y unico dispositivo puenteado (LUN#0)
para borrarlo es asi:

`VBoxManage setextradata vmERP1server VBoxInternal/Devices/e1000/1/LUN#0/Config/RestrictAccess`

**IMPORTANTE** si se configura incorrectamente este parametro fallara, 
por eso el comando para borrar, hay que intentar hasta configurar correctamente

**OJO** este debe ser a TODAS las que usaran el dispositivo a puentear sino no aplica.

# Vease tambien:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.
