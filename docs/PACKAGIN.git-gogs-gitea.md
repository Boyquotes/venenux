
Gogs y gitea : packaging
===========================

El trabajo de empaquetamoento esta en lso siguiente ITP/RTP de debian:

* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=792101 : para gitea
* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=780606 : para gogs

En un inicio mucha confusion creyendo que gitea era clonico, lo es solo emprincipio, 
pero su forma de manejo de desarrollo es muy distinta, cambia mucho en el tiempo.. 
ademas gitea pretende ser distinto y separado de gogs, llenandolo de nuevas caracteristicas
mientras gogs pretende manetener simple y con solo lo necesario.

**GOGS DEPENDENCY TREE (unfinished and unrefined)**

golang-gopkg-gomail.v2-dev
        Not Checked
golang-gopkg-ldap.v2-dev
        Not Checked
golang-gopkg-editorconfig-editorconfig-core-go.v1-dev
        Not Checked
golang-github-gogits-chardet-dev
        golang-github-saintfish-chardet-dev
golang-github-unknwon-paginater-dev
golang-github-gogits-cron-dev
golang-github-unknwon-cae-dev
        [[ CYCLICAL: golang-github-unknwon-cae-dev ]]
golang-github-sergi-go-diff-dev
        golang-github-stretchrcom-testify-dev
golang-github-issue9-identicon-dev
        golang-github-issue9-assert-dev
golang-github-microcosm-cc-bluemonday-dev
golang-github-nfnt-resize-dev
golang-github-strk-go-libravatar-dev
[[ dup: golang-github-unknwon-i18n-dev ]]
golang-github-go-macaron-csrf-dev
golang-github-go-macaron-toolbox-dev
golang-github-gogits-go-gogs-client-dev
golang-github-go-macaron-i18n-dev
        golang-github-unknwon-i18n-dev
golang-github-gogits-git-module-dev
        [[ dup: golang-github-mcuadros-go-version-dev ]]
golang-github-mcuadros-go-version-dev
golang-github-go-macaron-captcha-dev
        golang-github-go-macaron-cache-dev
                [[ dup: golang-github-siddontang-ledisdb-dev ]]
                golang-github-lunny-nodb-dev
                        [[ dup: golang-github-lunny-log-dev ]]
                        [[ dup: golang-github-syndtr-goleveldb-dev ]]
                        [[ dup: golang-github-siddontang-go-snappy-dev ]]
golang-github-jaytaylor-html2text-dev
golang-github-go-macaron-cache-dev
        golang-github-siddontang-ledisdb-dev
                golang-github-ugorji-go-dev
                [[ dup: golang-github-syndtr-goleveldb-dev ]]
                golang-github-siddontang-go-dev
                golang-github-siddontang-rdb-dev
                        golang-github-cupcake-rdb-dev
                                ???
(this one wants something from launchpad.net)
                golang-github-edsrzf-mmap-go-dev
                golang-github-siddontang-goredis-dev
                        golang-github-alicebob-miniredis-dev
                                golang-github-bsm-redeo-dev
                                        [[ dup: golang-github-onsi-ginkgo-dev ]]
                                        [[ dup: golang-github-onsi-gomega-dev ]]
        golang-github-lunny-nodb-dev
                golang-github-lunny-log-dev
                golang-github-siddontang-go-snappy-dev
                golang-github-syndtr-goleveldb-dev
                        golang-github-onsi-ginkgo-dev
                                [[ CIRCULAR: golang-github-onsi-gomega-dev ]]
                        golang-github-onsi-gomega-dev
                                [[ CIRCULAR: golang-github-onsi-ginkgo-dev ]]
                                golang-github-golang-protobuf-dev
**DONE gogs dependency tree**

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

