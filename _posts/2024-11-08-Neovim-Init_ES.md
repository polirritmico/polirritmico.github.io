---
title: '(WIP) Aventuras en Neovim: Comencemos por el init'
date: '2024-11-08 20:02 -0300'
permalink: /posts/Neovim-Init
lang: es
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: Primer capítulo de las aventuras en Neovim.
media_subpath: /assets/img/neovim/
image: nvim.png
---
# Aventuras en Neovim (WIP)

## Generalidades y Objetivos

Llevo algunos años utilizando casi exclusivamente [Neovim](https://neovim.io/)
para la edición de textos y código en mi sistema. A lo largo de este tiempo he
ido acumulando algunas recetas, conceptos e ideas (propias, copiadas y
adaptadas) que me han llevado a una organización bastante estable y flexible de
mi entorno de desarrollo junto a una serie de criterios que quiero compartir a
través de la presente serie de artículos.

No todas estas ideas han sobrevivido, la gran mayoría de hecho las he eliminado
al quedar relegadas por meses a código sin uso y por ende, al no haberse
integrado de forma orgánica en mi flujo de trabajo. Sin embargo, el aprendizaje
y la exploración de las posibilidades que ofrece Neovim para materializarlas
permiten extraer un aprendizaje realmente útil al enfrentar nuevos desarrollos.
El compartir este aprendizaje y sus conclusiones creo que puede ser algo
bastante interesante, en especial para aquellos que recién comienzan con su
propia _NeoVimtura_. Además, lo que a uno puede parecerle inútil, bien puede ser
la clave o el punto de inflexión para otro.

El generar material didáctico que aborde estos puntos y sobretodo con un poco
más de profundidad en cuanto al por qué de las cosas en lugar de solo hacer un
tutorial o receta a seguir, es otro de los objetivos principales de estos
artículos. Me interesa el poder trasmitir un enfoque más profundo sobre lo que
es el desarrollo de nuestras herramientas, el tratar de aportar al _"¿Por qué
quiero hacer esto?"_ o el más fundamental _"¿Será esta la mejor forma de abordar
el problema?_, pues a fin de cuentas, el _sacarle filo_, el ajustar una pieza
tan central para todos los que trabajamos con textos día a día puede traer
potencialmente beneficios que disfrutaremos diariamente. Evidentemente no aspiro
a llegar a una conclusión que sea válida para cada caso de uso, pero sí al menos
a poder plantear preguntas que considero más fundamentales.

### Plan de ruta. Qué y cómo.

_A priori_, la serie estará compuesta por los siguientes artículos:

> 1. Inicio y guía (este artículo)
> 2. Teclas y criterios
> 3. Organización y manejo de errores
> 4. Utilidades y personalización

### A quién

Si estás leyendo esto asumo dos cosas. Primero, que **te interesa** —_incluso
entretiene_— el ajuste de tus herramientas de trabajo, cual carpintero afilando
y afinando el filo de su cepillo hasta dar con el sonido y viruta perfectos. Y
en segundo lugar, asumo que **no quieres perder tiempo reinventando la rueda**.
Que no nos ocurra, que por ahorrarnos una lectura de minutos perdamos horas o
incluso _días_ en torno problemas ya resueltos y al alcance de una función, flag
o parámetro.

La serie estará principalmente enfocada hacia:

- Usuarios de Neovim con un poco de experiencia, curiosidad o que ya han dado
  sus primeros pasos con el editor y quieren armar por primera vez su propia
  configuración o una más definitiva.
- Aquellos que ya han escrito su configuración pero no les termina de convencer
  (problemas de encapsulación, rendimiento, comportamiento, etc.)
- Aquellos entusiastas que siempre están buscando formas e ideas de optimización
  o solo puntos de vista diferentes.
- Usuarios nóveles que aún naufragan en el mar de sobrecogedora abundancia que
  ofrece nvim sobre cómo resolver las cosas.

### Conocimiento previo

Específicamente la serie asumirá cierto conocimiento previo de los siguientes
puntos:

- **Funciones básicas:** Cambiar de modos, escribir texto, guardar, salir, etc.
  Por completitud las mencionaré, pero para aprender eso sugiero `:Tutor`.
- **Movimientos Vim:** Sí, editar un string mediante `ci"` es hermoso, pero ya
  se ha escrito demasiado al respecto.
- **Vimscript:** Ni en los días de Vim nunca me agradó y siempre lo encontré
  demasiado específico. A día de hoy esta sensación se mantiene y solo diré que
  prefiero aprender Lisp y cambiarme a Emacs a adentrarme en él (gracias Lua).
- **Conceptos de programación:** Utilizaré conceptos de programación tales como
  función, variable, tablas, API, etc., pero al mismo tiempo trataré de explicar
  a lo que me refiero para los más alejados o novatos con el código. Por lo
  demás, Lua es un lenguaje bastante sencillo, e incluso amigable en mi opinión,
  de modo que es una perfecta oportunidad para adentrarse un poco en la
  programación.
- **Cosas Windows:** Lo siento, hace años que no uso ese sistema operativo y
  creo que si los PC vinieran con Linux por defecto de las tiendas la
  informática en general se encontraría en un estado de mayor salubridad.

No obstante los puntos anteriormente mencionados, trataré de incluir recuadros
de ayuda y _tips_ que provean cierta información que considere valiosa o útil de
manejar sobre estos temas.

### Metodología

Va a depender del contenido, pero se podría acotar a los siguientes puntos:

- Plantearnos preguntas como usuarios y desarrolladores.
- Plantearnos meta-pregunta (el cuestionarnos si estamos o no haciendo la
  pregunta correcta).
- Encontrar la nomenclatura correcta y explorar qué nos dice la documentación de
  Neovim al respecto.
- Revisión de las posibilidades.
- Escribir el código.
- Conclusiones.

Con esto espero contribuir a una noción más depurada del cómo y cuándo construir
nuestras propias soluciones, que estas se adapten específicamente a nuestros
requerimientos (o lo que creamos necesitar), y por supuesto que ello no degrade
el desempeño o quite flexibilidad al código de configuración.

Con estas aclaraciones generales, procedamos al meollo de este capítulo: El
inicio, nuestro `init.lua`.

**¡Manos a la obra!**

---

## Init

_Volver al principio_

Para comenzar esta serie de aventuras, creo que lo mejor es hacerlo en partida
doble, es decir, el aprender acerca de cuál es la estructura de directorios y
ficheros que propone Neovim, y al mismo tiempo, el aprender a cómo utilizar la
documentación para evitar tener que estar abriendo el navegador web cada vez que
no sepamos cómo abordar alguna situación puntual. Mientras más tiempo estemos en
Neovim, más rápido nos familiarizaremos con su flujo, terminología; y el
aprender a navegar entre **ventanas** y **buffers** es algo que mientras antes
familiaricemos mejor.

<!-- prettier-ignore-start -->
> **Es importante matizar nuestro proceso de aprendizaje**:
>
> Un consejo que personalmente me sirvió mucho para una transición exitosa a
> Neovim y que siempre trataré de compartir, es el de no desechar completamente
> el editor con el que ya estamos familiarizados mientras nos encontremos en
> este estado de aprendizaje.
>
> Si después de un rato trabajando en Neovim, ya estamos cansados o frustrados y
> todo se siente cuesta arriba, o si de pronto tenemos que resolver situaciones
> inmediatamente o responder a fechas límite; lo mejor sin lugar a dudas en
> estos casos es volver a lo conocido y seguir aprendiendo en otro momento. De
> esta forma evitamos frustraciones completamente válidas y mejoramos nuestra
> capacidad de comprensión cuando realmente deseamos aprender.
>
> En contraste a otros editores, Neovim supone un paradigma completamente
> diferente, de modo que puede ser bastante abrumador en ciertos momentos el
> abordarlo todo al mismo tiempo.
{: .prompt-warning }
<!-- prettier-ignore-end -->

Entonces, a levantar nuestra terminal favorita
([Konsole](https://konsole.kde.org/) en mi caso) y ejecutar Neovim:

```sh
$ nvim
```

<!-- prettier-ignore-start -->
> Cuando vemos un comando de shell precedido por el signo `$` o (`%`), por
> convención quiere decir que podemos utilizarlo con cualquier usuario sin tener
> problemas de permisos. Por el contrario, cuando vemos que comienza con `#`,
> quiere decir que se espera tener permisos de superusuario o `root`.
{: .prompt-tip }
<!-- prettier-ignore-end -->

Asumo que no tenemos ningún fichero ni directorio de configuración. A futuro,
una forma de volver a abrir temporalmente Neovim en este estado por defecto es
mediante `nvim --clean`. Muy útil en especial cuando algo no funciona y queremos
conocer si la causa del problema está en nuestra configuración o no.

<!-- prettier-ignore-start -->
![Interfaz por defecto](default-nvim.png)
_La interfaz al vacío, tal como la belleza de una piedra al natural conteniendo
todas las formas y esculturas_
<!-- prettier-ignore-end -->

Bueno, con el editor ya abierto, ayudémosnos a nosotros mismos y veamos como
navegar por la ayuda.

## Ayuda y ventanas

Para abrir la ayuda de Neovim tenemos que abrir la `command-line` y ejecutar el
comando `help`.

<!-- prettier-ignore-start -->
> En terminología vim/nvim, ejecutar `:help` quiere decir:
>
> 1. Abrir la `command-line` (en **modo normal**, pulsar la tecla `:`).
> 2. Teclear el comando `help` (opcionalmente podemos escribir solo `h`).
> 3. Pulsar `<CR>` (la tecla `enter`)
{: .prompt-info }
<!-- prettier-ignore-end -->

Recordemos que Neovim es un **editor modal**, esto quiere decir que el teclado y
todo el software va a cambiar su funcionamiento en base al modo que nos
encontremos. En este caso, por defecto nos encontramos en el _modo normal_ al
que podemos retornar fácilmente en casi todo momento pulsando `<Esc>`. En fin,
con las instrucciones anteriores deberíamos haber abierto la ventana de ayuda
(`<F1>` también funciona).

Y así de simple, ya nos encontramos con el primer desafío: **¡¿Tengo que leer la
documentación ocupando solo la mitad de la pantalla?!** ¡Pues no! Precisamente
el resolver este tipo de situaciones es de lo que se trata esta serie. De modo
que antes de solucionarlo, invito a reflexionar un poco sobre este punto.

<!-- prettier-ignore-start -->
![Help!](help.png)
_Tanto desperdicio de espacio..._
<!-- prettier-ignore-end -->

La pregunta clave aquí es **¿Qué queremos hacer?**, y la respuesta no es obvia
como puede parecer: _"Quiero que la pantalla de ayuda ocupe toda la pantalla y
no solo la mitad, así que configuremos eso"_. Pues sí y no. Esta es la primera
[trampa x-y](https://xyproblem.info/) a la que nos enfrentamos. Lo que realmente
queremos no es que la **ventana** de ayuda ocupe toda la pantalla, sino el poder
leer la documentación cómodamente.

¡Retórica! Podrán decir algunos no sin un dejo de razón, no obstante el cambio
de foco es importante. _Es el contexto lo que determina nuestra comodidad_, y
recién nos estamos adentrando en esta herramienta. ¿Tenemos contexto suficiente
para cuestionar esta decisión? ¿En qué estaban pensando los desarrolladores
cuando se decidieron por este comportamiento por defecto? No lo sabemos. ¿Será
la ayuda en pantalla completa cómoda en los casos de utilización más usuales?
Tampoco lo sabemos. No al menos mientras estemos en este proceso de
familiarización y ciertamente no cuando apenas sabemos organizar y movernos
entre ventanas.

Por lo tanto nuestro primer instinto, el de _"voy a configurar la ventana de
ayuda para que se abra a pantalla completa por defecto"_, obedece al mismo
principio de optimización prematura, y por ende considerémoslo como un error en
este momento o al menos una decisión desinformada. Quizás efectivamente sea el
ajuste adecuado, pero carecemos de exploración suficiente para determinarlo.

El cambio de este comportamiento por defecto puede suponer una mayor pérdida de
tiempo que el beneficio o comodidad que estamos buscando. Una futura
_configuración fantasma_ que ahora encontramos lógica, pero es potencialmente
una de esas opciones que sabiendo navegar y manejar las ventanas puede tender a
entorpecer más que ayudar o acomodar.

¿Entonces como lo arreglamos? No lo hacemos. Simplemente, teniendo el foco sobre
la ayuda (o el foco sobre la _ventana_ que tiene el _buffer_ de ayuda) pulsamos
`<C-w>o` para hacer de la ventana seleccionada la única en la pantalla.

<!-- prettier-ignore-start -->
> **Escritura de combinaciones de teclado al estilo Neovim/Vim:**
>
> `<C-w>o` quiere decir que pulsemos las teclas `ctrl+w`, seguidas de la tecla
> `o`. Como ayuda mnemotécnica, podemos entender el `<C-w>` como _"cosas de
> ventana"_ (**w**indows) y la tecla `o` de **o**pen (abrir).
{: > .prompt-info }
<!-- prettier-ignore-end -->

Ahora, con la documentación a pantalla completa, nos queda dilucidar el cómo
buscamos la documentación acerca de cómo configurar Neovim. Intentemos entonces
encontrar esta sección con `:help configuration`.

¡Oh! ¿¡Qué rayos es `lsp-handler-configuration`!? Bueno, evidentemente no es lo
que buscamos.

Tradicionalmente las aplicaciones en terminales utilizan mecánicas que siguen
ciertos patrones de uso históricos. En este caso precisamente nos encontramos
frente a uno de ellos, dentro de la `command-line`, podemos utilizar `<Tab>` a
modo de autocompletar nuestro comando y sus argumentos. En este caso si
escribimos solo la primera `h` de `help` y pulsamos `<Tab>`, Neovim
**autocompletará** el comando. Cuando existe más de una posibilidad abrirá un
menú donde podemos elegir la opción que queramos y pulsar `<Tab>` nuevamente
para **expandir** la selección.

De esta forma, podemos ver que buscando `conf` tenemos muchas posibilidades a
elegir tales como `:conf`, `config`, `confirm`, etc. Intentemos con
`:help config`. ¡Ah!, ahora sí estamos hablando, esto sí tiene sentido.

Este es uno de los puntos que se nos irá facilitando a medida que nos
familiaricemos con Neovim, pues en principio no tenemos ninguna noción de la
terminología que utiliza el manual. Más adelante podremos utilizar
**Telescope**, **Fzf** u otras alternativas para ayudarnos con esto, pero por el
momento ya podemos leer en la ayuda el siguiente mensaje (editado):

> A file containing initialization commands is generically called a "vimrc" or
> config file...
>
> The config file is located at:
>
> - Unix: `~/.config/nvim/init.lua`
> - Windows: `~/AppData/Local/nvim/init.lua`
> - XDG: `$XDG_CONFIG_HOME/nvim/init.lua`

OK, generemos este archivo. ¿Pero cómo cerramos esta ventana de ayuda? Por
supuesto, pero aprovechando la necesidad indaguemos un poco en esto de las
ventanas.

## Moviendo entre ventanas

Podríamos intuir que lo que está mostrando cada ventana es _el archivo_, pero
esto no es del todo cierto. Neovim/Vim operan bajo una abstracción llamada
**buffer**. Apenas terminemos

Cuando hay más de una ventana en la pantalla actual, podemos cambiar el foco con
`<C-w>` más la dirección de la ventana (`h`, `j`, `k` o `l`).

## Paths

En terminología Unix, `~` representa el directorio home de nuestro usuario. Por
ejemplo, si nuestro usuario se llama `foo`, entonces `~/` representa la ruta
`/home/foo/` en nuestro sistema de archivos. Además, por lo general en sistemas
Linux es habitual tener por defecto la variable de entorno `XDG_CONFIG_HOME`
apuntando a `~/.config/`. Podemos revisar hacia dónde apunta, abriendo una
terminal y revisando la salida del comando:

```sh
$ echo "$XDG_CONFIG_HOME"
```

Ergo, Neovim va a buscar nuestro archivo `init.lua` en
`/home/<nombre de usuario>/.config/nvim/init.lua`. Creémoslo:

```sh
$ cd
$ mkdir .config/nvim
$ cd .config/nvim
$ touch init.lua
```

¡A editar!

```sh
$ nvim init.lua
```

Aquí tenemos nuestro glorioso archivo de configuración:

<!-- prettier-ignore-start -->
![Fichero vacío en Neovim](base-init.png)
_¡Nuestra hermosa página en blanco! ¡Véanla! ¡Tan llena de posibilidades!_
<!-- prettier-ignore-end -->

Ahora veamos si Neovim está leyendo o no el fichero a través de el clásico
_¡Hola mundo!_ (Sí, _Hola mundo_ para configurar nuestro editor). Utilizamos el
siguiente código:

```lua
-- (Los comentarios comienzan con --)

-- La siguiente función nos permite notificar al usuario con un mensaje:
vim.notify("¡Hola mundo!")
```

<!-- prettier-ignore-start -->
> Una buena fuente de información al respecto del lenguaje Lua la encontramos
> —como no— dentro de la documentación: `:help lua`.
{: .prompt-tip }
<!-- prettier-ignore-end -->

El código lo ingresamos cambiando del `normal-mode` al `insert-mode` con la
tecla `i` y una vez escrito volvemos al modo normal con `<Esc>`. Para guardar
los cambios y cerrar la instancia actual Neovim, podemos pulsar `:qw` o `<ZZ>`.

Tras volver a abrir Neovim tenemos esto:

![Hola mundo](hello-world.png)

Perfecto. Todo funcionando. Una vez comprobado limpiemos el archivo.

> Si en cualquier momento queremos volver a ver este o cualquier mensaje de la
> sesión actual podemos utilizar el comando `:messages` o `:mess`.

<!-- prettier-ignore-start -->
> Lua ofrece la función `print` que podemos utilizar en lugar a `vim.notify`,
> sin embargo, prefiero esta última porque permite agregar niveles de mensajes
> (advertencias, errores) y según la documentación se puede reemplazar por un
> proveedor personalizado.
{: .prompt-info }
<!-- prettier-ignore-end -->

## Nuestro init.lua

El cómo organizar nuestra configuración es un tema que abordaremos en detalle
más adelante, sin embargo en términos generales podemos mencionar dos opciones:

1. Manejamos todo en un único `init.lua`
2. Separamos los distintos ámbitos de nuestra configuración en múltiples
   archivos.

Ambas estrategias ofrecen beneficios y dificultades, es más que nada una
decisión personal, pero creo en base a mi experiencia trabajando en distintas
bases de código; que tarde o temprano un único archivo casi siempre se vuelve
difícil de mantener y organizar. Quizás la excepción más plausible, sea que
nuestra configuración ya haya alcanzado un punto de estabilidad importante y a
su vez, ésta no tenga tanta longitud en líneas de código.

De cualquier forma, creo que en este caso la estrategia correcta (o quizás la
con mayor probabilidad de demostrarse correcta) es dividir y agrupar nuestro
_setup_ en base a ficheros (o módulos) de funcionalidad. Esta división como en
cualquier código, es un desafío de sutiles implicaciones y allí su dificultad,
pero _mucha agua ha pasado bajo este puente_, de modo que podemos aplicar la
solución que muchos han encontrado al respecto, y es precisamente la que
aplicaremos: **dividir nuestra configuración en múltiples ficheros**. Además, el
navegar por los diversos ficheros de un proyecto en Neovim es una gozada en
comparación a otras IDE, sobretodo si lo potenciamos con algunos _plugins_
(aunque las posibilidades por defecto también son muy buenas).

Creemos entonces la siguiente propuesta de organización:

```
nvim/
├── init.lua
└── lua
    └── config
        ├── mappings.lua
        └── settings.lua
```

Y para que neovim "lea" los ficheros `mappings.lua` y `settings.lua` utilicemos
la función `require` dentro de nuestro `init.lua`:

```lua
require("config.settings")
require("config.mappings")
```

`require` es una función Lua que opera bajo la abstracción de **módulos**. Un
módulo Lua, es básicamente un fichero `.lua` que Lua lee a través de la función
`require`. Tras la lectura del archivo, ejecuta su código y guarda el resultado
en memoria de tal modo que sus funciones, variables o configuraciones queden a
disposición para utilizarse o reutilizarse.

En el caso anterior, `settings` y `mappings` son nuestros módulos y `config`
sería un directorio contenedor que utilizamos para la organización del código.

Creo que el sentido de cada uno de ambos módulos es autoexplicativo, pero en
cualquier caso, la idea es que dentro de `settings` agreguemos todas las
opciones específicas de Neovim tales como columnas con números relativos, ancho
del texto por defecto, opciones de la barra de estado, etc.; y dentro de
`mappings` agreguemos todo lo referente a atajos y configuraciones de teclado.

Su contenido ya lo revisaremos en la siguiente entrega.

---

## Conclusiones

Hemos avanzado poco, pero al mismo tiempo abarcado muchos elementos. Las futuras
entregas tendrán mucho más código. Por ahora, a modo de conclusión enumeremos
las ideas clave que hemos abarcado:

- Abrir la documentación o ayuda de Neovim.
- Comandos relacionados con las ventanas (`<C-w>`):
  - `<C-w>o` para agrandar una ventana dividida (con el foco en la ventana que
    queremos agrandar).
  - `<C-w>` más la dirección de la ventana a enfocar para cambiar el "foco de la
    ventana".
- Ayuda a través de la `command-line` (`:help <búsqueda>`).
- Archivo de configuración `:h config`.
- Función `require`.

