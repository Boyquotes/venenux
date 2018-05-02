## Introducción

El principal problema con ChromeOS es que **google está perfectamente integrado**, 
si, **DEMASIADO google, tanto que sin eso no se instala nada!** para rematar, 
cuadno su tiempo de soporte termina no les puedes instalar nada y muchas están 
alcanzando poco a poco el final de la vida (de julio de 2018, para la 303 samsung).

ESTOY EN CONTRA DE EL OBSOLESCENCIA asi que dado una persona la necesitaba, 
y estamos en crisis aprendemos apreciar las cosas:

El linux que mejor soporta las ARM es Arch, despues gracias a Debian el Kali linux 
esto porque Debian tiene ARM como arquitecturas soportada. Kali sin Debian no es nadie.

## Typos de Chromebooks

HAy dos tipos segun la arquitectura:
* ARM como las Samsung X303, que corren Exynos ARM, o las Acer que corren Rockchip
* INtel que las hay super viejas que corren Atom y Celeron y las nuevas i7

Entre las intel tenemos las **Broadwell** (que traen i7),
y las **Bailtray** (Silvermont Atom), las escazas **Haswell-ULX**;
estas **intel son las mejores, especialmente las Bailtray, Broadwell**, en estas  
puede colocarse GalliumOS un windubuntu para estas que hasta arranca dual boot 
pero ojo solo a las compatibles se les garantiza. Pero hay unas 
muy nuevas **Pineview** y **Apollolake** en estas no se garantiza nada.

Entre las arm tenemos las llamadas **Exynos**, las **Tegra** y las **RK** de Acer; 
hay que tener en cuenta que estas **son las peores, y mas dificiles de 
alterar**, si pretende cambir algo es mejor un crouton con Debian, 
auqnue la mas recomendada es ARchlinux, lo que si es seguro 
no correr las estupidas Winbuntu, Winsuse o Winhat, porque no 
soportan ARM, y no las instale dualboot, porque el bios es un problema.

**LA regla de oro basica es ARM->chroot/crounton, Intel/vieja->GalliumOS, Intel/nueva->chroot/GalliunOS**

**IMPORTANTE** la mejor fuente de info es http://linux-exynos.org/wiki/Installing_a_rootfs para las ARM

## Caracteristicas comunes:

* Procesador: siempre un 2 nucleos @ 1.7Ghz (ARM o Intel)
* Memoria: entre 2GB DDR3L y 8GB
* Disco: 16GB eMMC SSD o 32GB eMMC SSD
* Pantalla: siemrpe 11.6" / 1366x768 a 2024x1024
* Red: WiFi 802.11 a/b/g/n (Dual Band)
* Video: VGA (dongle) con HDMI siempre
* USB: 1x USB 3.0 / 1x USB 2.0 en las nuevas USB-c
* Tarjetas: SD/SDHC/SDXC
* Sonido: JAck 2mm microfono/audifono mixto en las nuevas se eliminara

## Developer Mode y Verificacion

Para tomar el control de la Chromebook es cambiar a **modo de desarrollador.**
Este deshabilitar el núcleo de verificación de firmas y dejar ejecutar cualquier 
firmware de la máquina con el inconveniente de tener en cada inicio de una 
molesta pantalla de advertencia...
**IMPORTANTE** no! la pantalla de advertencia NO ES reinstalar el chrome OS!

1. En primer lugar, apague la máquina
2. Despues presionando al mismo tiempo las teclas ESC+REFRESH o ESC+F3 y despues 
sin despresionar encender con la tecla POWER, es decir las tres al mismo tiempo 
pero la de power de ultimo. Esto saca una imagen con algo de "device inserted" 
o sino una imagen con un signo de exclamacion grande. Si sale la primera solo 
presionar CTRL+D y aparecera la imagen de signo de exclamacion
3. Si sale la pantalla de signo de exclamacion dice que debe recuperar el chrome OS 
y algo de USB, aqui presione CTRL+D al mismo tiempo, y .... rapidamente...
4. Si sale pantalla blanca con letras de "Verification OFF .. enter" y pulse ENTER 
es decir confirme pulsando enter, que se apagara rapidamente y espere re_encienda....
5. Al volver encender aparece una pantalla de advertencia "OS Verification OFF" y 
despues de 30 segundos se escuchara dos pitidos, y a lo cual volvera reinicar ...
y justo despues de este reinicio empieza un conteo arriba y una barra avanza 
esto durara de 10 a 15 minutos y borrara toda la data de su sesion (por favor "su", sera de google)
6. Cuendo termine reiniciara nuevamente, y en cada reinicio mostrara la pantalla "OS Verification OFF" 
esta pantalla se quita a los 30 segundos o presionando CRTL+D sobre ella

https://www.chromium.org/chromium-os/poking-around-your-chrome-os-device#TOC-Putting-your-Chrome-OS-Device-into-Developer-Mode

## Consola VT vs Consola Crosh vs Real

Las consolas solo estan activas despues de activar el modo desarrollador.

La diferencia es que en la VT no influye o no vale todo lo que es grafico, 
en la "crosh" variables de influencia de el entorno grafico estan disponibles 
es por esta razon que la mayoria de los scripts se mandan ejecutar aqui.

La **consola Crosh** es una consola limitada pero "favorecida" ya que es la 
unica que se integra al entorno X, se abre con CTRL+ALT+T y aparece en una 
pestaña del navegador. Hay que ejecutar al entrar el comando `shell` .

La **consola real** esta realizando CTRL+ALT+"->" esta flecha es la flecha de 
navegacion arriba y no la flecha de direccion, es la tecla de funcion F2 si 
se ve de izquierda a derecha, esto es tambien hacer CTRL+ALT+F2 simultaneamente.

## Developer Mode y USB BOOT

Una vez que el desarrollador modo está habilitado, es posible habilitar el arranque por USB:

7. Como la data se ha borrado, para poder trabajar despues que esta pantalla 
de "OS Verification OFF" arranque, dejela pasar Y NO PRESIONE SPACE
8. Cuando muestre el logo de "Chrome" de colores y pro fin carge de nuevo 
alli es cuando pulsara CRTL+ALT+"->" o sino CTRL+ALT+F2 al mismo tiempo y 
esto mostrara uan consola de Linux bash para hacer login.
9. escriba "chronos" como nombre de usuario, esto lo iniciara login de consola
10. escriba "sudo su" esto le dara acceso de root, windoseros ignorantes no seguir
11. ejecutar el comando `crossystem dev_boot_usb=1 dev_boot_signed_only=0`

https://www.chromium.org/chromium-os/poking-around-your-chrome-os-device#TOC-Get-the-command-prompt-through-VT-2

## Error: mensaje insertar USB y nunca arranca

Estas porquerias de google son Linux y si pulsas TAB en el inicio mostrara el error del kernel

1. si **revcovery_reason="0X54"** reintentar prender al menos 40 veces hasta que arranque.
2. si no, presiona CRTL+D en la pantalla y si vuelve preguntar "To turn OS Verification OFF"

De aqui en adelante igual que el paso numero 5 (cinco).

## Esquema de particiones y sequencia arranque

Aqui hay una caja negra pero hay vestigios de luz:

Emplea U-BOOT que afortunadamente es opensource, se especializa en ARM:

* al encender el CPU ejecuta U-BOOT desde el SPI flash de solo lectura embebida
* u-boot revisa la GPT en la 16 GiB SSD eMMC y busca la particion FIRMWARE activa
* encontrando la particion activa trata de arrancar la u-boot alli
* u-boot revisa otra vez la GPT y busca el Linux kernel ROOT-KERNEL marcado
* encontrado Linux kernel arranca desde su particion raiz, mas no la raiz sera!

En la eMMC SSD de 16 GB originalmente hay dos de estas particiones:

* dos FIRMWARE una ROOT-A en particion 3 y otra ROOT-B en la 5
* dos KERNEL un KERNEL-A en particon 2 y otro KERNEL-B en particion 4

El par 2-3 es el que aparentemente arranca, puesto en la 2 esta la que tiene U-BOOT. **POR VERIFICAR ESTA INFO**

https://sites.google.com/a/chromium.org/dev/chromium-os/firmware-porting-guide/using-nv-u-boot-on-the-samsung-arm-chromebook

https://forums.kali.org/showthread.php?27350-Chromebook-expand-resize-grow-partition-rootfs

## Montar la raiz del sistema ChromeOS imagen para escritura rootfs

A esto le llaman "root filesystem read-writable", otra vez en la consola o en el 
vty con CRTL+ALT+"->" o en el "crosh" con CRTL+ALT+T sesion con chronos y sudo:

`/usr/share/vboot/bin/make_dev_ssd.sh --remove_rootfs_verification`

**ARM samsung chromebook 303** necesita especificar 2 o 4 puesto una es A y otra B:

`/usr/share/vboot/bin/make_dev_ssd.sh --remove_rootfs_verification --partitions 2`

y saldra esto, pues tiene dos pares de particiones con el sistema: 

```
make_dev_ssd.sh: INFO: Kernel A: Disabled rootfs verification.
make_dev_ssd.sh: INFO: Backup of Kernel A is stored in: /mnt/stateful_partition/backups/kernel_A_20180411_105134.bin
make_dev_ssd.sh: INFO: Kernel A: Re-signed with developer keys successfully.
make_dev_ssd.sh: INFO: Successfully re-signed 1 of 1 kernel(s)  on device /dev/mmcblk0.
```

ESto previo modo developer y lo de verificacion de arranque signed desactivado 
y con el arranque USB activado es para complementar.

**IMPORTANTE** devolverlo a estado anterior al terminar de poner linux como sea.

`sudo /usr/share/vboot/bin/make_dev_ssd.sh --noremove_rootfs_verification`

https://www.chromium.org/chromium-os/poking-around-your-chrome-os-device#TOC-Making-changes-to-the-filesystem

# 1. Chroot y Crouton

**Tener en cuenta que esto lo que hace es "sub"instalar debian** es decir 
ejecutarlo detras de el ChromeOS, valiendose de como si una virtual fuera.

Se deber tener ya listo:

* [Developer Mode y USB boot](#developer-mode-y-usb-boot)
* [Developer Mode y Verificacion](#developer-mode-y-verificacion)
* [Montar la raiz del systema chromeos imagen para escritura rootfs](#montar-la-raiz-del-sistema-chromeos-imagen-para-escritura-rootfs)

## USar un login no invitado

Lastimosamente debe tener una cuenta gmail para el primer inicio y usarla, 
**despues no necesita tener internet**.

### Mecanismo de inicio

El ChromeOS para la primera vez necesita uan cuenta google, una vez se inicia, 
el sistema configura un usuario interno, con la calve del correo, y despues 
usara esta clave para cada inicio de sesion corroborandola contra el correo, 
ojo simepre offline, porque al verse online si encuentra discrepancias no iniciara.

Esto es un problema mayor, dado depende de el correo casi al 100% y no se puede alterar.

## Download and install crouton

`curl -o /home/chronos/crouton https://raw.githubusercontent.com/dnschneid/crouton/master/installer/crouton;sh crouton`

Para instalar hay que ejecutarlo sin parametros, y este dira que descargara 
el script crouton, al terminar mostrara la ayuda.

Lastimosamente requiere internet, para no usar de nuevo internet, el deja todo 
lo de installer en el directorio `/tmp/crouton-installer-cache/` 

Pero como este descargara todo de internet mover su contenido solo tendra 
sentido despues de instalado el chroot.

## Instalaciones

Vamos ir dejando todo independiente de internet, primero mover el verdadero instalador:

`mv /tmp/crouton-installer-cache/crouton /home/chronos/crouton-installer`

Aqui la t es cosas como que escritorio y que "set" de paquetes la r es distro 
y n identificador de chroot:

`sh crouton-installer -t core,audio,x11,lxde-desktop -r wheeze -n venenux1`

**NOTA** no siempre funciona en las Samsung si algo va mal: `sudo delete-chroot venenux1`

Se utiliza "wheeze" por la compatibilidad entre el xorg y el binario Mali GPU.

Todo lo que hace crouton es al vuelo (descargado de internet), pero 
despues se podra respaldar el chroot para no depender de internet.

La instalacion sera en `/usr/local/chroots/venenux1` y en la consola empezara 
salir cosas de debian, como si de un debian se tratase. TEner en cuenta que esto 
lo que hace es "sub"instalar debian y ejecutarlo detras de el ChromeOS, 
valiendose de como si una virtual fuera.

https://github.com/dnschneid/crouton/wiki/Crouton-Command-Cheat-Sheet

## Iniciar el escritorio

create .xinitrc
exec mate-session

to start x (inside chroot): xinit
startx doesn't work

# Fuentes

https://groups.google.com/a/chromium.org/forum/#!topic/chromium-os-dev/ZcbP-33Smiw/discussion
* https://wiki.debian.org/InstallingDebianOn/Samsung/ARMChromebook
* https://www.neowin.net/forum/topic/1173005-replacing-chrome-os-with-debian-jessie-on-the-samsung-series-3-chromebook/
* https://blog.pgeiser.com/posts/2018/02/installing-debian-stretch-on-an-arm-chromebook-xe303c12/#
* https://github.com/dnschneid/crouton/wiki/Autostart-crouton-chroot-at-ChromeOS-startup
* https://wiki.galliumos.org/Hardware_Compatibility
