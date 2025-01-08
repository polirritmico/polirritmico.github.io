---
title: 'Aventuras en Neovim: Comencemos por el init'
date: '2024-11-22 20:02 -0300'
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
# NeoVimturas

## Generalidades y Objetivos

Llevo algunos años utilizando casi exclusivamente [Neovim](https://neovim.io/)
para la edición de textos y código en mi sistema. A lo largo de este tiempo he
ido acumulando algunas recetas, conceptos e ideas (propias, copiadas y
adaptadas) que me han llevado a una organización bastante estable y flexible de
mi entorno de desarrollo junto a una serie de criterios que quiero compartir a
través de la presente serie de artículos.

No todas estas ideas han sobrevivido; de hecho, la gran mayoría solo existe
entre _commits_ olvidados en mi repositorio de configuraciones. Su destino fue
consecuencia de no haberse integrado de forma orgánica a mi flujo de trabajo.
Quedaban relegadas a líneas de código que nunca utilizaba, y peor aún, olvidaba
que existían. Ergo, no sobrevivieron a la limpieza de turno, siempre ávida de
purgar código innecesario en pos de evitar ser morada y caldo de cultivo de los
inevitables _bugs_ al acecho. En el fondo, eran simplemente malas ideas
relativamente bien ejecutadas, y de esas ya tenemos bastantes (por ejemplo,
_stalebots y compañía_).

Sin embargo, este proceso exploratorio en base a la gran variedad de
posibilidades que nos ofrece Neovim, también nos permite el desarrollar o
adquirir un aprendizaje realmente valioso al enfrentar estas implementaciones;
nos permite el poder manipular fácilmente el comportamiento de la herramienta, y
al mismo tiempo, nos enseña a estar siempre atentos a nuestros propios patrones
de uso. Con el tiempo, esta actitud mental convierten al editor en una
herramienta verdaderamente propia y potencialmente la mejor experiencia de
desarrollo personalizada, o al menos la más cómoda en su uso.

El compartir este aprendizaje y sus conclusiones creo que puede ser algo
bastante interesante, en especial para aquellos que recién comienzan con su
propia _NeoVimtura_. Lo que a uno puede parecerle inútil, bien puede ser la
clave o el punto de inflexión para otro. El generar material didáctico que
aborde estos puntos y sobretodo con un poco más de profundidad en cuanto al por
qué de las cosas en lugar de solo hacer un tutorial o receta a seguir, es uno de
los motivantes claves de estos artículos.

Me interesa el poder generar un enfoque más profundo sobre lo que es el
desarrollo de nuestras herramientas, el tratar de aportar al _"¿Por qué quiero
hacer esto?"_ o el más fundamental _"¿Será esta la mejor forma de abordar el
problema?_, pues a fin de cuentas, el _sacarle filo_, el ajustar una pieza tan
central para todos los que trabajamos con textos día a día puede traer
beneficios que potencialmente disfrutaremos a diario. Evidentemente no aspiro a
llegar a una conclusión que sea válida para cada caso de uso, pero sí al menos a
plantear preguntas que considero más fundamentales y que por la ansiedad de
tener todo funcionando _"ahora ya"_ se pierden entre la bruma de posibilidades.

### Plan de ruta. Qué y cómo.

_A priori_, la serie estará compuesta por los siguientes artículos:

> 1. Inicio y guía (este artículo)
> 2. Modulos y fallbacks
> 3. Teclas y criterios
> 4. Organización y manejo de errores
> 5. Utilidades y personalización

### A quién

![Afilando](medieval_tools.jpg)

Si estás leyendo esto asumo dos cosas. Primero, que **te interesa** —_incluso
entretiene_— el ajuste de tus herramientas de trabajo, cual carpintero afilando
y afinando el filo de su cepillo hasta dar con el sonido y viruta perfectos. Y
en segundo lugar, asumo que **no quieres perder tiempo reinventando la rueda**.
Que no nos ocurra, que por ahorrarnos una lectura de minutos perdamos horas o
incluso _días_ en torno problemas ya resueltos y al alcance de una función, flag
o parámetro.

La serie estará principalmente enfocada hacia:

- Usuarios de Neovim con un poco de experiencia , curiosidad o que ya han dado
  sus primeros pasos con el editor (al menos haber completado `:Tutor`) y
  quieren armar por primera vez su propia configuración o una más definitiva.
- Aquellos que ya han escrito su configuración pero no les termina de convencer
  (problemas de encapsulación, rendimiento, comportamiento, etc.)
- Aquellos entusiastas que siempre están buscando formas e ideas de optimización
  o solo puntos de vista diferentes, y que estén dispuestos o tengan la
  paciencia suficiente para saltartse las secciones demasiado básicas.
- Usuarios nóveles que aún naufragan en el mar de sobrecogedora abundancia que
  ofrece nvim sobre cómo resolver las cosas, en busca de alguna guía que les
  señale algun norte o criterios.

Por lo mismo quizás iré más lento de lo que a algunos lectores les gustaría, no
obstante evitaré repeticiones más allá de las fundamentales.

### Conocimiento previo

Específicamente la serie asumirá cierto conocimiento previo de los siguientes
puntos:

- **Funciones básicas:** Cambiar de modos, escribir texto, guardar, salir, etc.
  Por completitud las mencionaré en cualquier caso, al menos para establecer una
  nomenclatura común.
- **Movimientos Vim:** Sí, editar un string mediante `ci"` es hermoso, pero ya
  se ha escrito demasiado al respecto por lo que principalmente serán utilizados
  y anotados, pero no explicados.
- **Conceptos de programación:** Utilizaré conceptos de programación tales como
  función, variable, tablas, _string_, API, etc., pero al mismo tiempo trataré
  de explicar a lo que me refiero para los más alejados o novatos con el código.
  Por lo demás, Lua es un lenguaje bastante sencillo, e incluso amigable en mi
  opinión, de modo que es una perfecta oportunidad para adentrarse un poco en la
  programación con una voz amiga.

<!-- prettier-ignore-start -->
> No obstante los puntos anteriormente mencionados, trataré de incluir recuadros
> de ayuda y _tips_ que provean cierta información que considere valiosa o útil de
> manejar sobre estos temas dentro del contexto.
{: .prompt-tip }
<!-- prettier-ignore-end -->

### Lo que no...

En la serie no abordaré los siguientes puntos:

<!-- prettier-ignore-start -->
- **Vimscript:** Nunca me agradó; ni siquiera en los días de Vim. Siempre lo
  encontré demasiado específico y enrevesado, pero entendía su necesidad. A día
  de hoy, simplemente no me hace sentido y
  [Vim9script incluso me parece un error](https://github.com/vim/vim/blob/be4e01637e71c8d5095c33b9861fd70b41476732/README_VIM9.md).
  Sin embargo, mientras Vim siga siendo el estándar por defecto en el ecosistema
  GNU/Linux (preinstalado en muchas distribuciones), Vimscript seguirá siendo
  relevante; no obstante, Lua simplemente es una mejor solución. Y creo que la
  explosión de plugins que hemos visto desde su adopción en Neovim me da la
  razón. Quizás un buen resumen de mi opinión al respecto es que preferiría usar
  ese tiempo en aprender Lisp y probar Emacs que a adentrarme más de lo
  necesario en sus intrincaciones.
  > ![Vimscript](vimscript.png)
  > _Vimscript "9true"_

- **Cosas Windows:** Lo siento, lo odio. Hace años que no uso ese sistema
  operativo (tiempos oscuros) y creo que si los PC vinieran con Linux por
  defecto de las tiendas, la informática en general se encontraría en un estado
  de mayor salubridad.
<!-- prettier-ignore-end -->

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

<!-- prettier-ignore-start -->
> **Es importante matizar nuestro proceso de aprendizaje**:
>
> Un consejo que personalmente me sirvió mucho para una transición exitosa a
> Neovim y que siempre trataré de compartir, es el de no desechar completamente
> el editor con el que ya estamos acostumbrados mientras nos encontremos en este
> estado de aprendizaje inicial.
>
> Tras un rato trabajando en Neovim, ya estamos cansados, frustrados o
> simplemente todo se siente cuesta arriba, o si de pronto tenemos que resolver
> situaciones inmediatamente o responder a fechas límite; lo mejor en estos
> casos sin lugar a dudas es volver a lo conocido y seguir aprendiendo en otro
> momento. De esta forma evitamos frustraciones (completamente válidas) y
> mejoramos nuestra capacidad de comprensión cuando realmente deseamos aprender.
>
> En contraste a otros editores, Neovim supone un paradigma completamente
> diferente, de modo que puede ser bastante abrumador en ciertos momentos el
> abordarlo todo al mismo tiempo.
{: .prompt-warning }
<!-- prettier-ignore-end -->

---

## Init

_Volver al principio_

Probablemente la mejor forma de aprender a utilizar Neovim es utilizando Neovim.
Con esto me refiero a evitar el abrir a cada duda el navegador para preguntarle
a Google, el foro de turno o a la IA cualquier duda que tengamos, sino,
busquemos las respuestas en la documentación a través de la navegación al 100%
con nuestro teclado.

Por ello, propongo la siguiente tarea: aprendamos y familiaricémonos con la
navegación entre **ventanas** y **buffers**, mientras buscamos información
acerca del sistema de configuración de Neovim.

Entonces, a levantar nuestra terminal favorita
([Konsole](https://konsole.kde.org/) en mi caso) y ejecutar Neovim:

```sh
$ nvim
```

<!-- prettier-ignore-start -->
> Cuando vemos un comando de shell precedido por el signo `$` o (`%`), por
> convención quiere decir que podemos utilizarlo con cualquier usuario sin tener
> problemas de permisos. Por el contrario, cuando vemos que comienza con `#`,
> quiere decir que se espera tener permisos de `root` o superusuario.
{: .prompt-tip }
<!-- prettier-ignore-end -->

Asumo que no tenemos ningún fichero ni directorio de configuración. A futuro,
una forma de volver a abrir temporalmente Neovim en este estado por defecto es
mediante `nvim --clean`. Muy útil en especial cuando algo no funciona y queremos
conocer si la causa del problema está en nuestra configuración o no.

<!-- prettier-ignore-start -->
![Interfaz por defecto](default-nvim.png)
_La interfaz al vacío, tal como la belleza de una piedra al natural conteniendo
todas las formas y esculturas._
<!-- prettier-ignore-end -->

Bueno, con el editor ya abierto, ayudémosnos a nosotros mismos y veamos como
navegar por la ayuda.

## Ayuda y ventanas

Para abrir la ayuda de Neovim tenemos que abrir la `command-line` y ejecutar el
comando `help`: `:help`.

<!-- prettier-ignore-start -->
> En terminología vim/nvim, ejecutar `:help` quiere decir:
>
> 1. Abrir la `command-line` (en **modo normal**, pulsar la tecla `:`).
> 2. Teclear el comando `help` (opcionalmente podemos escribir solo `h`).
> 3. Pulsar `<CR>` (la tecla `enter`)
>
> Aunque obvia de momento, esta información nos será útil más adelante, cuando
> necesitemos definir las acciones de nuestro `keymaps` personalizados.
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
que antes de solucionarlo, invito a reflexionar un poco sobre este punto, en
especial para establecer ciertos criterios.

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
ayuda para que se abra a pantalla completa por defecto y así facilitarme la vida
en el futuro"_, obedece al mismo principio de optimización prematura, y por ende
considerémoslo como un error en este momento o al menos una decisión
desinformada. Quizás efectivamente sea el ajuste adecuado, pero carecemos de la
exploratoria suficiente para determinarlo.

El cambio de este comportamiento por defecto puede suponer una mayor pérdida de
tiempo que el beneficio o comodidad que estamos buscando. Una futura
_configuración fantasma_ que ahora que tenemos todo nuestro foco cognitivo en
ella encontramos lógica, pero es potencialmente una de esas opciones que
sabiendo navegar y manejar las ventanas puede tender a entorpecer más que ayudar
o acomodar.

¿Entonces como lo arreglamos? No lo hacemos. Simplemente, teniendo el foco sobre
la ayuda (o el foco sobre la _ventana_ que tiene el _buffer_ de ayuda) pulsamos
`<C-w>o` para hacer de la ventana seleccionada la única en la pantalla.

<!-- prettier-ignore-start -->
> **Escritura de combinaciones de teclado al estilo Neovim/Vim:**
>
> `<C-w>o` quiere decir que pulsemos las teclas `ctrl + w`, seguidas de la tecla
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
ciertos patrones de uso históricos, y precisamente nos encontramos frente a uno
de ellos. Dentro de la `command-line` podemos utilizar `<Tab>` para
autocompletar nuestro comando y sus argumentos. En este caso si escribimos solo
la primera `h` de `help` y pulsamos `<Tab>`, Neovim **autocompletará** el
comando. Cuando existe más de una posibilidad abrirá un menú donde podemos
elegir la opción que queramos y pulsar `<Tab>` nuevamente para **expandir** la
selección.

<!-- prettier-ignore-start -->
> Para seleccionar las opciones en el menú de autocompletado recomiendo usar
> `<C-n>` y `<C-p>` en lugar de las flechas `<Left>` y `<Right>` que siempre lo
> he encontrado una (configuración por
> defecto)[https://github.com/neovim/neovim/issues/9953] rarísima.
{: > .prompt-tip }
<!-- prettier-ignore-end -->

De esta forma, podemos ver que buscando `conf` tenemos muchas posibilidades a
elegir tales como `:conf`, `config`, `confirm`, etc. (nótese la diferencia entre
los comandos con y sin `:`) Intentemos con `:help config`. ¡Ah!, ahora sí
estamos hablando, esto sí tiene sentido.

Este es uno de los puntos que se nos irá facilitando a medida que, como decimos
por estos lares, no le _quitemos el culo a la jeringa_ y nos vayamos
familiarizando con Neovim; pues en principio no tenemos ninguna noción de la
terminología que utiliza el manual. Más adelante vamos a ver cómo utilizar
**Telescope**, **Fzf** u otras alternativas para ayudarnos con esto. De
cualquier forma ya podemos leer en la ayuda el siguiente mensaje (editado):

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
**buffer**, que viene a ser el texto en memoria que genera Nvim en base a los
datos del fichero. En la siguiente entrega lo revisaremos más a fondo. Lo
relevante es que _por lo general_ un usuario de Neovim opera en base a buffers,
no ventanas o pestañas.

En fin, cuando hay más de una ventana en la pantalla actual podemos cambiar el
foco con `<C-w>` más la dirección de la ventana (`h`, `j`, `k` o `l`). Y si
queremos mover la ventana actual a otra posición podemos usar `<C-w>` y la
dirección en mayúscula (`H`, `J`, `K` o `L`).

## Rutas

En terminología Unix, `~` representa el directorio home de nuestro usuario. Por
ejemplo, en una configuración estándar si nuestro usuario se llama `foo`,
entonces `~/` representa la ruta `/home/foo/` en nuestro sistema de archivos.
Además, por lo general en sistemas Linux es habitual tener por defecto la
variable de entorno `XDG_CONFIG_HOME` apuntando a `~/.config/`. Podemos revisar
hacia dónde apunta, abriendo una terminal y revisando la salida del comando:

```sh
$ echo "$XDG_CONFIG_HOME"
```

Ergo, en base al mensaje de ayuda que acabamos de leer, Neovim va a buscar
nuestro archivo `init.lua` en `/home/<nombre de usuario>/.config/nvim/init.lua`.
Creemos a través de la consola la ruta y el archivo:

```sh
$ cd
$ mkdir .config/nvim
$ cd .config/nvim
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

Ahora veamos si Neovim está leyendo o no el fichero a través del clásico de la
programación: _¡Hola mundo!_ (Sí, _Hola mundo_ para configurar nuestro editor).
Utilizamos el siguiente código:

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
los cambios y cerrar la instancia actual de Neovim podemos usar `:qw` o la
opción con mucho más estilo: `ZZ` y mirar de manera condescendiente a los que
abren la línea de comandos para esto.

Tras volver a abrir Neovim tenemos esto:

![Hola mundo](hello-world.png)

Perfecto. Todo funcionando. Una vez comprobado limpiemos el archivo.

> Si en cualquier momento queremos volver a ver este o cualquier mensaje de la
> sesión actual podemos utilizar el comando `:messages` o `:mess`, y
> `:mess clear` para limpiarlos.

<!-- prettier-ignore-start -->
> Lua ofrece la función `print` que podemos utilizar en lugar a `vim.notify`,
> sin embargo, prefiero esta última porque permite agregar niveles de mensajes
> (como advertencias y errores) y además nos permite reemplazarlo cuando
> queramos por un proveedor personalizado.
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
_setup_ en base a ficheros (o módulos) de funcionalidad. El cómo ejecutar esta
división, al igual que en cualquier código, es un desafío de sutiles
implicaciones y allí su dificultad. No obstante, _mucha agua ha pasado bajo este
puente_, de modo que podemos aplicar la solución que muchos han encontrado al
respecto: **dividir para conquistar**.

Una ventaja que pasa desapercibida de este enfoque es que desde ya podemos poner
en práctica las diversas técnicas de navegación entre buffers y archivos. Y es
que en Neovim, el navegar por los diversos ficheros de un proyecto es una gozada
en comparación a otras IDE, sobretodo si lo potenciamos con algunos _plugins_
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

Y para que Neovim "lea" los ficheros `mappings.lua` y `settings.lua` utilicemos
la función `require` dentro de nuestro `init.lua`:

```lua
require("config.settings")
require("config.mappings")
```

A esta altura, espero que solo leyendo esas dos líneas de código ya estén
tecleando `:h require` en sus editores. ¿No? Bueno al menos háganlo por
curiosidad 🥲.

¿Todavía no? ¡Ya bueno!, no se preocupen que para eso estamos aquí. `require` es
una función Lua que opera bajo la abstracción de **módulos**. Básicamente, un
módulo Lua es un fichero `.lua` que el intérprete de Lua lee a través de la
función `require`. Tras la lectura del archivo, procesa su código y guarda el
resultado en memoria de tal modo que sus funciones, variables o configuraciones
queden a disposición para utilizarse o reutilizarse.

En el caso anterior, `settings` y `mappings` vendrían siendo nuestros módulos y
`config` sería un directorio contenedor que utilizamos para la organización del
código.

Creo que el sentido de cada uno de ambos módulos es autoexplicativo, pero en
cualquier caso, la idea es que dentro de `settings` agreguemos todas las
opciones genéricas de Neovim tales como columnas con números relativos, ancho
del texto por defecto, opciones de la barra de estado, etc.; y dentro de
`mappings` la idea es agregar todo lo referente a atajos y configuraciones de
teclado.

¡Uy pero miren la hora! Vamos a tener que dejar para el siguiente artículo el
llenado de ambos ficheros.

---

## Conclusiones

Hemos avanzado poco, lo sé, mí culpa, pero también es cierto que hemos abarcado
muchos elementos en paralelo. Prometo que las futuras entregas tendrán un
enfoque mucho más empírico y más código/configuraciones. Por ahora, y a modo de
conclusión, enumeremos las ideas clave que hemos abarcado:

- Abrir la documentación o ayuda de Neovim a través de la `command-line`
  (`:help <búsqueda>`).
- Comandos relacionados con las ventanas (`<C-w>`):
  - `<C-w>o` para agrandar una ventana dividida (con el foco en la ventana que
    queremos agrandar).
  - `<C-w>` más la dirección de la ventana a enfocar para cambiar el "foco de la
    ventana".
  - `<C-w>` más la dirección en mayúsculas para mover la ventana actual.
- Ruta de la configuración `init.lua` (`:h config`).
- Función `require`.
- Función `vim.notify`.
- Comando `:messages`/`mess`.

