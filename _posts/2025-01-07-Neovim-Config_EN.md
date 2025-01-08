---
title: 'Neovimtures: The Art of Surviving Our Reckless Configurations'
date: '2025-01-08 17:01 -0300'
permalink: /posts/Neovim-Config
lang: en
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: Building a module to load fallback configurations
media_subpath: /assets/img/neovim/
image: nvim.png
---
# Adventures in Neovim

## 1. Context and Objectives

How many times we mistakenly added that malicious character that breaks our
configuration into a thousand pieces? A simple `rr` unnoticed over a bracket and
surprise! Let's eat a chunk of error messages at the most inopportune moment!

In this article I propose to build a solution to this, a module that will help
us not only to handle the errors of our own config, but also to make the process
of correcting and dealing with them much more friendly.

To this end, this 'safety net' should fulfill the following tasks:

- load our configuration as usual.
- In case of error:
  - Automatically load a fallback configuration.
  - Give us information on where are the problems located
  - Ask us whether or not to open the offending file for editing

Sounds good? Well, let's code it!

---

## 2. Approach:

As we customize our Neovim experience, we will probably accumulate code to
simplify things for us. For example, the
[typical wrapper](https://github.com/polirritmico/neovim.conf/blob/bd78a4eb5d97c890bd10a685993bdee3a64ff182/lua/utils/config.lua#L58)
to `vim.keymap.set`.

A standard way we can observe in more advanced Neovim configurations, is to
group that code or functionality inside a utilities module; the typical `utils`
module in our `lua` path. Thinking about our future selves -and if you have not
already done so-, I suggest incorporating this style or any other to better
organize our configuration and make it more maintainable in the process.

### Current Status

Now, let's look at a standard example config with the following structure:

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

In this setup, through `init.lua` the `settings`, `mappings` and `plugins`
modules are loaded. All standard:

```lua
-- nvim/init.lua
require("config.settings")
require("config.mappings")
require("plugins")
```

The problem with this approach to building a configuration capable of supporting
out own inoperability is that any error will lead to chaos and we will end up
with the default settings. Depending on the case, this can be a rather cloying
situation, as not only would we be dealing with the error, but we would be doing
so without access to our fundamental settings.

## 3. Our fallaback config

For example, I will comment on 2 basic settings for me. The first, simply
demanded by my finger's memory:

```lua
-- lua/config/mappings.lua
vim.opt.langmap = "ñ:,Ñ\\;"
```

> For the curious, on `ISO-ES` keyboards, to the right of the `l` key is the `ñ`
> key and not `;`. So with that setting I not only enable the `ñ` key in normal
> mode, but I set it to `:`. A 2x1 (enable the key plus "invert" `;` with `:`)
>
> ![ISO-ES Keyboard](iso-es.png)

Without this, every time I press the `ñ` key (the equivalent for `:`) nothing
happens and then such usual things as `:w` (`ñw`) followed by `ZQ` would not
have the expected effect 🥲.

As a second fundamental setting for me, is to enable relative line numbers (to
do `[count]`+`j`/`l` type moves):

```lua
-- lua/config/settings.lua
opt.relativenumber = true -- Show relative line numbers
opt.number = true -- Shows the current line number instead of 0
```

Not having any of these settings is a complete nuisance to me. So in the
fallback configuration, yes or yes they should go.

Obviously here everyone should adjust the content of their fallback
configuration to suit themselves. I suggest trying to approximate it with a
minimalistic and robust approach.

For the structure of this fallback configuration, let's replicate the same
structure of our normal config, although perfectly unlike what I do here, you
could group everything in a single module. In any case, let's create the
`fallback` directory and include `settings.lua` and `mappings.lua` inside:

```terminal
$ mkdir lua/config/fallbacks
```

```lua
-- Fallback settings (lua/config/fallback/settings.lua)
opt.relativenumber = true -- Show relative line numbers
opt.number = true -- Shows the current line number instead of 0
```

And also:

```lua
-- Fallback mappings (lua/config/fallback/mappings.lua)
vim.opt.langmap = "ñ:,Ñ\\;"
```

Ok, all set. I assume that everyone is already clear on what goes into their
fallback config. Since we have already defined the _what_, now we just need to
focus on the _how_.

## 4. Our fallback module

For configurations loading, we basically have to implement a wrapper around
calls to `require`, or more specifically, a wrapper around `require` using
[pcall](<https://neovim.io/doc/user/luaref.html#pcall()>). From the error that
`pcall` catches, we will be able to get the necessary info and act accordingly.

### The module

The time has come to show off our lua skills. Let's see the basic form of the
code (in `lua/utils/loaders.lua`):

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

From here we can infer the idea, instead of using `require("config.mappings")`
we will use `load_config("config.mappings")` and after that we check if there
was or not any error with `check_errors()`:

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

Why separate loading from error resolution? Because they are 2 different tasks.
First, we want to try to load the standard configuration. Then, instead of
throwing the error immediately we keep trying to load further modules. If these
modules work correctly, great; the current configuration is closer to what we
are looking for despite having encountered errors. Second, having collected the
errors, if there are more than one module with problems we can report them at
once and not one at a time; making the loading more comprehensive and getting
more info in a single run.

### load_configs

Let's review the `load_configs` function. We will collect the errors in a table
that we could call `catched_errors`. We create it outside the function:

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

Now we have the info of the errors, but we lack what to do with it. That will be
taken care of by the `check_errors` function:

```lua
---@param fallbacks? boolean `true` to load fallback settings if errors are found.
---@return boolean -- Returns `true` if no errors are detected.
function Loaders.check_errors(fallbacks)
  -- First the happy path:
  if #Loaders.catched_errors == 0 then
    return true
  end

  -- If troubles arrive, we load the fallback configuration:
  if fallbacks then
    vim.notify("Loading fallback configs.", vim.log.levels.ERROR)
    require("config.fallback.settings")
    require("config.fallback.mappings")
  end

  -- Finally, we inform the user
  local msg = string.format("%s\nDetected errors:\n", string.rep("-", 80))
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  return false
end
```

## 5. Improving the code

Why do we load the _full_ fallback configuration and not just what has failed?

That was my first approach, but depending on what we have in both `settings` and
`mappings`, the config could be in an indeterminate state, for example, if we
associate some shortcut with some `utils` function or based on a specific
setting.

Secondly and more importantly, let's not lose focus. The goal here is to detect
that there has been a problem and to have a relatively comfortable environment
to fix it, not to have an instance of Neovim that works with as many features as
possible.

Anyway, with `load_config` and `check_errors` we already have the basic
functionality we were looking for. However, let's go a little step further and
add the last point of our goals: asking whether or not to open the file with
problems. As most of the time with Neovim, this is quite simple through
`vim.fn.input`:

```lua
function Loaders.check_errors(fallbacks)
  -- etc.
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  if vim.fn.input("Attempt to open offending files for editing? (y/n): ") == "y" then
    print(" ") -- (I think I added this to make the message look better formatted)
    print("Opening files...")

    for _, error in pairs(Loaders.catched_errors) do
      -- We get the path to the troubled file from the error message
      if error:sub(1, 1) ~= "/" then
        error = string.format("%s/lua/%s.lua", NeovimPath, str:gsub("%.", "/"))
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

<!-- prettier-ignore-start -->
> `NeovimPath` is a global variable that I have defined in my `init.lua`. I
> leave here for reference in case you want to use it in your implementation:
>
> ```lua
> ---Path of the lua config (`nvim/lua/config/`).
> MyConfigPath =  vim.fn.stdpath("config") .. "/lua/config/"
> ```
{: .prompt-info }
<!-- prettier-ignore-end -->

We can refactor the new code a little, by moving the obtaining-path logic to its
own function, `get_path_from_error`:

```lua
local function get_path_from_error(str)
  if str:sub(1, 1) == "/" then
    return str
  end

  return string.format("%s/lua/%s.lua", NeovimPath, str:gsub("%.", "/"))
end
```

I will add it inside `check_errors`, but it could very well be as a local or
`UtilsLoader` function.

## 6. Putting the parts together

Incorporating all this in the final code, since we are civilized people, we take
the opportunity to add the corresponding annotations to help us with our lsp
server:

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

But when has _the final_ version of a code actually been _the final_? The most
attentive readers will have noticed at this point that we have actually ignored
an important part of the problem. Let's look at our `init.lua` example:

```lua
local utils = require("utils")

utils.load_config("config.settings")
utils.load_config("config.mappings")
check_errors(true)
utils.load_config("config.plugins")
utils.check_errors()
```

And what happens inside `require("utils")`? What if there is a bug in there?

Well, it would fail, and everything we've built would be for nothing.

## 7. Back to the beginning?

No. This is not like the typical subscription article in _Medium_ that abandon
us when we think a little on out own and go a few millimiters out of its main
frame (😗🎶). Remember that we already have our `UtilsLoader` module working and
therefore there is nothing left to do but apply our `loader` to, in full
_Inception_ style, load the `utils` itself.

<!-- prettier-ignore-start -->
![Ouroboros](ouroboros.jpg)
_Loading the loader of the loader_
<!-- prettier-ignore-end -->

This is the content of my `utils/init.lua` file which is in charge of
incorporating the different modules in one (edited to avoid distractions):

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

In the same way that in our main `init` we have replaced the `require` calls
with those of our own module, here we can do the same and even go a little
further by adding a _loaders loader_.

First, we load our `loaders` module and replace the `require` calls:

```lua
---@class Utils
local Utils

local loaders = require("utils.loaders")

-- load the utils modules
Utils.config = loaders.load_config("utils.config")
Utils.custom = loaders.load_config("utils.custom")
Utils.helpers = loaders.load_config("utils.helpers")

-- etc.
return Utils
```

But why stop there? What if we have a problem within the `loaders` itself?
Again, `pcall` to the rescue:

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
> This time I will leave the code directly in the module, but if you want you
> can create a function. In that case remember to make the corresponding call.
{: .prompt-info }
<!-- prettier-ignore-end -->

Excellent! Now we have all our loads protected and we finally have our own
error-proof armored shoes.

If I may, I will now do a little refactoring to gorup the module loading into a
single function (excluding `loaders` of course) and separate the steps of
loading `utils`, which would only load the `loaders` from the load of the rest
of the utility modules:

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

Beautiful.

With this, we must remember that now in addition to using `require("utils")` we
must initialize the module with `require("utils").load_utils()`. Also, with this
`load_utils` function, in the future we can easily add parameters to the loading
of our utilities. For example, add some profiler, raise a DAP session, or simply
avoid loading certain modules in certain contexts.

Now we just need to adjust our `init.lua` and enjoy:

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

Well, I hope it has been an interesting read and that some of the ideas
presented here will be useful, especially for those who, fearful of ruining
their configuration, refrain from exploring the options offered by this
fantastic "editor".

Good luck!

---

If you are curious, I use this technique in my own configuration. You can check
it out [here](https://github.com/polirritmico/neovim.conf). It's basically the
same as what I have exposed here, but with some additions like dealing with lazy
specs and debugging capabilities.

