---
title: 'Neovimturas: El arte de sobrevivir a nuestras configuraciones temerarias'
date: '2025-01-08 17:01 -0300'
permalink: /posts/Neovim-Config
lang: es
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: Construyendo un módulo para cargar configuraciones de respaldo
media_subpath: /assets/img/neovim/
image: nvim.png
---
# Aventuras en Neovim

## Contexto y objetivos

¿Cuántas veces hemos por error agregado ese carácter maligno que rompe nuestra
configuración en mil pedazos? Un simple `rr` inadvertido sobre un paréntesis y
¡sorpresa! ¡A comerse un kilo de mensajes con errores en el momento más
inoportuno!

En el presente artículo propongo construir una solución al respecto, un módulo
que nos ayude no solo a manejar los errores de nuestra propia configuración,
sino que también haga más amable el proceso de corregirlos.

Para ello, esta red de seguridad debería cumplir las siguientes funciones:

- Cargar nuestros módulos de configuración de forma normal
- En caso de error:
  - Cargar automáticamente una configuración fallback
  - Darnos información de dónde está el o los problemas
  - Preguntarnos si abrir o no los archivos con problemas para su edición

¿Suena bien no? ¡Pues vamos a escribirlo!

---

## Enfoque:

A medida que vamos personalizando nuestra experiencia con Neovim, probablemente
iremos acumulando código para simplificarnos las cosas. Por ejemplo, el
[típico wrapper](https://github.com/polirritmico/neovim.conf/blob/bd78a4eb5d97c890bd10a685993bdee3a64ff182/lua/utils/config.lua#L58)
a `vim.keymap.set`.

Un estilo estándar que podemos observar en las configuraciones de Neovim más
avanzadas, es el ir agrupando ese código o funcionalidad dentro de un módulo de
utilidades; el típico módulo `utils`. Pensando en nuestro yo futuro —y si no lo
han hecho ya—, sugiero el incorporar este estilo o al menos otro similar para
organizar mejor nuestra configuración y hacerla más mantenible en el proceso.

### Estado actual

Ahora, veamos una configuración estándar con la siguiente estructura:

```terminal
$ cd ~/.config
$ tree nvim/
nvim/
├── init.lua
└── lua
    ├── config
    │   ├── mappings.lua
    │   └── settings.lua
    ├── plugins
    │   └── etc...
    └── utils
        └── etc...
```

En este setup, a través de `init.lua` se cargan los módulos de `settings`,
`mappings` y `plugins`. Todo normal:

```lua
-- nvim/init.lua
require("config.settings")
require("config.mappings")
require("plugins")
```

El problema con este enfoque en relación a construir una configuración capaz de
soportar nuestra propia inoperancia, es que cualquier error va a producir el
caos y terminaremos con los ajustes por defecto. Dependiendo el caso, esto puede
ser una situación bastante empalagosa, pues no solo estaríamos lidiando con el
error, sino que lo estaríamos haciendo sin el acceso a nuestros ajustes
fundamentales.

## 1. Nuestra configuración de respaldo

Por ejemplo, comentaré 2 ajustes básicos para mí. El primero, simplemente
exigido por mi memoria muscular:

```lua
-- lua/config/mappings.lua
vim.opt.langmap = "ñ:,Ñ\\;"
```

> Para los curiosos, en los teclados ISO-ES, a la derecha de la tecla `l` está
> la tecla `ñ` y no `;`. De modo que con ese ajuste no solo habilito la tecla
> `ñ` en el modo normal, sino que la configuro a `:` en lugar del `;`. Un 2x1
> (habilitar la tecla `ñ` e invertir `;` con `:`).
>
> ![Teclado ISO-ES](iso-es.png)

Sin esto, cada vez que presiono la tecla `ñ` no ocurre nada y entonces cosas tan
habituales como `ñw` (`:w`) seguido de `ZQ` no tendrían el efecto esperado 🥲.

Como segundo ajuste fundamental para mí, es el habilitar los números de línea
relativos (para hacer movimientos del tipo `[count]`+`j`/`l`):

```lua
-- lua/config/settings.lua
opt.relativenumber = true -- Show relative line numbers
opt.number = true -- Shows the current line number instead of 0
```

El no tener cualquiera de estos ajustes es una completa molestia para mí. De
modo que en esta configuración fallback, sí o sí deberían estar presentes.

Evidentemente aquí cada uno debería ajustar el contenido de su configuración
fallback a medida. Sugiero tratar de aproximarla con un enfoque minimalista y
robusto.

Para la estructura de esta configuración fallback, repliquemos la misma
estructura de nuestra configuración normal, aunque perfectamente a diferencia de
lo que hago aquí, se podría agrupar todo en un único módulo. En cualquier caso,
creemos el directorio `fallback` e incluyamos `settings.lua` y `mappings.lua`
dentro:

```terminal
$ mkdir lua/config/fallbacks
```

```lua
-- Fallback settings (lua/config/fallback/settings.lua)
opt.relativenumber = true -- Show relative line numbers
opt.number = true -- Shows the current line number instead of 0
```

Y también:

```lua
-- Fallback mappings (lua/config/fallback/mappings.lua)
vim.opt.langmap = "ñ:,Ñ\\;"
```

Ok, todo listo. Asumo que cada uno ya tiene claro lo que va dentro de su
configuración fallback. Dado que ya definimos el _qué_, ahora sólo queda
concentrarnos en el _cómo_.

## 2. Nuestro módulo fallback

Para la carga de la configuración, básicamente debemos implementar un wrapper
sobre las llamadas a `require`, o más específicamente, un wrapper en torno a
`require` utilizando [pcall](<https://neovim.io/doc/user/luaref.html#pcall()>).
Del error que captura `pcall`, podremos obtener la información necesaria y
actuar en consecuencia.

### El módulo

Ha llegado el momento de sacar a relucir nuestras habilidades con lua. Veamos la
forma básica del código (en `lua/utils/loaders.lua`):

```lua
---Helper functions used to safely load Neovim config modules.
---@class UtilsLoader
local Loaders = {}

---@param module string
function Loaders.load_config(module) end

---@param fallbacks? boolean `true` to load fallback settings if errors are found.
---@return boolean -- Returns `true` if no errors are detected.
function Loaders.check_errors(fallback) end
```

De aquí podemos inferir la idea, en lugar de utilizar
`require("config.mappings")` utilizaremos `load_config("config.mappings")` y
tras ello revisamos si hubo o no algún error con `check_errors()`:

```lua
-- nvim/init.lua

-- require("config.settings")
-- require("config.mappings")
-- require("plugins")

local utils = require("utils")

utils.load_config("config.settings")
utils.load_config("config.mappings")
check_errors(true)
utils.load_config("config.plugins")
utils.check_errors()
```

¿Por qué separar la carga de la resolución de errores? Porque son 2 tareas
distintas. Primero, queremos intentar cargar la configuración estándar. Luego,
en lugar de lanzar el error inmediatamente seguimos intentando cargar los
siguientes módulos, si estos módulos o parte de ellos funcionan correctamente,
genial; la configuración actual es más cercana a lo que buscamos a pesar de
haber encontrado errores. Segundo, al haber ido recolectando los errores, si es
que hay más de un módulo con problemas podemos informarlo de una vez y no de uno
en uno; haciendo más comprehensiva la carga y obteniendo más información en una
única ejecución.

### load_configs

Revisemos la función `load_configs`. Recolectaremos los errores en una tabla que
llamaremos `catched_errors`. La creamos fuera de la función:

```lua
---@type table Collection of errors detected by `load_config` (if any).
Loaders.catched_errors = {}

---@param module string
function Loaders.load_config(module)
  local ok, call_return = pcall(require, module)
  if not ok then
    print("- Error loading the module '" .. module .. "':\n " .. call_return)
    table.insert(Loaders.catched_errors, module)
  end
  return call_return
end
```

### check_errors

Ahora tenemos la información de los errores, pero nos falta el qué hacer con
ella. De eso se encargará la función `check_errors`:

```lua
---@param fallbacks? boolean `true` to load fallback settings if errors are found.
---@return boolean -- Returns `true` if no errors are detected.
function Loaders.check_errors(fallbacks)
  -- Primero el camino feliz:
  if #Loaders.catched_errors == 0 then
    return true
  end

  -- Si los problemas llegan, cargamos la configuración fallback:
  if fallbacks then
    vim.notify("Loading fallback configs.", vim.log.levels.ERROR)
    require("config.fallback.settings")
    require("config.fallback.mappings")
  end

  -- Y finalmente informamos al usuario:
  local msg = string.format("%s\nDetected errors:\n", string.rep("-", 80))
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  return false
end
```

## Revisando

¿Por qué cargamos _toda_ la configuración fallback y no sólo lo que ha fallado?

Ese fue mi primer enfoque, pero dependiendo lo que tengamos tanto en `settings`
como en `mappings` la configuración podría quedar en un estado inestable, por
ejemplo, si asociamos algún atajo con alguna función de `utils` o en base a un
setting específigo.

En segundo lugar y más importante, no perdamos el foco. El objetivo aquí es
detectar que ha habido un problema y tener un entorno relativamente cómodo para
solucionarlo, no el tener una instancia de Neovim que funcione con la mayor
cantidad de features posibles.

También vale la pena añadir, el no menos relevante punto de que al simplificar
la lógica también reducimos la cantidad de código a mantener y con ello todos
los beneficios que ello implica (performance, mantenibilidad, legibilidad, etc).

En fin, con `load_config` y `check_errors` ya tenemos la funcionalidad básica
que buscábamos. No obstante, vamos un pequeño paso más allá y agreguemos el
último punto de nuestros objetivos: el preguntar si abrir o no el fichero con
problemas. Como la mayoría de las veces con Neovim, esto es bastante sencillo a
través de `vim.fn.input`:

```lua
function Loaders.check_errors(fallbacks)
  -- etc.
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  if vim.fn.input("Attempt to open offending files for editing? (y/n): ") == "y" then
    print(" ") -- (creo que añadí esto para que el mensaje quedara mejor formateado)
    print("Opening files...")

    local neovim_path = vim.fn.stdpath("config") .. "/lua/config"
    for _, error in pairs(Loaders.catched_errors) do
      -- Obtenemos el path del archivo con problemas desde el mensaje de error
      if error:sub(1, 1) ~= "/" then
        error = string.format("%s/lua/%s.lua", neovim_path, str:gsub("%.", "/"))
      end

      local path = get_path_from_error(error)

      if vim.fn.findfile(path) ~= "" then
        vim.cmd("edit " .. path)
      end
    end
  end

  return false
end
```

Podemos refactorizar un poco el nuevo código, moviendo la lógica de obtener el
path a su propia función, `get_path_from_error`:

```lua
local function get_path_from_error(str)
  if str:sub(1, 1) == "/" then
    return str
  end

  -- Este path lo utilizo en varios lugares distintos, de modo que lo tengo
  -- definido como una variable global al principio de mi init.lua con el nombre
  -- de NeovimPath junto a otras variables globales que luego valido con assert.
  local neovim_path = vim.fn.stdpath("config") .. "/lua/config"

  return string.format("%s/lua/%s.lua", neovim_path, str:gsub("%.", "/"))
end
```

Yo la agregaré dentro de `check_errors`, pero perfectamente podría estar como
una variable local dentro de `loaders`.

## Juntando las partes

Incorporando todo esto en el código final, ya que somos personas civilizadas,
aprovechamos de agregar las anotaciones correspondientes para ayudarnos con
nuestro servidor lsp:

```lua
---Helper functions used to load Neovim config modules.
---@class UtilsLoader
local Loaders = {}

---@type table Collection of errors detected by `load_config` (if any).
Loaders.catched_errors = {}

---_Helper function to load the passed module._
---
---If the module returns an **error**, then print it and store it in the
---`catched_errors` table.
---@param module string Name of the module to load.
---@return any call_return Return of the require module call (if any).
function Loaders.load_config(module)
  local ok, call_return = pcall(require, module)
  if not ok then
    print("- Error loading the module '" .. module .. "':\n " .. call_return)
    table.insert(Loaders.catched_errors, module)
  end
  return call_return
end

---_Helper function to handle detected `load_config` errors_
---
---If an error is detected it will load the fallback settings and **ask the
---user** to open or not the offending file.
---@param fallbacks? boolean `true` to load fallback settings if errors are found.
---@return boolean -- Returns `true` if no errors are detected.
function Loaders.check_errors(fallbacks)
  if #Loaders.catched_errors == 0 then
    return true
  end

  local function get_path_from_error(str)
    if str:sub(1, 1) == "/" then
      return str
    end

    return string.format("%s/lua/%s.lua", NeovimPath, str:gsub("%.", "/"))
  end

  if fallbacks then
    vim.notify("Loading fallback configs.", vim.log.levels.ERROR)
    require("config.fallback.settings")
    require("config.fallback.mappings")
  end

  local msg = string.format("%s\nDetected errors:\n", string.rep("-", 80))
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  if vim.fn.input("Attempt to open offending files for editing? (y/n): ") == "y" then
    print(" ")
    print("Opening files...")
    for _, error in pairs(Loaders.catched_errors) do
      local path = get_path_from_error(error)

      if vim.fn.findfile(path) ~= "" then
        vim.cmd("edit " .. path)
      end
    end
  end
  return false
end

return Loaders
```

¿Pero cuándo la versión final de un código ha sido realmente la final? Los
lectores más atentos se habrán dado cuenta que en realidad no hemos solventado
el problema al 100%. Veamos nuestro `init.lua`:

```lua
local utils = require("utils")

utils.load_config("config.settings")
utils.load_config("config.mappings")
check_errors(true)
utils.load_config("config.plugins")
utils.check_errors()
```

¿Y qué ocurre dentro de `require("utils")`? ¿Qué pasa si hay un error allí
dentro?

Pues fallaría, y todo lo que hemos construido estaría de adorno.

## ¿De vuelta al principio?

No. Este no es como el típico artículo por subscripción en _Medium_ que nos
abandona al pensar un poco por nuestra cuenta y salirnos unos milímetros de su
marco principal (😗🎶). Recordemos que ya tenemos nuestro módulo `UtilsLoader`
funcionando y por lo tanto no queda más que aplicar nuestro `loader` para, en
pleno estilo _Inception_, cargar el propio `utils`.

<!-- prettier-ignore-start -->
![Ouroboros](ouroboros.jpg)
_Cargando el cargador del cargador_
<!-- prettier-ignore-end -->

Este es el contenido de mi archivo `utils/init.lua` que se encarga de incorporar
los distintos módulos en uno (editado para no distraernos):

```lua
---A collection of custom helper functions.
---@class Utils
---@field config UtilsConfig
---@field custom UtilsCustom
---@field helpers UtilsHelpers
local Utils = {}

Utils.config = require("utils.config")
Utils.custom = require("utils.custom")
Utils.helpers = require("utils.helpers")

return Utils
```

De la misma forma que en nuestro `init` principal hemos reemplazado los llamados
de `require` con los de nuestro propio módulo, aquí podemos hacer lo mismo e
incluso ir un poco más allá agregando _la carga de los cargadores_.

Lo primero, es cargar nuestro módulo `loaders` y reemplazar los llamados a
`require`:

```lua
---@class Utils
local Utils

local loaders = require("utils.loaders")

Utils.config = loaders.load_config("utils.config")
Utils.custom = loaders.load_config("utils.custom")
Utils.helpers = loaders.load_config("utils.helpers")

-- etc.
return Utils
```

Pero, ¿por qué detenernos allí? ¿y si tenemos un problema dentro del propio
`loaders`? De nuevo, `pcall` al rescate:

```lua
local Utils

local ok, loaders = pcall(require, "utils.loaders")
if not ok then
  vim.cmd("edit " .. NeovimPath .. "/lua/utils/loaders.lua")
  error(string.format("Error in 'utils.loaders':\n\n%s\n", loaders))
end

Utils.load = loaders.load_config
Utils.check_errors = loaders.check_errors

-- load the utils modules
Utils.custom = loaders.load_config("utils.custom")
Utils.config = loaders.load_config("utils.config")
Utils.helpers = loaders.load_config("utils.helpers")
assert(loaders.check_errors())

return Utils
```

<!-- prettier-ignore-start -->
> Esta vez dejaré el código directamente en el módulo, pero si quieren pueden
> crear una función. En ese caso recuerden hacer el llamado correspondiente.
{: .prompt-info }
<!-- prettier-ignore-end -->

¡Excelente! Ahora tenemos todas nuestras cargas protegidas y finalmente tenemos
nuestros propios zapatos blindados a prueba de errores.

Si me permiten, ahora haré una pequeña refactorización para agrupar la carga de
módulos en una única función (excluyendo `loaders` por supuesto) y separar los
pasos de cargar `utils`, que solo cargaría los `loaders` del resto de módulos de
utilidades:

```lua
---A collection of custom helper functions.
---@class Utils
---@field config UtilsConfig
---@field custom UtilsCustom
---@field helpers UtilsHelpers
local Utils = {}

---Load the loaders
local ok, loaders = pcall(require, "utils.loaders")
if not ok then
  vim.cmd("edit " .. NeovimPath .. "/lua/utils/loaders.lua")
  error(string.format("Can't load 'utils.loaders':\n\n%s\n", loaders))
end

Utils.load = loaders.load_config
Utils.check_errors = loaders.check_errors

function Utils.load_utils()
  Utils.config = Utils.load("utils.config")
  Utils.custom = Utils.load("utils.custom")
  Utils.helpers = Utils.load("utils.helpers")

  assert(loaders.check_errors())
end

return Utils
```

Hermoso.

Con esto, debemos recordar que ahora además de usar `require("utils")` debemos
inicializar el módulo con `require("utils").load_utils()`. Además, con esta
función de `load_utils`, a futuro podremos agregar fácilmente parámetros a la
carga de nuestras utilidades. Por ejemplo, añadir algún profiler, levantar una
sesión DAP o simplemente evitar la carga de ciertos módulos en determinados
contextos.

Ahora sólo queda ajustar nuestro `init.lua` y a disfrutar:

```lua
local utils = require("utils")
utils.load_utils()

utils.load("config.settings")
utils.load("config.mappings")
assert(utils.check_errors(true))
utils.load("config.lazy")
utils.check_errors()
```

```terminal
$ nvim
```

Bueno, espero que haya sido una lectura interesante y que alguna de las ideas
presentadas sean de utilidad especialmente para aquellos, que temerosos de
arruinar su configuración, dejan de explorar las opciones que nos ofrece este
fantástico "editor".

¡Suerte!

---

Si les da curiosidad, utilizo esta técnica en mi configuración de modo que
pueden revisarla [aquí](https://github.com/polirritmico/neovim.conf). Es
básicamente lo mismo que he expuesto aquí, pero con ciertos añadidos como lidiar
con specs de lazy y opciones de debugging.

