# Equivalencias entre Debian y Winbuntus para uso de repos o ppas

Todos sabemos que a veces necesitamos sacar recursos de otros lados, porque ya se saben que 
**no siempre sera los ignorantes windosers los que se viven siempre nutriendo de linux**, 
cabe destacar como lso windosers hoy dia emplean mas proyectos linux dentro de windos que nunca..

Para usar un repositorio PPA o un repositorio de una mala imitacion de debian se necesita saber 
que version se usara de PPA, aqui las equivalecias cuando se emplee.

Los porcentajes ayudan a saber que tanto puede emplearse o incluso backportar un paquete entre releases.

|debian |numero |mint   | numero|wibuntu|numero |compat |compat |compat |compat |compat |compat |
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|       |       |       |       |       |deb ver|     9 |     8 |     7 |     6 |     5 |     4 |
|       |       |       |       |       |deb nam|stretch| jessie| Wheezy|Squeeze| lenny |  etch |
|deb nam|deb ver|mint nam|int ve|ppa nam|ppa ver|%      |%      |%      |%      |%      |%      |
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|sid    |    10 |rosa   |  18   |bionic |18.04  |20     |1      | 0     | 0     | 0     |       |
|stretch|     9 |rosa   |  18   |artful |17.10  |84     |1      | 0     | 0     | 0     |       |
|stretch|     9 |rosa   |  18   |zesty  |17.04  |92     |2      | 0     | 0     | 0     |       |
|stretch|     9 |rosa   |  18   |yakkety|16.10  |70     |10     | 0     | 0     | 0     |       |
|stretch|     9 |rosa   |  18   |xenial |16.04  |20     |30     | 0     | 0     | 0     |  -    |
|jessie |     8 |rosa   |  18   |wily   |15.10  |10     |78     |10     | 0     | 0     | 0     |
|jessie |     8 |rafaela|  17.2 |vivid  |15.04  |10     |89     |20     | 1     | 0     | 0     |
|jessie |     8 |rebecca|  17.1 |utopic |14.10  | 1     |96     |40     | 5     | 0     | 0     |
|jessie |     8 |quiana |  17   |trusty |14.04  | 1     |82     |25     |10     | 0     | 0     |
|Wheezy |     7 |petra  |  16   |saucy  |13.10  | 1     |42     |90     |10     | 0     | 0     |
|Wheezy |     7 |oliva  |  15   |raring |13.04  | 1     |45     |89     |14     | 0     | 0     |
|Wheezy |     7 |nadya  |  14   |quantal|12.10  | 1     |35     |70     |18     | 0     | 0     |
|Wheezy |     7 |maya   |  13   |precise|12.04  | 1     |25     |70     |20     | 0     | 0     |
|wheezy |     7 |lisa   |  12   |oneiric|11.10  | 0     |20     |60     |30     | 0     | 0     |
|Squeeze|     6 |katia  |  11   |natty  |11.04  | 0     |10     |40     |79     | 0     | 0     |
|Squeeze|     6 |julia  |  10   |maveric|10.10  | 0     | 5     |25     |96     | 0     | 0     |
|Squeeze|     6 |isadora|   9   |lucid  |10.04  | 0     | 5     |10     |89     | 0     | 0     |
|Squeeze|     6 |helena |   8   |karmic |9.10   | 0     | 1     |1      |34     |30     | 0     |
|lenny  |     5 |gloria |   7   |jaunty |9.04   | 0     | 1     |1      |11     |79     | 0     |
|lenny  |     5 |felisia|   6   |intrepi|8.10   | 0     | 0     |1      |11     |96     | 1     |
|lenny  |     5 |elyssa |   5   |hardy  |8.04   | 0     | 0     |1      |10     |82     | 5     |
|etch   |     4 |daryna |   4   |gutsy  |7.10   | 0     | 0     |1      | 8     |30     |82     |
|etch   |     4 |daryna |   4   |dapper |7.04   | 0     | 0     |0      | 1     |10     |98     |

Segun las fechas winbuntu, las salidas de LTS que coinciden con Debian no son paquetes coincidentes:

* para cuando Strecth Debian 9, esta artful 17.10
* para cuando Jessie Debian 8, esta wily 15.10
* para cuando Wheeze Debian 7, esta saucy 13.10
* para cuando Strecth Debian 6, esta maverick 10.10
* cercana solamente Debian 5, esta intrepid o hardy
* para cuando Etch Debian 4, esta dapper 7.04 exacta


