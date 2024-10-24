---
title: 'Neovim Adventures: Let''s start from the init'
date: '2024-10-24 20:02 -0300'
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: First chapter in Neovim Adventures.
media_subpath: /assets/img/20241024/
image: nvim.png
---
# Aventuras en Neovim

## Generalidades y Objetivos

Llevo algunos años utilizando casi exclusivamente [Neovim](https://neovim.io/)
para la edición de textos y código en mi sistema y a lo largo de este tiempo he
ido acumulando algunas recetas, ideas y conceptos que me han llevado a una
organización bastante estable y flexible de mi entorno de desarrollo.

No todas estas ideas han sobrevivido, la gran mayoría de hecho las he eliminado
cuando no se integraron orgánicamente a mi flujo de trabajo quedando
inutilizadas por meses. Lo único que queda de ellas es el aprendizaje sobre como
escribirlas en lua, y el conocimiento de las distintas formas que ofrece Neovim
para realizar ciertas tareas. El registrar esta experiencia es un objetivo
importante de esta serie de artículos de modo que pueda ser útil para los que
recién comienzan con su propia _NeoVimtura_.

Si estás leyendo esto asumo dos cosas. Primero, que **te interesa** —_incluso
entretiene_— el ajuste de tus herramientas de trabajo, cual carpintero afilando
y ajustando su cepillo para que cante en armonía consiguiendo esa viruta
semitransparente perfecta; y en segundo lugar asumo que **no quieres perder
tiempo reinventando la rueda**. Que no nos ocurra, que por ahorrarnos una
lectura de minutos perdamos horas o incluso _días_ en torno problemas ya
resueltos y al alcance de una función, flag o parámetro.

Otro objetivo de esta serie, es por un lado ordenar el montón de ideas y
exploraciones que he ido desarrollando, experimentado e integrando; y por otro,
disponerlas en un formato didáctico con un poco más de profundidad en cuanto al
por qué de las cosas en lugar de solo hacer un tutorial o una receta a seguir.

Me interesa poder trasmitir un enfoque más profundo sobre lo que es el
desarrollo de nuestras herramientas, el tratar de aportar al "¿Por qué quiero
hacer esto?" o el más fundamental "¿Será esta la mejor forma de abordar el
problema? A fin de cuentas, el sacarle filo, ajustar de una pieza tan central
para todos los que trabajamos con textos día a día.

En fin, **¡manos a la obra!**

## El plan de ruta

_A priori_, en esta serie abordaré los siguientes puntos:

1. Inicio y guía (este artículo)
2. Teclas y criterios.
3. Organización y manejo de errores.
4. Utilidades. Personalizando la navaja suiza

Todos estos puntos están sujetos a ajuste, pero en términos generales este es el
mapa principal que planteo seguir.

La metodología utilizada va a depender del punto a tratar, pero más que nada
será:

- Plantearnos preguntas.
- Plantearnos meta-preguntas (el cuestionarnos si estamos o no haciendo la
  pregunta correcta).
- Explorar qué nos dice la documentación de Neovim al respecto.
- Revisión de las posibilidades.
- Escribir el código.

Siguiendo estos pasos espero poder ayudar a tener una noción más o menos clara
del procedimiento necesario para construir nuestras propias soluciones que se
adapten específicamente a nuestras necesidades —o lo que creemos necesitar—. Y
por supuesto que ello no involucre el rehacer completamente el resto de nuestra
configuración o volverla inestable.

## Lo que no abordaré

- Movimientos Vim: Sí, editar un string mediante `ci"` es hermoso, pero ya se ha
  escrito demasiado al respecto. Inevitablemente serán abordados, pero no como
  el objetivo final, sino como un medio y no en profundidad.
- Funciones básicas como cambiar de modos, escribir texto, guardar, salir, etc.
  Por completitud las mencionaré, pero para eso mejor seguir el tutor.
- Vimscript: En los días de Vim nunca me gustó y siempre lo encontré confuso. A
  día de hoy esta sensación se mantiene y solo diré que prefiero aprender Lisp y
  cambiarme a Emacs a adentrarme en él. Gracias Lua.
- Conceptos de programación como funciones, variables, tablas, API, etc. Aquí
  asumiré cierto conocimiento básico al respecto. Lua por lo demás es un
  lenguaje bastante sencillo, de modo que es una perfecta oportunidad para
  adentrarse en este mundo.

---

Con estas aclaraciones generales, ahora sí, procedamos al meollo de este
capítulo: El inicio, nuestro `init.lua`.

## Init

Para comenzar esta serie de aventuras, creo que lo mejor es hacerlo en partida
doble, es decir, como primera tarea aprendamos cuál es la estructura de
directorios y ficheros que permite Neovim y al mismo tiempo veamos cómo Neovim
por sí mismo nos puede ayudar.

Entonces, desempolvemos nuestra terminal favorita
([Konsole](https://konsole.kde.org/) en mi caso) y abramos Neovim:

```sh
$ nvim
```

Asumo que no tenemos ningún fichero ni directorio de configuración. A futuro,
una forma de volver a abrir Neovim en este estado por defecto es mediante
`nvim --clean`, muy útil en especial cuando algo no funciona y queremos conocer
si es un problema en el sistema operativo, el propio Neovim, o nuestra
configuración.

![Interfaz por defecto](default-nvim.png) _La interfaz al vacío, tal como la
belleza de una piedra al natural conteniendo todas las formas y esculturas_

Y la pregunta sería ¿por dónde partimos? Bueno, lo primero sería aprender a
navegar por la ayuda.

Para abrir la ayuda de Neovim tenemos que abrir la `command-line` y ejecutar el
comando `help`. En terminología vim/nvim, ejecutar `:help<CR>`. ¿Qué quiere
decir esto?:

> Decir `:help` equivale a:
>
> 1. Abrir la `command-line` (pulsar la tecla `:`).
> 2. Teclear el comando `help` (opcionalmente podemos escribir solo `h`).
> 3. Pulsar `<CR>` (la tecla `enter`)

Y con esto abrimos la ventana de ayuda (`<F1>` también funciona):

Y así de simple, ya nos encontramos con el primer desafío: **¡¿Tengo que leer la
documentación ocupando solo la mitad de la pantalla?!** ¡Pues no! Precisamente
de esto se trata esta serie. De modo que antes de solucionarlo, invito a
reflexionar un poco sobre este punto.

![Help!](help.png) _Tanto desperdicio de espacio..._

La pregunta clave aquí es **¿Qué queremos hacer?**, y la respuesta no es obvia
como puede parecer. _"Quiero que la pantalla de ayuda ocupe toda la pantalla y
no solo la mitad"_ dirán algunos. Pues sí y no, esta es la primera
[trampa x-y](https://xyproblem.info/) a la que nos enfrentamos. Lo que realmente
queremos no es que la **ventana** de ayuda ocupe toda la pantalla, sino el poder
leer la documentación cómodamente.

¡Retórica! Podrán decir algunos no sin un dejo de razón, no obstante el cambio
de foco es importante. Es el contexto es lo que determina nuestra comodidad, y
recién nos estamos adentrando en esta herramienta. ¿Tenemos contexto suficiente
para cuestionar esta decisión? ¿En qué estaban pensando los desarrolladores
cuando se decidieron por este comportamiento por defecto? No lo sabemos. ¿Será
la ayuda en pantalla completa cómoda en los casos de utilización más usuales?
Tampoco lo sabemos. No al menos mientras estemos en este proceso de
familiarización y ciertamente no cuando apenas sabemos organizar y movernos
entre ventanas.

Por lo tanto nuestro primer instinto, el de _"voy a configurar la pantalla de
ayuda para que se abra a pantalla completa por defecto"_, obedece al mismo
principio de optimización prematura, y por ende considerémoslo como un error en
este momento, quizás efectivamente sea el ajuste adecuado, pero carecemos de la
exploración suficiente para determinarlo. Su ajuste puede suponer una mayor
pérdida de tiempo que el beneficio u comodidad que estamos buscando. Una futura
_configuración fantasma_ que nos pesa e incomoda, per

¿Entonces como arreglamos esto? No lo hacemos. Simplemente pulsamos `<C-w>o`
para hacer de la ventana seleccionada la única en la pantalla.

Para escribir combinaciones de teclado

Así se escriben las combinaciones de teclas en Neovim/Vim. Básicamente quiere
decir que pulsemos las teclas `ctrl+w`, seguidas de la tecla `o`. Como ayuda
mnemotécnica, podemos entender el `<C-w>` como _"cosas de ventana"_
(**w**indows) y la tecla `o` de **o**pen (abrir). Más adelante veremos qué
quiere decir _ventana_ exactamente y la diferencia de esta con _buffers_ y
_tabs_. Todo esto está en el manual, pero no nos adelantemos.

Lo importante ahora, es que si pulsamos `<C-w>o` teniendo el foco sobre la ayuda
(o el foco sobre la _ventana_ que tiene el _buffer_ de ayuda), ya tendremos la
documentación a pantalla completa. Pero lamentablemente no con el contenido que
estamos buscando. Queremos la documentación acerca de cómo configurar Neovim.
Intentemos entonces encontrar esta sección. Intentemos con
`:help configuration`.

> Cuando hay más de una ventana en la pantalla actual, podemos cambiar el foco
> con `<C-w>` más la dirección de la ventana (`h`, `j`, `k` o `l`). {:
> .prompt-tip }

¡Oh! ¿¡Qué rayos es `lsp-handler-configuration`!? Bueno, evidentemente no es lo
que buscamos. Intentemos con `:help config`. ¡Ah!, ahora sí estamos hablando. De
aquí se lee (editado):

> A file containing initialization commands is generically called a "vimrc" or
> config file...
>
> The config file is located at:
>
> - Unix → ~/.config/nvim/init.lua
> - Windows → ~/AppData/Local/nvim/init.lua
> - XDG → $XDG_CONFIG_HOME/nvim/init.lua

Entonces, generemos este archivo. En terminología Unix, `~` representa el
directorio home de nuestro usuario. Por ejemplo, si nuestro usuario se llama
`foo`, entonces `~/` representa la siguiente ruta en nuestro sistema de
archivos: `/home/foo/`. Además, por lo general en sistemas Linux es habitual
tener por defecto la variable de entorno `XDG_CONFIG_HOME` apuntando a
`~/.config/`. Ergo en nuestro entorno de shell, `nvim` va a buscar el archivo
`init.lua` en `/home/<nombre de usuario>/.config/nvim/init.lua`:

```sh
cd # Lo mismo que cd /home/<nombre de usuario>
mkdir .config/nvim
cd .config/nvim
touch init.lua
```

¡Y a editar!

```sh
nvim init.lua
```

Y tenemos nuestro glorioso archivo de configuración:

![Fichero vacío en Neovim](base-init.png) _¡Nuestra hermosa página en blanco!
¡Véanla! ¡Tan llena de posibilidades!_

Ahora veamos si Neovim está leyendo o no el fichero a través de el clásico
_¡Hola mundo!_ (sí, hola mundo para configurar nuestro editor). Utilizamos el
siguiente código:

```lua
-- Esto es un comentario que debe comenzar con --

-- La siguiente función nos permite notificar al usuario con un mensaje:
vim.notify("¡Hola mundo!")
```

Que ingresamos cambiando del `normal-mode` al `insert-mode` con la tecla `i`.
Una vez escrito, volvemos al modo normal con `<Esc>`, guardamos los cambios y
cerramos Neovim pulsando `:qw` o `<ZZ>`.

Volvemos a ejecutar Neovim:

![Hola mundo](hello-world.png)

> Si en cualquier momento queremos volver a ver este o cualquier mensaje podemos
> utilizar el comando `:messages` o `:mess`. {: .prompt-tip }

Perfecto. Todo funcionando. Es necesario aclarar que lua ofrece la función
`print` que podemos utilizar en lugar a `vim.notify`. Sin embargo, prefiero esta
última porque permite agregar niveles de mensajes (advertencias, errores) y
según la documentación se puede reemplazar por un proveedor personalizado.

### Llenando nuestro `init.lua`

El cómo organizar nuestra configuración es un tema que abordaremos más adelante,
pero básicamente tenemos 2 opciones, o agregamos toda nuestra configuración a un
único `init.lua` o separamos los distintos ámbitos de nuestra configuración en
múltiples archivos.

La mayor ventaja de un único fichero de configuración es la facilidad que
tendremos al manejarlo; hacer respaldos o copiarlo a otro sistema sera muy
sencillo. Sin embargo creo que como modelo, e incluso práctica de programación,
es mejor tener los distintos elementos separados y agrupados por funcionalidad o
conceptos acordes.

