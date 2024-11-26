---
title: '(WIP) Aventuras en Neovim II: Módulos y Fallbacks'
date: '2024-11-24 17:45 -0300'
permalink: /posts/Neovim-Modules
lang: es
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: Segundo capítulo de las aventuras en Neovim.
media_subpath: /assets/img/neovim/
image: nvim.png
---
# NeoVimturas II

En la entrega anterior abordamos los criterios y nos adentramos un poco en lo
básico de la configuración, de modo que ahora nos concentraremos principalmente
en la aplicación de esos conceptos y construiremos desde el principio nuestros
módulos y funciones personales en lua para ayudarnos en todo el proceso.
Continuemos entonces.

## Manejo de archivos

Si siguieron al pie de la letra el artículo anterior esto es lo que deberíamos
tener:

```
$ ls ~/.config/nvim
nvim/
└── init.lua
```

Eliminemos `init.lua` y hagamos uno nuevo a través de la consola:

```terminal
$ cd ~/.config/nvim
$ rm init.lua
$ nvim init.lua
```

Hermoso, volvimos al paso 1, pero ya no somos los mismos.

## El plan

Este es el plan:

```lua
require("config.settings")
require("config.mappings")
```

Ya conocemos lo que hace `require`, de modo que ya podríamos describir en
palabras las intenciones de esas líneas. ¿No? Bueno, no hay problema. En
concreto, dentro de la carpeta `config` vamos a crear los módulos `settings` y
`mappings`, e `init.lua` va a ser el encargado de llamar o ejecutar ambos
módulos. ¿Se nos escapa algo? Pues sí, Lua estila que los módulos se encuentren
dentro del directorio `lua`, de modo que este debiese ser el _path_ donde crear
nuestros módulos de configuración: `nvim/lua/config/settings.lua` y
`nvim/lua/config/mappings.lua`.

Evidentemente, mientras no exista esa ruta ni esos archivos, el requerir los
módulos va a producir un error al momento de iniciar una nueva instancia de
Neovim, lo que interrumpiría la configuración hasta el punto del primer fallo.
En esa situación tendríamos que arreglar el problema con la configuración por
defecto, o peor aún, dependiendo la naturaleza del error podríamos estar
teniendo mensajes de error a cada rato. El objecto de esta entrega es
precisamente el abordar este problema creando un módulo para robustecer un poco
la carga de nuestro propia configuración a modo de evitar que nos disparemos en
el pie y tengamos que estar arreglando cosas con
[Nano](https://nano-editor.org/) u otro editor.

Pero estamos lejos todavía; de momento esta es nuestra tarea:

1. Escribir los cambios en `init.lua`.
2. Crear los subdirectorios `lua` y `lua/config`.
3. Crear los módulos `settings` y `mappings` dentro de esa ruta.
4. Revisar que los módulos de ejecuten correctamente

### Buffers, ventanas y pestañas

El primer paso es bastante sencillo, abrimos la `command-line` con `:` y usamos
el comando `w` para escribir `init.lua`. Dado que abrimos Neovim mediante
`nvim init.lua`, `:w` es suficiente pues Neovim infiere que estamos trabajando
sobre `init.lua`:

```vim
:w
```

De querer especificar el nombre del archivo, un clásico "guardar cómo" de toda
la vida, simplemente pasamos a `write` la ruta al archivo como parámetro:

```vim
:w path/nombre
```

Si pasamos solo el nombre del archivo, se guarda en el `cwd` o _current working
directory_, que no es otra cosa que el directorio donde hemos ejecutado Neovim o
al que hayamos cambiado con `:cd`.

<!-- prettier-ignore-start -->
> En Unix, para referirnos al mismo directorio donde estamos se usa `./`, y para
> referirnos al directorio que contiene el directorio actual, o el directorio
> padre, se utiliza `../`.
{: .prompt-warning }
<!-- prettier-ignore-end -->

Conceptualmente, Neovim y Vim trabajan con la idea de `buffers`, que se refiere
al texto o datos de un fichero **en memoria**. Lo que hace el comando `write` es
simplemente escribir el contenido del buffer actual desde la memoria a un
archivo en el hardware.

Desde ya es importante definir y diferenciar los conceptos de `buffer`, `window`
y `tab`; y como no, la documentación hace un excelente trabajo al respecto. Si
buscamos `:h buffers` podemos leer el siguiente resumen:

> **Summary:**
>
> - A buffer is the in-memory text of a file.
> - A window is a viewport on a buffer.
> - A tab page is a collection of windows.

_(Recordemos que con `<C-w>c` cerramos la ventana actual)_

Por si acaso lo anterior les parece demasiado hermético y no acaba de cuajar, lo
podemos entender de la siguiente forma:

- El `buffer` es el texto que extrae Neovim después de leer cada archivo y sobre
  el cual hacemos todas nuestras ediciones.
- Las `ventanas` son lo que vemos en pantalla. "Cámaras" que nos permiten ver el
  contenido de un `buffer` específico. Podemos controlar a qué "apunta" cada una
  de las ventanas e incluso podemos tener más de una "apuntando" al mismo
  texto-buffer en ventanas independientes.
- Y finalmente las `tabs` o pestañas, que son simplemente una colección de
  ventanas.

#### Diferencias prácticas

La diferencia práctica más importante es que en Neovim es común trabajar en base
a buffers, podemos cambiar en la misma ventana rápidamente de un buffer a otro y
en caso necesario podemos dividir la pestaña actual en múltiples ventanas para
ver muchos archivos distintos al mismo tiempo, o incluso secciones distintas de
de un mismo archivo (útil cuando trabajamos con archivos grandes).

![Múltiples ventanas](windows_and_buffers.png)

En la imagen vemos 3 ventas:

- La superior izquierda muestra el contenido del buffer del archivo `init.lua`
- Tanto la inferior izquierda y la pantalla a la derecha muestran el contenido
  del buffer `utils.loaders.lua` pero "apuntando" a distintas secciones del
  texto.

### Manipulación de carpetas y archivos

Continuemos con punto número 2. Recordemos que lua espera que todos los módulos
se encuentren dentro del directorio `lua`, y nuestra primera línea
(`require("config.settings")`) carga el módulo `settings` dentro del directorio
`config`. Entonces debemos crear esa ruta y nuestros módulos lua dentro de ella.
Para hacer esto tenemos muchas alternativas, pero exploremos dos alternativas
dentro de Neovim: Con **Netrw** y al estilo **Ex-commands**.

¿Recuerdan que hablamos de las trampas conceptuales de Neovim? Bueno, si no
quedó claro el concepto, aquí tenemos la primera de este artículo: Ex vs Ex.

## Ex vs Ex

**No debemos confundir Ex con Ex**. ¿Qué? Sí, como lo leen, dos cosas iguales se
refieren a dos cosas distintas; bueno, 3 en realidad, pero la última la voy a
mencionar solo de manera anecdótica.

La primera acepción se refiere al comando `:Ex` (de _Explorer_), que ejecuta el
plugin nativo `Netrw`, una especie de navegador de archivos integrado en
Neovim/Vim. La segunda acepción se refiere a los **Ex-commands**, que son los
comandos que introducimos en la **command-line**, por ejemplo `:w`/`:write`,
`:q`/`:quit` y —espero no se confundan aquí— al propio comando `:Ex`. Sí, `Ex`
es el comando **Ex-command** para ejecutar el plugin **Netrw**. La tercera
acepción se refiere al **Ex-mode**, un modo que la mayoría de las veces parecía
un error maligno en la matrix y que en versiones anteriores de Neovim
conjurábamos pulsando por error con la tecla `Q`. Entonces, la conclusión es no
confundir `:Ex` con `Ex-commands` ni con `Ex-mode`.

Exploremos primero la opción con **Netrw**:

## Netrw

```vim
:Ex
```

---

## Conclusiones

Conclusiones

