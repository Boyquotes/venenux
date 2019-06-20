# VirtualBox #

***VirtualBox es*** **software virtualizar computadores modesto y facil** de manejar, para oficinas o entornos comunes, 
mientras que **VMware, Xen o MSvirtual es para entornos grandes pero mas complejos** aunque de mayor control.

Documentacion VirtualBox VenenuX Debian linux:

1. [virtualbox1essentials.md](virtualbox1essentials.md) que es este documento.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.

## Historia Virtualbox ##

Desarrollado por Innotek. Posteriormente, Sun Microsystems adquirió Innotek y despues paso a Oracle al comprar este a Sun.

**Inicialmente** era en **dos ediciones diferentes** la abierta y la privada: 
**A)** VirtualBox Full Binary Edition: gratis para uso personal y con fines de evaluación. El código fuente de esta edición no está disponible y las licencias deben comprarse para uso comercial del producto. 
**B)** VirtualBox Open Source Edition (OSE) - Libremente disponible para uso personal y comercial e incluye código fuente publicado bajo los términos de la Licencia Pública GNU (GPL) evidentemente con limitaciones.

**Despues se unifico, pero con las adiciones aparte**, una version unica, y la parte licenciada se ofrece en un "paquete de extensión extra" disponible únicamente de forma gratuita pero binaria que provee adiciones.

## Caracteristicas ##

Virtualbox tiene dos partes, el programa base, y los driver/modulos que se integran al hardware de virtualizacion.

Virtualbox usa tanto discos virtuales como reales, **lee cualquier otro formato de disco virtual**, 
aunque ninguno puede leer el formato de disco de virtualbox

VirtualBox es un proceso isolado por usuario, **reventones en el sistema emulado o `Invitado` (guest) nunca afectaran** al sistema real (host) que corre el proceso.

#### Caracteristicas libres y alterables: ####

- Soporte de Hardware de virtualizacion AMD-V u Intel-VT
- Discos virtuales, cada disco duro emulado es un archivo unico tambien accedible
- Interfaces tanto grafica como minimas, inclusive administracion por comandos
- Emulacion total de arquitectura x86 y x64, con perfiles especiales para cada sistema operativo popular conocido.
- Soporte USB: proporciona acceso de máquina virtual a dispositivos USB conectados al sistema de la computadora host.

#### Caracteristicas adiciones con el "paquete de extension": ####

- Servidor de Protocolo de pantalla remota (RDP): permite el acceso remoto a máquinas virtuales a través de clientes basados en RDP.
- USB sobre RDP: permite que los dispositivos USB conectados a un sistema local estén disponibles para las máquinas virtuales que se ejecutan en hosts remotos.
- Iniciador iSCSI: permite que los dispositivos de almacenamiento iSCSI se usen como discos virtuales sin necesidad de controladores iSCSI en el sistema operativo invitado, similar a los SAS de Dell.

### Limitaciones ###

- Memoria fija, lo que la limita a menos de la mitad de la real, por la necesida de las operaciones en la real.
- Memoria video maxima a 128Megas, por la dificultad de cambiar entre ventana y pantalla la virtual.
- Lenta transferencia entre USB virtual y USB real, ademas de soporte limitado a USB3.
- EFI solo basico, no hay un set completo de el soporte UEFI, solo EFI.
- Emulacion de sistemas MAC limitado a soporte basico, igual para sistemas menos populares.
- Emular otro hardware distinto al de la instalacion real, solo se puede limitado y en ultimas versiones.

## VirtualBox y su unica Architecture ##

Inusual y especial, dado emplea las dos formas: **software virtualization** y **hardware virtualization**. 
Este es el orden en que VirtualBox emplea su virtualizacion combinada:

1. software virtualization, 
1.1. shared kernel virtualization, 
1.2 kernel level virtualization, 
1.3 hypervisor virtualization, 
1.4 paravirtualization, 
1.5 full virtualization 
2. hardware virtualization.

Desde el primer nivel el OS cree que el computador es real, adicional, 
mediante el ultimo nivel se le provee acceso directo a partes de hardware como leer puertos USB y acceder a redes.

**IMPORTANTE** la virtualizacion de hardware necesita estar presente en el CPU, como las de **Intel VT** y/o **AMD-V** 
las cuales deben habilitarse en el BIOS de la maquina computador.

**NOTA**  Para la emulacion en 64bits la virtualizacion por hardware es mandatoria.

### Multiemulacion ###

Cuando se emula mas de una maquina virtual hay algunas limitaciones:

* USUARIO: el usuario que ejecuta el virtualbox es el que arranca las virtuales, virtualbox tiene en cuenta esto para varias virtuales.
* CPU-V: todas deben ser de la misma arquitectura o son 64bits o son 32bits sino se desactiva virtualizacion hardware.
* RED: no se puede compartir el dispositivo, a menos todas las maquinas virtuales se ejecuten por el mismo usuario que arranca las virtuales.
* DISCOS: no se puede acceder o escribir en un disco virtual mientras esta siendo usado por la maquina virtual

## Interfaz e Instalacion

Virtualbox provee **interfaz grafica asi como linea de comandos**, la interfaz 
se instala con `virtualbox-qt` mientras que la linea de comandos esta asumida en el software en si.

Virtualbox requiere una **instalacion base, consta de modulo de kernel para gpu asi como modulo de kernel de red**, 
tambien de un **script de inicio** que se encarga de que estos esten cargados y presentes.

### instalacion base

`apt-get -y install virtualbox virtualbox-source build-essential module-assistant`

### instalacion grafica

`apt-get -y install virtualbox virtualbox-qt virtualbox-source build-essential module-assistant`

### instalacion linux viejos

`apt-get -y install virtualbox-ose virtualbox-ose-source build-essential module-assistant`

## Preparacion instalacion base

`apt-get -y install linux-headers-$(uname -r|sed 's,[^-]*-[^-]*-,,') virtualbox-source`

`m-a prepare`

`m-a build virtualbox-source`

Aqui se abrira una ventana de `ncurses` gris con azul y rojo, que mostrara los avances.

El rsultado es un paquete para instalar en /usr/src/ especifico para el kernel en ejecucion asi:

`dpkg -i /usr/src/virtualbox-modules*$(uname -r|sed 's,[^-]*-[^-]*-,,')*.deb`

# Comparacion de Virtualbox contra Docker


## Tabla comparativa software virtuales

|  item       | virtualbox        | bochs               |  qemu y/o KVM          |  vmware(soft)  | Docker/Xen    |
| ----------- | ----------------- | ------------------- | ---------------------- | -------------- | ------------- |
| Hardware VT | Intel-VT/AMD-V    | NO, soft only       | Indirect-access        | Direct-access  | Direct-access |
| Limites     | disco, cpu, ram   | disco, 1 cpu, ram   | disco, cpu, ram, gpu   | disco          | Ilimitados    |
| Rendimiento | Cercano al real   | Pobre y flexible    | Mediano                | Cercano al real | Alto, bueno   |
| CPU/instala | x86/x64           | x86/x64/ARM/PwrPC   | x86/x64/ARM/PwrPC/m68k | x86/x64        | solo el os    |
| CPU/emula   | x86/x64/PwrPC     | x86/x64            | x86/x64[ARM/PwrPC/m68k] | x86/x64        | solo el os    |
| OS/emula    | Linux,Mac,BSD.Win | Lnx,Mac,BSD,Os2,Win | Lnx,Mac,BSD,Os2,Win    | Linux,Win      | solo el os    |
| OS/instala  | Linux,Mac,BSD.Win | Lnx,Mac,BSD,Os2,Win | Lnx,Mac,BSD,Os2,Win    | Linux,Win      | Linux         |
| InterfazGUI | Si, completa      | NO                  | Si, terceros/incomplet | Si, completa   | No            |
| Pausar      | Si, recuperable   | NO                  | Si, recuperable        | Si             | Si, recuperab |
| Ram (dinam) | Si, reasignable   | NO                  | Si, reasignable        | Dinamica       | No, la del os |

## Tabla comparativa de imagenes disco virtuales y acceso

|  tipo       | virtualbox | bochs     | qemu y/o KVM | vmware(soft) | Docker/Xen   |
| ----------- | ---------- | --------- | ------------ | ------------ | ------------ |
| ISO         | Si         | Si        | Si           | Si           | No           |
| USB 1.1     | Si         | Si        | Si           | Si           | Si           |
| USB 2/3     | No         | No        | Si           | Si           | Si           |
| HDD(disco)  | Si         | Si        | Si           | Si           | Si           |
| HDD(parti)  | Si         | Si        | Si           | Si           | Si           |
| RAW         | Si         | Si        | Si           | No           | Si           |
| VDI         | Si         | No        | No           | No           | No           |
| VHD         | Si         | Si        | No           | No           | Si           |
| VMDK4       | Si         | Si        | No           | Si           | No           |
| VMDK3       | Si         | Si        | Si           | Si           | No           |
| QCOW        | Si         | No        | Si           | No           | No           |
| QCOW2       | Si         | No        | Si           | No           | No           |

# Conclusiones

**Para emular y contener en general, el mejor caso es VirtualBox** 
para iniciados y administracion nivel novato o medio es VirtualBox.

Configurar un Docker o un VMWare es complicado si no se tiene el conocimiento, 
especialmente si no provee interfaces graficas para los iniciados.

**Para alto rendimiento se requiere mejor Docker o VMWare/XEN** ya que 
los contenedores acceden directo, pero requiere mayor nivel de experiencia con estos.

## Casos de usos virtualbox vs contenedores

Un docker no puede emular software de distinta arquitectura, mucho menos de distinto sistema operativo.

Las maquinas virtuales son **mejores y mas faciles de administrar por su aislamiento**, es decir, 
se le puede dejar la administracion de la virtualizada a un experto sin que este altere el entorno real, 
de esta manera no hay que administrar accesos reales al sistema real, 
si algo malo sucede, se reestablece el respaldo facilmente.

Los contenedores **Docker deben usarse cuando se emplee porciones de software**, 
mientras que las maquinas virtuales separan por completo el entorno invitado (emulado/guest) 
proveyendo mayor facilidad en la seguridad.

Las maquinas **virtuales se pueden migrar/exportar a otras reales (host) sin alteraciones**, 
los contenedores requiren volver adaptarse al nuevo sistema que los ejecutan, 
puestos estos dependen del sistema operativo que los implementan.

Los **contenedores son muchisimos mas veloces** y optimos que una maquina virtual.

Las maquinas **virtuales aislan el entorno, los contenedores o Docker son alterados 
con las actualizaciones respecto el sistema operativo que los ejecutan**. 
Esto aplica cuando se necesita ejecutar un software especial para otros sistemas distintos 
ejemplo, arquitectura distinta o incluso ditinto sistema operativo.

# Este documento continua en:

1. [virtualbox1essentials.md](virtualbox1essentials.md) Lo inicia, explica y dice que instalar.
2. [virtualbox2serviciosvirtuales.md](virtualbox2serviciosvirtuales.md) Detalla como crear sin graficos virtuales.
3. [virtualbox3gurumeditation.md](virtualbox3gurumeditation.md) Explica algunos problemas mas raros en estas.
4. [virtualbox4montarvdivmdk.md](virtualbox4montarvdivmdk.md) Dice como extraer datos sin arrancar las virtuales.
5. [virtualbox5scriptsactivas.md](virtualbox5scriptsactivas.md) Avanza como automatizar las virtuales en un server.
