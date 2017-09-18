
Fuentes gitea/gogs
-------------------

Similares.. gitea ramifica lo que va en data en un directorio extra optional

 debian/control
-----------------

#### Build depends:

* golang: gogs requiere de 1.6 minimo y gitea de 1.7 minimo, desde jeesie backport ofrece 1.7, go solo depende de libc6 2.3.6
* rsync: se empleo rsync en vez de usar solo cp, era mas facil para excluir directorio debian posible de upstream
* libpq-dev : para que tenga soporte postgresql, raro que no pide lo mismo para mysql/mariadb
* libsqlite3-dev: para que tenga soporte sqlite3 en su db interna, raro no pide lo mismo para mysql/mariadb
* libpam0g-dev: para que pueda autenticar usuarios git/web con pam, nota: implica gitea/gogs lea /etc/shadow
* lsb-release: para detectar si es Devuan, Debian o Venenux y empaquetar soporte systemd o no.

#### Depends:

* init-system-helpers |  file-rc | sysv-rc: porque emplea uso de update-rc.d en postinst y prerm
* git: porque emplea el comando git para iniciar y configurar repositorios
* adduser: porque configura un usuario para el mismo sistema (no corre como root) en postinst y prerm
* cron | anacron : en si para monitorear (git es descentralizado, no hay servidor central con eventos que consultar)
* sqlite3 | postgresql | mysql-server | mariadb-server: sqlite3 primero para configuracion automatica

#### recomienda:

* mta: para notificaciones pro correo, desactivadas por defecto

#### sugiere:

* httpd, lighty, apache2, nginx: en dicho orden ya que apache2 pide mucho y es dificil colocar seguridad

debian/rules:
--------------

#### parches:


Parches:
---------

No se parchea directo las fuentes, si se alterazen, el empaquetado fallaria, para facilidad se prefiere 
un empaquetado mas directo y menos dependiente de las fuentes, por ende se parchea usando sed.. 

Es decir no se agrega ni quita nada, solo se cambia lo existente, los cambios se listan aqui:

