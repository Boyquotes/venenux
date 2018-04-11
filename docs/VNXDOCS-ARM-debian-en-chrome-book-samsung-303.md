## Introducción

El principal problema con ChromeOS es que todos los servicios de **google están perfectamente integradas**, 
si **DEMASIADO tanto que sin eso no se instala nada!** para rematar, cuadno su tiempo de soporte termina 
no les puedes instalar nada está alcanzando poco a poco el final de la vida (de julio de 2018, según google).

ESTOY EN CONTRA DE EL OBSOLESCENCIA asi que dado una persona del trabajo la necesitaba, y estamos en crisis aprendemos apreciar las cosas:

Yo traduje esto al español: https://blog.pgeiser.com/posts/2018/02/installing-debian-stretch-on-an-arm-chromebook-xe303c12/
La mejor fuente de información es de los chicos de Kali Linux. Ellos proporcionan el BRAZO de imágenes para un montón de diferentes sistemas.

Los scripts utilizados para generar estas imágenes ya envejecieron, y son inservibles, pero están disponibles en Github 
solo si navegas en el historial. Que proporcionan una excelente base para preparar una debian por su cuenta.

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

## Caracteristicas comunes:

Procesador: siempre un 2 nucleos @ 1.7Ghz (ARM o Intel)
Memoria: entre 2GB DDR3L y 8GB
Disco: 16GB eMMC SSD o 32GB eMMC SSD
Pantalla: siemrpe 11.6" / 1366x768 a 2024x1024
Red: WiFi 802.11 a/b/g/n (Dual Band)
Video: VGA (dongle) con HDMI siempre
USB: 1x USB 3.0 / 1x USB 2.0 en las nuevas USB-c
Tarjetas: SD/SDHC/SDXC
Sonido: JAck 2mm microfono/audifono mixto en las nuevas se eliminara

## Developer Mode y Verificacion

El primero para tomar el control de la Chromebook es cambiar a modo de desarrollador.
Este deshabilitar el núcleo de verificación de firmas y dejar ejecutar cualquier firmware 
de la máquina con el inconveniente de tener en cada inicio de una molesta pantalla de advertencia...
**IMPORTANTE** no pensar que la pantalla de advertencia es tener que reinstalar el chrome OS!

1. En primer lugar, apague la máquina
2. Despues deje presionado las teclas EXC+REFRESH y despues sin despresionar encender 
con la tecla POWER, es decir las tres al mismo tiempo pero la de power de ultimo. 
Esto sacara una imagen con un signo de exclamacion grande..
3. En la pantalla de signo de exclamacion sale diciendo que debe recuperar el chrome OS 
y algo de USB presione CTRL+D al mismo tiempo, y .... rapidamente...
4. Sale pantalla en blanco con letras, diciendo algo de "Verification OFF .. enter" pulse ENTER 
es decir confirme pulsando enter, que se apagara rapidamente y espere re_encienda....
5. Al volver encender aparece una pantalla de advertencia "OS Verification OFF" y 
despues de 30 segundos se escuchara dos pitidos, y a lo cual volvera reinicar ...
y justo despues de este reinicio empieza un conteo arriba y una barra avanza 
esto durara de 10 a 15 minutos y borrara toda la data de su sesion (por favor "su", sera de google)
6. Cuendo termine reiniciara nuevamente, y en cada reinicio mostrara la pantalla "OS Verification OFF" 
esta pantalla se quita a los 30 segundos o presionando CRTL+D sobre ella

## Developer Mode y USB BOOT

Una vez que el desarrollador modo está habilitado, es posible habilitar el arranque por USB:

7. Como la data se ha borrado, para poder trabajar despues que esta pantalla 
de "OS Verification OFF" arranque, dejela pasar Y NO PRESIONE SPACE
8. Cuando muestre el logo de "Chrome" de colores y pro fin carge de nuevo 
alli es cuando pulsara CRTL+ALT+"->" o sino CTRL+ALT+F2 al mismo tiempo y 
esto mostrara uan consola de Linux bash para hacer login.
9. escriba "chronos" como nombre de usuario, esto lo iniciara login de consola
10. escriba "sudo su" esto le dara acceso de root, windoseros ignorantes no seguir aqui..
11. ejecutar el comando `crossystem dev_boot_usb=1 dev_boot_signed_only=0`

## Error: mensaje insertar USB y nunca arranca

Estas porquerias de google son Linux y si pulsas TAB en el inicio mostrara el error del kernel

1. si **revcovery_reason="0X54"** reintentar prender al menos 40 veces hasta que arranque.
2. si no, presiona CRTL+D en la pantalla y si vuelve preguntar "To turn OS Verification OFF", enter

De aqui en adelante igual que el paso numero 5 (cinco).

## Montar la raiz del systema ChromeOS iamgern para escritura rootfs

A esto le llaman 

# Fuentes

https://wiki.debian.org/InstallingDebianOn/Samsung/ARMChromebook
https://www.neowin.net/forum/topic/1173005-replacing-chrome-os-with-debian-jessie-on-the-samsung-series-3-chromebook/
https://blog.pgeiser.com/posts/2018/02/installing-debian-stretch-on-an-arm-chromebook-xe303c12/#
https://github.com/dnschneid/crouton/wiki/Autostart-crouton-chroot-at-ChromeOS-startup
https://wiki.galliumos.org/Hardware_Compatibility
