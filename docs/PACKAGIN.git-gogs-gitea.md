
Gogs y gitea : packaging
===========================

El trabajo de empaquetamoento esta en los siguiente ITP/RTP de debian:

* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=792101 : para gitea
* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=780606 : para gogs

En un inicio mucha confusion creyendo que gitea era clonico, lo es solo en principio, 
pero su forma de manejo de desarrollo es muy distinta, cambia mucho en el tiempo.. 
ademas gitea pretende ser distinto y separado de gogs, llenandolo de nuevas caracteristicas
mientras gogs pretende manetener simple y con solo lo necesario.

**el paqeute de venenux maneja solo las dependencias actuales, 
a medida aparece empaquetada una, se sustituye y maneja en el paqeute gogs/gitea**

Dependencias de gogs/gitea y subdependencias en Debian
--------------------------------------

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

### Dependency Progress de cada paquete en Debian

Import  					InDeb	DebPkgName
==						==	==

autolink				   	IP	libjs-autolink
dropzone					IP	libjs-dropzone
semantic					IP	libjs-semantic
vue						IP	libjs-vue
swagger-ui					IP	libjs-swaggerui


github.com/go-gitea/gitea			^HOLD	gitea


--	[ fonts ]				--	--
octicons					Y	fonts-octicons
font-awesome					Y	fonts-font-awesome

--	[ js ]					--	--
jquery.areyousure				PEND	libjs-jquery-areyousure
gitgraph					PEND	libjs-jquery-gitgraph
simplemde					PEND	libjs-simplemde
cssrelpreload					PEND	libjs-cssrelpreload
jquery.minicolors				Y	libjs-jquery-minicolor
pdfjs						Y	libjs-pdf / pdf.js-common
emojify						Y	libjs-emojify
codemirror					Y	libjs-codemirror
jquery.datetimepicker				Y	libjs-jquery-timepicker
clipboard					Y	libjs-clipboard
highlight					Y	libjs-highlight.js
jquery						Y	libjs-jquery

--	[ go ]					--	--
github.com/siddontang/ledisdb			Y	golang-github-siddontang-ledisdb-dev
github.com/glendc/gopher-json			Y	golang-github-glendc-gopher-json-dev
github.com/jaytaylor/html2text			Y	golang-github-jaytaylor-html2text-dev
github.com/ssor/bom				Y	golang-github-ssor-bom-dev
github.com/yuin/gopher-lua			Y	golang-github-yuin-gopher-lua-dev
github.com/bsm/redeo				Y	golang-github-bsm-redeo-dev
github.com/bsm/pool				Y	golang-github-bsm-pool
github.com/go-macaron/captcha			Y	golang-github-go-macaron-captcha-dev
github.com/go-macaron/cache			Y	golang-github-go-macaron-cache-dev
github.com/gogits/chardet			Y	golang-github-gogits-chardet-dev
github.com/siddontang/ledisdb			Y	golang-github-siddontang-ledisdb-dev
github.com/siddontang/rdb			Y	golang-github-siddontang-rdb-dev
github.com/markbates/goth			Y	golang-github-markbates-goth-dev
github.com/gopherjs/gopherjs			Y	golang-github-gopherjs-gopherjs-dev
github.com/lunny/nodb				Y	golang-github-lunny-nodb-dev
github.com/go-macaron/i18n			Y	golang-github-go-macaron-i18n-dev
github.com/kardianos/osext			Y	golang-github-kardianos-osext-dev
github.com/Unknwon/i18n				Y	golang-github-unknwon-i18n-dev
github.com/Unknwon/paginater			Y	golang-github-unknwon-paginater-dev
github.com/Unknwon/cae				Y	golang-github-unknwon-cae-dev
github.com/siddontang/goredis			Y	golang-github-siddontang-goredis-dev
gopkg.in/testfixtures.v2			Y	golang-gopkg-testfixtures.v2-dev
github.com/siddontang/go-snappy			Y	golang-github-siddontang-go-snappy-dev
golang.org/x/sync				Y	golang-golang-x-sync-dev
github.com/syndtr/goleveldb			Y	golang-github-syndtr-goleveldb-dev
github.com/neelance/astrewrite			Y	golang-github-neelance-astrewrite-dev
github.com/gorilla/sessions			Y	golang-github-gorilla-sessions-dev
gopkg.in/jarcoal/httpmock.v1			Y	golang-gopkg-jarcoal-httpmock.v1-dev
github.com/alicebob/miniredis			Y	golang-github-alicebob-miniredis-dev
github.com/blevesearch/bleve			Y	golang-github-blevesearch-bleve-dev
gopkg.in/alexcesaro/quotedprintable.v3		Y	golang-gopkg-alexcesaro-quotedprintable.v3-dev
gopkg.in/editorconfig/editorconfig-core-go.v1	Y	golang-gopkg-editorconfig-editorconfig-core-go.v1-dev
github.com/joho/godotenv			Y	golang-github-joho-godotenv-dev
github.com/gorilla/pat				Y	golang-github-gorilla-pat-dev
github.com/cupcake/rdb				Y	golang-github-cupcake-rdb-dev
github.com/gorilla/securecookie			Y	golang-github-gorilla-securecookie-dev
github.com/lunny/log				Y	golang-github-lunny-log-dev
github.com/kisielk/gotool			Y	golang-github-kisielk-gotool-dev
github.com/mrjones/oauth			Y	golang-github-mrjones-oauth-dev
github.com/nbutton23/zxcvbn-go			Y	golang-github-nbutton23-zxcvbn-go-dev
github.com/neelance/sourcemap			Y	golang-github-neelance-sourcemap-dev
github.com/siddontang/go			Y	golang-github-siddontang-go-dev
github.com/yohcop/openid-go			Y	golang-github-yohcop-openid-go-dev
code.gitea.io/sdk/gitea				Y	golang-code.gitea-sdk-dev
github.com/facebookgo/grace			Y	golang-github-facebookgo-grace-dev
github.com/spf13/cobra				Y	golang-github-spf13-cobra-dev
github.com/facebookgo/httpdown			Y	golang-github-facebookgo-httpdown-dev
code.gitea.io/git				Y	golang-code.gitea-git-dev
github.com/willf/bitset				Y	golang-github-willf-bitset-dev
golang-github-seiflotfy-cuckoofilter		Y	golang-github-seiflotfy-cuckoofilter-dev
golang-github-couchbase-moss			Y	golang-github-couchbase-moss-dev
github.com/petar/GoLLRB				Y	golang-github-petar-gollrb-dev
strk.kbt.io/projects/go/libravatar		Y	golang-strk.kbt-projects-go-libravatar-dev
github.com/pquerna/otp				Y	golang-github-pquerna-otp-dev
github.com/go-macaron/csrf			Y	golang-github-go-macaron-csrf-dev
github.com/go-xorm/builder			Y	golang-github-go-xorm-builder-dev
github.com/mcuadros/go-version			Y	golang-github-mcuadros-go-version-dev
github.com/twinj/uuid				Y	golang-github-twinj-uuid-dev
gopkg.in/gomail.v2				Y	golang-gopkg-gomail.v2-dev
github.com/leemcloughlin/gofarmhash		Y	golang-github-leemcloughlin-gofarmhash-dev
github.com/edsrzf/mmap-go			Y	golang-github-edsrzf-mmap-go
github.com/facebookgo/stats			Y	golang-github-facebookgo-stats-dev
github.com/msteinert/pam			Y	golang-github-msteinert-pam-dev
github.com/pingcap/check			Y	golang-github-pingcap-check-dev
github.com/go-macaron/bindata			Y	golang-github-go-macaron-bindata-dev
github.com/go-macaron/toolbox			Y	golang-github-go-macaron-toolbox-dev
github.com/shurcooL/sanitized_anchor_name	Y	golang-github-shurcool-sanitized-anchor-name-dev
github.com/boombuler/barcode			Y	golang-barcode-dev
github.com/coreos/go-etcd			Y	golang-etcd-dev
github.com/coreos/etcd				Y	golang-etcd-server-dev
github.com/blevesearch/go-porterstemmer		Y	golang-github-blevesearch-go-porterstemmer-dev
github.com/blevesearch/segment			Y	golang-github-blevesearch-segment-dev
github.com/onsi/ginkgo				Y	golang-ginkgo-dev
github.com/onsi/gomega				Y	golang-gomega-dev
github.com/boltdb/bolt				Y	golang-github-boltdb-bolt-dev
github.com/bradfitz/gomemcache			Y	golang-github-bradfitz-gomemcache-dev
github.com/davecgh/go-spew			Y	golang-github-davecgh-go-spew-dev
github.com/denisenkom/go-mssqldb		Y	golang-github-denisenkom-go-mssqldb-dev
github.com/dgrijalva/jwt-go			Y	golang-github-dgrijalva-jwt-go-v3-dev
github.com/elazarl/go-bindata-assetfs		Y	golang-github-elazarl-go-bindata-assetfs-dev
github.com/facebookgo/clock			Y	golang-github-facebookgo-clock-dev
github.com/gogits/cron				Y	golang-github-gogits-cron-dev
github.com/golang/snappy			Y	golang-github-golang-snappy-dev
github.com/go-macaron/binding			Y	golang-github-go-macaron-binding-dev
github.com/go-macaron/gzip			Y	golang-github-go-macaron-gzip-dev
github.com/go-macaron/inject			Y	golang-github-go-macaron-inject-dev
github.com/go-macaron/session			Y	golang-github-go-macaron-session-dev
github.com/go-sql-driver/mysql			Y	golang-github-go-sql-driver-mysql-dev
github.com/go-xorm/core				Y	golang-github-go-xorm-core-dev
github.com/go-xorm/xorm				Y	golang-github-go-xorm-xorm-dev
github.com/issue9/identicon			Y	golang-github-issue9-identicon-dev
github.com/jtolds/gls				Y	golang-github-jtolds-gls-dev
github.com/juju/errors				Y	golang-github-juju-errors-dev
github.com/klauspost/compress			Y	golang-github-klauspost-compress-dev
github.com/klauspost/cpuid			Y	golang-github-klauspost-cpuid-dev
github.com/klauspost/crc32			Y	golang-github-klauspost-crc32-dev
github.com/lib/pq				Y	golang-github-lib-pq-dev
github.com/mattn/go-sqlite3			Y	golang-github-mattn-go-sqlite3-dev
github.com/microcosm-cc/bluemonday		Y	golang-github-microcosm-cc-bluemonday-dev
github.com/nfnt/resize				Y	golang-github-nfnt-resize-dev
github.com/ngaut/deadline			Y	golang-github-ngaut-deadline-dev
github.com/ngaut/go-zookeeper			Y	golang-github-ngaut-go-zookeeper-dev
github.com/ngaut/log				Y	golang-github-ngaut-log-dev
github.com/ngaut/pools				Y	golang-github-ngaut-pools-dev
github.com/golang/protobuf			Y	golang-goprotobuf-dev
github.com/ngaut/sync2				Y	golang-github-ngaut-sync2-dev
github.com/pmezard/go-difflib			Y	golang-github-pmezard-go-difflib-dev
github.com/russross/blackfriday			Y	golang-github-russross-blackfriday-dev
github.com/satori/go.uuid			Y	golang-github-satori-go.uuid-dev
github.com/sergi/go-diff			Y	golang-github-sergi-go-diff-dev
github.com/smartystreets/assertions		Y	golang-github-smartystreets-assertions-dev
github.com/smartystreets/goconvey		Y	golang-github-smartystreets-goconvey-dev
github.com/steveyen/gtreap			Y	golang-github-steveyen-gtreap-dev
github.com/stretchr/testify			Y	golang-github-stretchr-testify-dev
github.com/urfave/cli				Y	golang-github-urfave-cli-dev
github.com/ugorji/go				Y	golang-ugorji-go-dev
golang.org/x/crypto				Y	golang-golang-x-crypto-dev
golang.org/x/net				Y	golang-golang-x-net-dev
golang.org/x/sys				Y	golang-golang-x-sys-dev
golang.org/x/text				Y	golang-golang-x-text-dev
gopkg.in/asn1-ber.v1				Y	golang-gopkg-asn1-ber.v1-dev
gopkg.in/bufio.v1				Y	golang-gopkg-bufio.v1-dev
gopkg.in/ini.v1					Y	golang-gopkg-ini.v1-dev
gopkg.in/yaml.v2				Y	golang-goyaml-dev
gopkg.in/macaron.v1				Y	golang-gopkg-macaron.v1-dev
gopkg.in/ldap.v2				Y	golang-github-go-ldap-ldap-dev
gopkg.in/redis.v2				Y	golang-gopkg-redis.v2-dev

Fuentes y como manejado de gitea/gogs
-------------------

Similares.. gitea ramifica lo que va en data en un directorio extra optional

Directorio debian y proceso de empaquetado
==========================================

Se emplea las reglas de http://pkg-go.alioth.debian.org/packaging.html#_binary_only_packages
so the resulting package are named gitea, its a binary only and the version number are correctly.

Compilacion basada en https://github.com/Martchus/PKGBUILDs/tree/master/gogs/default

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

