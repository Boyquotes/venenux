## Guía basica de lua

en esta guía aprenderemos lo basico de lua.

* [Acerca de Lua](#acerca-de-lua)
  * [Para que sirve](#para-que-sirve)
  * [Instalación](#instalacion)
    * [Multiples versiones](#multiples-versiones)
    * [JIT optimizacion al vuelo](#jit-optimizacion-al-vuelo)
* [Entorno](#entorno)
  * [Guia rapida para impacientes](#guia-rapida-para-impacientes)
  * [Guia mas detallada pero larga](#guia-mas-detallada-pero-larga)
  * Comentarios
  * Variables
  * Tipos de datos
  * Operadores
   * Operadores aritméticos
   * Operadores logicos
   * Operadores relacionales
  * Estructuras de control
   * Condicionales
   * Bucles
  * Funciónes
* Contactos

## Acerca de Lua

Lua **es igual que Python lastimosamente, cada componente se compila contra la version especifica** 
aunque una gran diferencia **entre versiones medias, Lua es compatible en gran medida, 
por ejemplo un codigo para Lua 5.1 puede ejecute sin cambios en Lua 5.3 casi siempre.**

Lua es lenguaje scripting, es decir es mayormente interpretado, como Java y Python, 
recientemente esta la famosa Luajit que ayuda en gran medida, pero Lua es flojo 
para calculos numericos, mismo problema que tiene Java y Gambas...

### Para que sirve

**No sirve para aplicaciones de escritorio ni para aplicaciones de señalizacion**, ejemplo 
no se recomienda implementar un enrutador como Kamailio en Lua, debido a el gran 
calculo matematico necesario carente y la **falta de soporte por hilos.**

**Se recomienda para soluciones puntuales disgregadas, servicios web**, y manejo de 
archivos, sockets y comunicacion menor entre cliente servidor, **un gran ejemplo es 
el sistema de chat xmpp "prosody"** fabricado en Lua.

## Instalación

Actualmente se **require Lua 5.1 como minimo, esta extendido Lua 5.2 y se prefiere Lua 5.3**, 

* En Debian gracias a la alta calidad **(a diferencia de otras distros) podemos tener 
varias versiones de Lua para desarrollar con alta compatibilidad**, usando el sistema 
de "alternatives"
* En debian Lua como tal se presenta en la rama 5 desde 5.0 (Debian etch), 
hasta la version 5.3, pero solo la version 5.1 se asume en todos (Debian strech, jessie) 
* En Debian se asume 5.1 como la version por defecto en el sistema, por ende **cada vez 
que instale un paquete desde el repositorio el entorno al cual se adhiere es Lua 5.1** y 
si lo combina con algun software instalado externamente de Lua no sera muy compatible.

`apt-get install lua50 lua5.1 lua5.2 lua5.3`

### Multiples versiones

A diferencia de otras distros podemos tener controlado y con alta compatibilidad 
muchos lenguajes, entre ellos `gcc4` junto con `gcc8` y `lua5.2` junto con `lua5.3`.

Para escoger entre que version de lua se ejecutara se emplea el sistema de alternativas, 
que nos permite lo que casi ninguna distro ofrece. Esto es ejecutando el comando 
`update-alternatives --config lua-interpreter` para el interprete `lua` y el comando 
`update-alternatives --config lua-compiler` para el compialdor `luac` que aputara 
el comando hacia la version sea `lua5.1`, `lua5.2`, `lua50` como se muestra a continuacion:

```
root@venenux-massenkoh# ls -l /usr/bin/lua*
lrwxrwxrwx 1 root root      33 abr  8 09:07 /usr/bin/lua -> /etc/alternatives/lua-interpreter
-rwxr-xr-x 1 root root   12688 ago 13  2013 /usr/bin/lua50
-rwxr-xr-x 1 root root  190252 oct 14  2014 /usr/bin/lua5.1
-rwxr-xr-x 1 root root  214808 oct 14  2014 /usr/bin/lua5.2
lrwxrwxrwx 1 root root      30 abr  8 09:07 /usr/bin/luac -> /etc/alternatives/lua-compiler
-rwxr-xr-x 1 root root    9940 ago 13  2013 /usr/bin/luac50
-rwxr-xr-x 1 root root  119184 oct 14  2014 /usr/bin/luac5.1
-rwxr-xr-x 1 root root  136784 oct 14  2014 /usr/bin/luac5.2
-rwxr-xr-x 1 root root 7009924 sep 21  2018 /usr/bin/luajittex
lrwxrwxrwx 1 root root       6 mar  7  2017 /usr/bin/lualatex -> luatex
-rwxr-xr-x 1 root root 6768260 sep 21  2018 /usr/bin/luatex
root@venenux-massenkoh# update-alternatives --config lua-interpreter
Existen 3 opciones para la alternativa lua-interpreter (que provee /usr/bin/lua).

  Selección   Ruta             Prioridad  Estado
------------------------------------------------------------
* 0            /usr/bin/lua5.2   120       modo automático
  1            /usr/bin/lua5.1   110       modo manual
  2            /usr/bin/lua5.2   120       modo manual
  3            /usr/bin/lua50    80        modo manual

Pulse <Intro> para mantener el valor por omisión [*] o pulse un número de selección: 0
root@venenux-massenkoh# update-alternatives --config lua-compiler
Existen 3 opciones para la alternativa lua-compiler (que provee /usr/bin/luac).

  Selección   Ruta              Prioridad  Estado
------------------------------------------------------------
* 0            /usr/bin/luac5.2   120       modo automático
  1            /usr/bin/luac5.1   110       modo manual
  2            /usr/bin/luac5.2   120       modo manual
  3            /usr/bin/luac50    80        modo manual

Pulse <Intro> para mantener el valor por omisión [*] o pulse un número de selección: 0
```

### JIT optimizacion al vuelo

En debian tenemos disponible desde Debian jessie o Debian 8 el 
**compialdor en tiempo de ejecucion**, esto significa que los scripts 
seran compilados a codigo maquina optimizados al momento de ejecutar, 
y no solo eso, **podran entonces ser enlazados con programas compilados en C/C++.**

Para VenenuX y Debian etch, lenny, squeeze y wheeze esta disponible 
en los repositorios de VenenuX. 

`apt-get install luajit`

Tal como se menciono **luajit asume un entorno Lua 5.1** debido a la misma razon 
que se expuso arriba, **al igual que python, entre versiones de Lua 
hay sumas incompatibilidades**.

# Entorno

Tenemos tres tipos de entornos, y como se menciono es como Java y Python, 
no genera ejecutables sino codigos intermedios interpretables, asi que hay tres 
entornos:

* Interprete: `lua` o `lua5.1` o `lua5.3` etc
* Bycompilter: `luac` o `luac5.1` o `luac5.3` etc
* Jitcompiler: `luajit`

Dado el JIT es para 5.1 asumiremos siempre 5.1 el entorno:

```
lua -v
Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio

luajit -v
LuaJIT 2.0.3 -- Copyright (C) 2005-2014 Mike Pall. http://luajit.org/
```


## Guia rapida para impacientes

Para el interprete, el tipico "hola mundo" es:

```
lua -e "print ('hola mundo 2')"
```

Para el bycompiler, el tipico "hola mundo" es:

```
cat > holamundo.lua << EOF
print ('hola mundo 1')
EOF
luac holamundo.lua -o holamundo.luac
lua holamundo.luac
```

Para el jitcompiler, se toma lo anterior y es:

```
luajit -be "print('hola mundo')" holamundo.out  # desde linea comando o sino:
luajit -b holamundo.lua holamundo.obj    # genera codigo compilado objeto
ld holamundo.obj holamundo               # enlaza como si fuera un programa c y listo
./holamundo
```

Usar variables:

`lua -e "a=1" -e "print(a)"`

Concatenar variables con el print:

`lua -e "a=1" -e "print('La variable a tiene:'..a)"`

Sumar dos numeros:

`lua -e "a=1" -e "b=2" -e "c=a+b" -e "print('La variable c fue:'..c)"`

Pedir una variable al usuario

```
cat > holamundoingresa.lua << EOF
print('ingresa numero:'')
a = io.read('*number')
print('el numero es:'..a)
EOF
lua holamundoingresa.lua
```

Compliquemosno, avancemos y hagamos recursividad:

```
cat > ultrapeludoprograma.lua << EOF
function llamameamimisma (n)
      if n == 0 then
        return 1
      else
        return n * llamameamimisma(n-1)
      end
    end
    
    print("ingresa un numero:")
    a = io.read("*number")
    print('llamandolo a simismo imprime :'..llamameamimisma(a))
EOF
lua ultrapeludoprograma.lua
```

En este ultimo juntamos todo lo aprendido:
* usamos variables
* imprimimos
* leimos una variable del usuarios
* y usamos funciones

Adicional hemos empleado calculo matematico avanzado, 
el numero ingresado lo multiplicamos tantas veces el mismo 
hasta agotarse el unico ciclo, es decir lo multiplicamos 
por el mismo (pero restando uno) una vez mas y acto seguido 
lo imprimimos.

## Guia mas detallada pero larga

Lua es un lenguaje de programación imperativo, estructurado y
bastante ligero que fue diseñado como un lenguaje interpretado con una semántica extendible.


#### Comentarios

Los comentarios en lua son los siguientes:

```
-- Esto es un comentario de una linea

--[[
 Este es un comentario multi linea
]]--
```

#### Variables

Una variable es una palabra que almacena un valor.

`variable = valor`

en `lua` hay variables locales, globales y tablas.

```lua
variable = "global" -- lua reconoce todas las variables como globales a no ser, que la variable este declarada como local de esta manera
local variable = "local"
-- arreglo o tabla simple
arreglo = {"uno","dos","tres"} -- valores
```

#### Tipos de datos

Lua es un lenguaje dinámicamente tipado. Esto significa que las variables no tienen tipos; sólo tienen tipo
los valores. No existen definiciones de tipo en el lenguaje. Todos los valores almacenan su propio tipo.

Todos los valores en Lua son valores de primera clase. Esto significa que todos ellos pueden ser
almacenados en variables, pueden ser pasados como argumentos de funciones, y también ser devueltos
como resultados.

En lua hay 8 tipos, pero veremos 6:_nil_,_boolean_,_number_,_string_,_function_,_table_. Nil es el
tipo del valor nil, cuya principal propiedad es ser diferente de cualquier otro valor; normalmente representa
la ausencia de un valor útil.El valor Boolean es false (falso) y true (verdadero). Tanto nil como
false hacen una condición falsa; Number representa números reales (en coma flotante y doble precision).
String representa una tira de caracteres. Lua trabaja con 8 bits: los strings pueden contener
cualquier carácter de 8 bits, incluyendo el carácter cero (0).

veamos un ejemplo usando variables:

```lua
-- tipo string
nombre = "Victor Díaz"
-- tipo number
edad = 16
nombre_completo = nombre .. " tiene" .. edad .. " años" -- los dos puntos (..) sirven para unir cadenas de caracteres con variables
-- tipo boolean
masculino = true
-- tipo nil o nulo
alien = nil
-- tipo float
decimal = 1730.3
-- tipo table/array
estados = {
    "Bolivar", 
    "Carabobo", 
    "Miranda", 
    "Zulia", 
    "Trujillo", 
    "Apure"
}
```

para comprobar que las variables son del tipo especificado usamos la funcion interna de lua `type`, que retorna un string que 
describe el tipo de un valor dado.

veamos un ejemplo:

```lua
print(type(nombre))
```

si colocamos la variable nombre, la salida sera la siguiente:

```
string
```

y asi puedes probar con las de mas variables.

## Operadores.

### Operadores aritméticos.

Son aquellos que permiten realizar operaciones matemáticas básicas, Lua tiene, los operadores aritméticos comunes:

| Operador | Propósito |
| -------- | --------- |
| + | adición |
| - | substracción |
| * | multiplicación |
| / | división |
| // | división  de piso |
| % | modulo |
| - | Negación (Unario) |

### Operadores logicos.

 Los operadores lógicos en Lua son and, or y not. Como las estructuras de control todos los operadores lógicos consideran false
y nil como falso y todo lo demás como verdadero.

El operador negación not siempre retorna false o true. El operador conjunción and retorna su primer operando si su valor
es false o nil; en caso contrario and retorna su segundo operando. El operador disyunción or retorna su primer 
operando si su valor es diferente de nil y false; en caso contrario or retorna su segundo argumento. 
Tanto and como or usan evaluación de cortocircuito; esto es, su segundo operando se evalúa sólo si es necesario.

| Operadores |
| ---------- |
| and        |
| or         |
| not        |

vease tambien [tabla de la verdad de operadores logicos](https://es.wikipedia.org/wiki/Tabla_de_verdad)

### Operadores relacionales.

Los operadores relacionales son usados para comparar dos valores, si la exprecion evaluada es correcta dará 
un resultado true (verdadero) de lo contrario sera false (falsa). 
Los operadores relacionales en Lua son:

| Operadores |       Uso     |     Nombre     | Descripción |
| ---------- | ------------- | -------------- | ----------- |
| ==         |     1 == "texto"    | Igualdad       | 1 es igual a "texto", no por que 1 es un valor de tipo number y "texto" es un valor de tipo "string", por lo tanto retorna un valor booleano en este caso false. |
| >          |         3 > 2       | Mayor que      | 3 es mayor que 2, si por lo tanto retorna true. |
| <          |         5 < 9       | Menor que      | 5 es menor que 9, por lo tanto retorna true. |
| >=         |        7 >= 7       | Mayor  o igual | 7 es mayor que 7, no pero si es igual, por lo tanto retorna true. |
| <=         |        9 <= 6       | Menor  o igual | 9 es menor que 6 o igual, no por lo tanto retorna false. |
| ~=         |    1 ~= "texto"     | Diferente      | 1 es diferente a "texto" que es un valor de tipo "string", si por lo tanto retorna true. | 

## Estructuras de control

Las estructuras de control if, while y repeat tienen el significado habitual.

Lua tiene también una sentencia for, en dos formatos.

### Condicionales

Para crear una condicional se utiliza la palabra reservada `if`.

ejemplo de sintaxis:

```lua
if ( condicion ) then
    ...
end
```

Si la condición se cumple (es decir, si su valor es true) se ejecutan todas las instrucciones que se encuentran 
dentro del bloque {...}. Si la condición no se cumple (es decir, si su valor es false) no se ejecuta ninguna instrucción
contenida en {...} y el programa continúa ejecutando el resto de instrucciones del script.

Ejercicio:

```lua
edad = 16

if edad == 16 then
    print "Tengo 16 años de edad."
else
    print "No tengo 16 años"
end

edad = 12

if (edad == 16) then
    print("Tengo " .. edad .. " años de edad.")
else
    print("No tengo " .. edad .. " años.")
end
```

El if comprueba si se cumple la condición (que puede estar entre paréntesis) e imprime en la pantalla (a través de una consola) 
lo que hay debajo del if (el cuerpo), en caso contrario imprime lo que hay debajo de else (el cuerpo), 
que viene a decir "si no". Como ves, los valores se asignan a las variables a través de un solo igual (=), mientras que 
para comprobar en las condicionantes hay que usar forzosamente doble igual (==). Además del doble igual, también se puede 
comparar utilizando mayor (>), menor (<), mayor o igual (>=), menor o igual (<=) y distinto (~=).

Se pueden establecer varias condiciones a la vez, pudiendo utilizar and, or y not.

Elaboraremos un script que me simule una clave de acceso. Si el usuario es: "admin" y la clave "123456" mostrara el mensaje "acceso permitido" caso contrario mostrara el mensaje "acceso denegado".

usaremos las funciones `print` y `io.read`, con `print` imprimimos en pantalla lo que se le pase como parametro y con `io.read`
escribiremos en patalla.

```lua
-- @Nota: la tilde no puede estar en una variable

print("Usuario:")
usuario = io.read() -- lo que insertemos cuando pida el nombre de usuario, lo guardamos en la variable usuario.
print("Contraseña:")
contrasena = io.read() -- lo que insertemos cuando pida la contraseña de usuario, lo guardamos en la variable contrasena.

if ( usuario == "admin" ) and (contrasena == "123456") then
    print("acceso permitido")
else
    print("acceso denegado")
end
```

ahora podemos hacer que cuando ingresen el usuario de "admin" pero la "contrasena" que imprima en pantalla "acceso denegado, su contraseña no es valida"
y al reves con "contrasena" y "admin" con otra condicion llamada elseif que ya la veremos:

```lua
print("Usuario:")
usuario = io.read() -- lo que insertemos cuando pida el nombre de usuario, lo guardamos en la variable usuario.
print("Contraseña:")
contrasena = io.read() -- lo que insertemos cuando pida la contraseña de usuario, lo guardamos en la variable contrasena.

if ( usuario == "admin" ) and (contrasena == "123456") then
    print("acceso permitido")
elseif (usuario == "admin") and (contrasena ~= "123456")
    print("acceso denegado, su contraseña no es valida")
elseif (usuario ~= "admin") and (contrasena == "123456")
    print("acceso denegado, su usuario no es valido")
else
    print("acceso denegado")
end
```

### Bucles

#### While

La estructura while ejecuta un simple bucle, mientras se cumpla la condición. Su definición formal es la siguiente:

Ejemplo de codigo:

```
while (condición) do
    -- cuerpo
end
```

digamos que tenemos una alarma y lo programamos para que comienze a las 0 horas, y cuando llegue a las 10 horas se detenga
y suene la alarma, bueno con este ejercicio lo hacemos.

la variable estado representa el estado de la alarma en este caso esta encendida, y la variable hora representa la hora en la que
inicia la alarma, en este caso a las 0 horas, bueno, el bucle se encargara de contar las horas hasta que finalice en este caso
a las 10 horas, dicho esto cuando el bucle llegue a dicha hora se detendra la alarma y mandara un mensaje.

Ejercicio:

```lua
estado = true
hora = 0

while (estado) do
	hora = (numero + 1)
	print("Son las" .. hora)
	if (hora >= 10) then
		estado = false
        print("Despierta")
	end
end
```

#### Repeat
La estructura repeat es parecida a la anterior con la diferencia que si la condición es verdadera el ciclo se interrumpe, 
en otros lenguajes se parece al do while.

Ejemplo:

```lua
repeat
    --cuerpo
until condición
```

bueno haremos el ejercicio anterior para que vean el parecido.

Ejercicio:

```lua
hora = 0

repeat
  
  hora = hora + 1
  print(hora)
  if (hora == 10) then
    print("Despierta")
  end
until hora == 10
```

#### For

La estructura for al igual que while repite un bloque de código siempre y cuando  sus condiciones se cumplan, en Lua se 
puede usar de dos formas.

#### Loop for numérico

Ejecuta el bloque con la variable igual a inicio, luego sigue incrementándola cantidad de pasos y ejecutando el bloque 
nuevamente hasta que es mayor que el valor de parada. la variable de paso se puede omitir y se establecerá de manera 
predeterminada en 1 (Se entenderá mejor en el ejemplo).

Ejemplo:

```lua
for variable = inicio, parada, paso do
    -- cuerpo
end
```

bueno haremos el ejercicio anterior para que vean el parecido.

Ejercicio:

```lua
for hora = 0 , 10, 1 do
    print("Son las " .. hora)
    if (hora == 10) then 
        print("Despierta")
    end
end
```

#### Loop for iterador

La versión del iterador `for` toma una función de iterador especial y puede tener cualquier cantidad de variables. 
se entendera mejor mas adelante.

Ejemplo:
```
for variable1, variable2, variable3 in iterador do
    -- cuerpo
end
```

En este ejercicio contaremos los habitantes de alguna ciudad, y la mostraremos en pantalla cual tiene mas y cual tiene menos, 
para ello crearemos una tabla bidimencional con el nombre de "ciudades", en esa tabla añadiremos cuatro variables
con el nombre de ciudad y su valor (el nombre, de la ciudad) y otras cuatro con el nombre de "habitantes" su valor es de tipo number(númerico)

en el bucle `for` la variable i seria el "indíce" y v el "valor", esas variables estaran asociadas
a la tabla ciudades donde i tomara el lugar de la "posicion de la ciudad" y v tomara el lugar de "ciudad o habitantes",
luego con una condicion verificamos cual variable tiene mas habitantes con la funcion `tonumber` le decimos al script que el valor
de "habitantes" debe ser un numero, luego el bucle dara vueltas hasta encontrar la ciudad con mas habitantes.

Ejercicio:

```
-- Arreglo Bidimensional

ciudades = {
	{ciudad = "Upata", habitantes = "1200"}, -- numero de indíce 1
	{ciudad = "Tumeremo", habitantes = "4000"}, -- numero de indíce 2
	{ciudad = "San felix", habitantes = "1500"}, -- numero de indíce 3
	{ciudad = "Puerto ordaz", habitantes = "1400"}  -- numero de indíce 4
}

-- i = indice de la ciudad
-- v = valor de la ciudad

for i, v in ipairs(ciudades) do
	if (tonumber(v.habitantes) > 1000 and tonumber(v.habitantes) < 1500) then
		print(i, v.ciudad," tiene ".. v.habitantes .. " habitantes")
	end
end
```

### Funciónes

Básicamente para cualquier lenguaje de programación, una estructura de control permite modificar el flujo de ejecución de instrucciones de un programa.

Lua cuenta con 4 estructuras de control, la estructura if, while, repeat y for.

Para crear una función en Lua, se utiliza la palabra reservada function , la estructura básica es la siguiente:

```lua
function nombreFuncion ()
    -- cuerpo
end
```

Pero también podemos declarar funciones como si se trataran de variables utilizando la siguiente estructura:

```lua
nombreFuncion = function ()
    -- cuerpo
end
```

### Funciones con argumentos

Las funciones también pueden recibir argumentos, esto quiere decir que son parámetros o valores de 
entrada que son enviados hacia la función.

```lua
function nombreFuncion (argumento1, argumento2)
    -- cuerpo
end
```

Ejercicio basico:

```lua
function saludar ( nombre )
    print("Hola " .. nombre)
end

saludar("Victor")
```

Si ejecutamos nos devuelve el valor de:

`Hola Victor`

Como ejercicio complejo, haremos la famosa calculadora, estaremos usando la funcion `type` que retorna un string que 
describe el tipo de un valor dado, `io.read` escribiremos en patalla, `print` con la cual imprimimos en pantalla y condicionales.

nuestra funcion se llamara "calculadora" y recibira tres parametros, los cuales seran a que vendria siendo el primer valor,
op que seria el operador (+,-,*,/) y b que seria el segundo valor para hacer la operacion.

luego comprobaremos con la funcion `type` si el op (operador) es un string y el op es "+" sumamos a con a , si es "-" restamos 
a con b y asi con los demas,
si ninguna condicion se cumple, imprimimos en pantalla "el operador no es valido".

Ejercicio:

```lua
function calculadora(a, op, b)
    if (type(op) == 'string') then
		if (op == '+') then
			print(a + b)
		elseif (op == '-') then
			print(a - b)
		elseif (op == '/') then
			print(a / b)
		elseif (op == '*') then
			print(a * b)
		else
			print("el operador no es valido")
		end
	end
end

print("Introdusca primer valor")
a = io.read()
print("Introdusca operador")
op = io.read()
print("Introdusca tercer valor")
b = io.read()

print("El resultado es ")

calculadora(a, op, b)
```

## Copyright

* author: diaz.victor@openmailbox.org
* modificado para VenenuX: @mckaygerhard
* Facebook: https://www.facebook.com/DiazUrbanejaVictor
* Github: https://github.com/diazvictor/

