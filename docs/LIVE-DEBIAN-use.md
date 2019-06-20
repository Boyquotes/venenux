# Debian LiveCd Recuperaciones

El proposito es que remotamente un usuario sin conocimiento en linux inicie el disco 
y proporcione acceso rapido a un administrador remoto.. una vez que acceda restaura limitantes y seguridad.

Debian live no proporciona acceso simple ni facil a el computador por seguridad! 
y para un usuario ignorante windosero le seria dificil proporcionar el acceso.

## Tabla de Contenidos

* [Versiones e inicio]
* [2. Conocer la direccion ip](#2-conocer-la-ip)
* [3. clave de root](#3-clave-de-root)
* [4. ssh libre](#4-ssh-libre)


Esta página describe los métodos para usar el Debian Live CD original anteriores 
(lenny, etch, squeeze, wheezy, jessie) para recuperarse de diferentes tipos de problemas.
Usamos los anteriores porque muchas maquinas necesitaran versiones especificas.


| Version | CD? DVD? USB? | PC soportadas | Enlace Descarga |
| ------- | ------ |-------- | ------- |
| 5.0 lenny | solo CD/DVD | viejas 2006-2010 | https://cdimage.debian.org/images/archive/5.0.10-live/i386/iso-cd/debian-live-5010-i386-lxde-desktop.iso |
| 6.0 squeeze | CD/DVD/USB | viejas 2009-2013 | https://cdimage.debian.org/images/archive/6.0.10-live/i386/iso-hybrid/debian-live-6.0.10-i386-lxde-desktop.iso |
| 7.0 wheeze | DVD/USB | 2012-2016 | https://cdimage.debian.org/images/archive/7.11.0-live/i386/iso-hybrid/debian-live-7.11.0-i386-lxde-desktop.iso |
| 8.0 jessie | DVD/USB | 2015-2019 | https://cdimage.debian.org/images/archive/8.11.0-live/i386/iso-hybrid/debian-live-8.11.0-i386-lxde-desktop.iso |


## 1. iniciar en CD

Hay que colocar el CD o DVD en la computadora y decirle a esta que arranque desde 
el cdrom o dvdrom o unidad optica, aqui varios ejemplos

DELL: desktops usar tecla F12 y seleccionar la unidad DVD/CD
CLONES: usar F8 o F1 aqui depende el caso, sino configurar en el bios

## 2. conocer la ip

1. El CD/DVD/USB inicia y una vez terminado muestra un escritorio, abajo la barra de tareas y en la esquina menu
2. ir al menu y en Accesorios (Utility en inglesh) ejecutar terminal, sino en System, Xterm o Terminal de root
3. al abrir se muestra una ventana negra, aqui ejecutar `ip addr` y sldran varias lineas
4. la linea que tiene "lo" no es la ip de la maquina sino la conocida "127.0.0.1"
5. la lineas restantes tienen la ip, descarte la que tenga "inet6" y solo use las que tenga "inet"


## 3. clave de root

1. El CD/DVD/USB inicia y una vez terminado muestra un escritorio, abajo la barra de tareas y en la esquina menu
2. ir al menu y en Accesorios (Utility en inglesh) ejecutar terminal, sino en System, Xterm o Terminal de root
3. al abrir se muestra una ventana negra, aqui ejecutar `sudo su` y entrara como root sin pedir clave (Debian)
4. el simbolo `$` cambiara a `#` y ahora ejecute el comando `passwd root` el cual preguntara por nueva clave
5. **ADVERTENCIA** ***no se mostrara lo que escribe, coloque como clave "root" y pulse enter***
6. ***Una vez mas preguntara para confirmar clave, vuelva escribir "root" y pulse enter***
7. Si todo sale bien dira "passowrd sucesfully updated" sino repita el proceso completo desde el menu

## 4. ssh libre

1. Debe haber colocado la "clave de root"
2. ir al menu y en Accesorios (Utility en inglesh) ejecutar terminal, sino en System, Xterm o Terminal de root
3. al abrir se muestra una ventana negra, aqui ejecutar `sudo su` y entrara como root sin pedir clave (Debian)
5. el simbolo `$` cambiara a `#` y ahora ejecute el comando `service ssh restart` y espere
6. **EJECUTAR ESTOS COMANDOS SIN ERRORES** si no puede "editar el archivo" es ejecutando `nano /etc/ssh/sshd_config`, debe cambiar lo siguiente:
7. `sed -i -r 's#PermitRootLogin.*#PermitRootLogin yes#g' /etc/ssh/sshd_config` 
8. `sed -i -r 's#RSAAuthentication.*#RSAAuthentication no#g' /etc/ssh/sshd_config` 
9. `sed -i -r 's#PubkeyAuthentication.*#PubkeyAuthentication no#g' /etc/ssh/sshd_config`
10. reiniciar el servicio con el comando `service ssh restart`
6. Si todo sale bien dira "service ssh restarted" sino saldr uan palabra failed .. en este caso repita todo

Metodo alternativo. Si no puede escribir los comandos correctamente, intente de esta manera:

* Abrir y editar el archivo asi `nano /etc/ssh/sshd_config`, 
* buscar y debe cambiar lo siguiente `PermitRootLogin` en `yes`
* buscar ahora y cambiar lo siguiente y poner `RSAAuthentication` en `no`
* buscar ahora y cambiar lo siguiente y poner `PubkeyAuthentication` en `no`
* Presionar despues de todo las teclas `CTRL` (control) y al mismo tiempo la tecla `X` juntas
* Realizara uan pregunta en ingles, escriba "Y"  sin las comillas y pulse enter
* Realizara una pregunta sobre nombre del archivo, pulse enter tal cual como esta sin cambiarlo
* despues seguir con el paso 10

