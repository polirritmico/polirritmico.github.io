---
title: 'Adventures in Neovim: The Art of Surviving Our Reckless Configurations'
date: '2025-01-17 17:03 -0300'
permalink: /posts/Neovim-FalbackConfig
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
# Neovimtures

## 1. Context and Objectives

How many times have we mistakenly added that malicious character that breaks our
configuration into a thousand pieces? A simple `r` unnoticed over a bracket
and... Surprise! Let's eat a chunk of error messages at the most inopportune
moment!

In this article, I propose to approaching a solution to this by building a
module that will help us not only handle the errors in our own config, but also
make the process of correcting and dealing with them much more friendly.

To this end, this _safety net_ should fulfill the following tasks:

- Load our configuration as usual.
- In case of error:
  - Automatically load a fallback configuration.
  - Give us information on where the problems are located.
  - Ask us whether or not to open the offending file(s) for editing.

Sounds good? Well, let's code it!

---

## 2. Approach:

As we customize our Neovim experience, we will probably accumulate code to
simplify things for us. For example, the
[typical wrapper](https://github.com/polirritmico/neovim.conf/blob/bd78a4eb5d97c890bd10a685993bdee3a64ff182/lua/utils/config.lua#L58)
to `vim.keymap.set`.

A standard way commonly observed in more advanced Neovim configurations, is to
group that code or functionality inside a utilities module; the typical `utils`
module in our `lua` path. Thinking about our future selves —and if you have not
already done so—, I suggest incorporating this style or any other to better
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
our own inoperability is that any error on those modules will lead to chaos and
we will end up with the default settings. Depending on the case, this can be a
rather frustrating situation, as not only would we be dealing with the error,
but we would be doing so without access to our fundamental settings.

## 3. Our fallback config

For example, I'll show 2 basic settings for me as examples of the content of
this fallback config to explain the criteria I'm following here, the idea is to
include only the essentials.

The first setting, simply demanded by my fingers' memory:

```lua
-- lua/config/mappings.lua
vim.opt.langmap = "ñ:,Ñ\\;"
```

> For those curious, on `ISO-ES` keyboards, the `ñ` key is located to the right
> of the `l` key, and not `;`. With this setting, I not only enable the `ñ` key
> in normal mode, but also map it to `:` —A two-in-one solution (enable the key
> plus "invert" `;` with `:`).
>
> ![ISO-ES Keyboard](iso-es.png)

Without this, every time I press the `ñ` key (the positional equivalent for
`:`), nothing happens, and common actions like `:w` (`ñw`) followed by `ZQ`
won't have the expected effect 🥲. (`ZQ` is equivalent to `:q!`)

In this example, the second fundamental setting for me is enabling relative line
numbers for `[count]`+`j`/`l` style moves:

```lua
-- lua/config/settings.lua
opt.relativenumber = true -- Show relative line numbers
opt.number = true -- Shows the current line number instead of 0
```

Not having any of these settings is a complete nuisance to me. So in the
fallback configuration, they should definitely go.

Obviously, everyone should adjust the content of their fallback configuration to
suit themselves. I suggest aiming for a minimalistic and robust approach.

For the structure of this fallback configuration, let's replicate the structure
of our normal config. Although, unlike what I do here, you could group
everything into a single module; it's up to you. In any case, let's create the
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

Ok, all set. I assume everyone is clear on what goes into their fallback config
by now. Since we've already defined the _what_, now we just need to focus on the
_how_.

## 4. Our fallback module

For loading our configurations, we basically have to implement a wrapper around
calls to `require`, or more specifically, a wrapper around `require` using
[pcall](<https://neovim.io/doc/user/luaref.html#pcall()>). From the error that
`pcall` catches, we will be able to get the necessary info and act accordingly.

### The module

The time has come to show off our lua skills. Let's see the basic shape of the
module's code (in `lua/utils/loaders.lua`):

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

From here we can infer the idea: instead of using `require("config.mappings")`
we will use `load_config("config.mappings")`. After that, we could check whether
there was an error or not with `check_errors()`:

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

Why separate loading from error resolution? Because they are two distinct tasks.
First, we want to try loading the standard configuration. Secondly, if a problem
arises, instead of throwing the error immediately, we continue loading further
modules to gather more potential errors. In this way, if more than one module
has problems, we can report them all at once rather than one at a time. This
approach makes the loading process more comprehensive and provides more
information about the issues in a single run.

### load_configs

Let's review the `load_configs` function. We will collect the errors in a table,
which we can call `catched_errors`. We create it at module level so we can
access it later:

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

Now that we've collected the errors, we still need to handle them. This will be
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

As you see, the logic and code is very simple.

## 5. Improving the code

Why do we load the _full_ fallback configuration and not just what has failed?

That was my first approach on this, but depending on what we have in both
`settings` and `mappings`, the config could be in an undetermined state. For
example, if we associate a shortcut with some `utils` function or base it on a
specific setting, that relationship would leave the code in a bad state. This is
a fallback functionality; let's avoid potential issues.

Secondly and more importantly, let's not lose focus. The goal here is to detect
that there has been a problem and provide a relatively comfortable environment
to fix it, not to have an instance of Neovim that works with as many features as
possible.

Anyway, with `load_config` and `check_errors`, we already have the basic
functionality we were looking for. However, let's take a small step forward and
add the last point of our goals: asking whether or not to open the file with
problems. As is often the case with Neovim, this is quite simple to do with the
provided built-in tools. In this case, we can use `vim.fn.input`:

```lua
function Loaders.check_errors(fallbacks)
  -- etc.
  vim.notify(msg .. vim.inspect(Loaders.catched_errors), vim.log.levels.ERROR)

  if vim.fn.input("Attempt to open offending files for editing? (y/n): ") == "y" then
    print(" ") -- (I think I added this to make the message look better formatted)
    print("Opening files...")
    for _, module in pairs(Loaders.catched_errors) do
      -- We get the path to the troubled file from the error message
      local path = string.format("%s/lua/%s.lua", NeovimPath, module:gsub("%.", "/"))
      if vim.fn.findfile(path) ~= "" then
        vim.cmd("edit " .. path)
      end
    end
  end

  return false
end
```

<!-- prettier-ignore-start -->
> `NeovimPath` is a global variable defined in my `init.lua`. I'm leaving it
> here for reference in case you want to use it in your implementation:
>
> ```lua
> ---Path of the lua config (`nvim/lua/config/`).
> MyConfigPath =  vim.fn.stdpath("config") .. "/lua/config/"
> ```
{: .prompt-info }
<!-- prettier-ignore-end -->

We can refactor the new code a little, by moving the obtaining-path logic to its
own function, `get_path_from_error`, and also lets add support for more cases:

```lua
local function get_path_from_error(str)
  if str:sub(1, 1) == "/" then
    return str
  end

  return string.format("%s/lua/%s.lua", NeovimPath, str:gsub("%.", "/"))
end
```

I will add it inside `check_errors` to avoid parsing it if no errors are found
(check the code below), but it could just as well be defined as a `UtilsLoader`
or a local function.

## 6. Putting the parts together

First, we need to choose a way to expose our code. For example, we could simply
do something like `require("utils.loaders").load("config.mappings")` and call it
a day. However, let's build something a little more ergonomic. In
`utils/init.lua`:

```lua
local Utils = {}

Utils.load = require("utils.loaders").load_config
Utils.check_errors = require("utils.loaders").check_errors

-- etc.

return Utils
```

Let's incorporate all this into the final code. Since we are civilized people,
we should also take the opportunity to add the corresponding annotations to
assist our LSP server in its noble and chivalrous task:

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

But when has _the final_ version of any code ever actually been _the final_? The
most attentive readers will have noticed by now that we have actually ignored an
important part of the problem. Let's look at our `init.lua` example:

```lua
local utils = require("utils")

utils.load_config("config.settings")
utils.load_config("config.mappings")
check_errors(true)
utils.load_config("config.plugins")
utils.check_errors()
```

And what happens inside `require("utils")`? What if there's a bug in there?
After all, it's a huge bug vector.

Well, it would fail, and all of our work would be for nothing.

## 7. Back to the beginning?

No. This is not like the typical subscription article on _Medium_ that abandons
us the moment we think a little on our own and venture a few millimeters out of
its main frame (😗🎶). Remember that we already have our `UtilsLoader` module
working, and therefore, there is nothing left to do but apply our `loader` to
—in full _Inception_ style— load the `utils` itself.

<!-- prettier-ignore-start -->
![Ouroboros](ouroboros.jpg)
_Loading the loader of the loader_
<!-- prettier-ignore-end -->

Let's assume this is the base content of our `utils/init.lua` file, responsible
for incorporating the different utility modules into one source (narrowed down
to avoid distractions):

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

In the same way as in our main `init`, we have replaced the `require` calls with
those of our own module; here, we can do the same and even go a little further
by adding a _loaders loader_.

First, we load the `loaders` module itself and then replace the `require` calls
with it (don't forget to check if there were errors in the importing procedure):

```lua
---@class Utils
local Utils

local loaders = require("utils.loaders")

-- load the utils modules
Utils.config = loaders.load_config("utils.config")
Utils.custom = loaders.load_config("utils.custom")
Utils.helpers = loaders.load_config("utils.helpers")
assert(loaders.check_errors())

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

-- Expose the module through utils
Utils.load = loaders.load_config
Utils.check_errors = loaders.check_errors

-- load the utils modules
Utils.custom = loaders.load_config("utils.custom")
Utils.config = loaders.load_config("utils.config")
Utils.helpers = loaders.load_config("utils.helpers")
assert(loaders.check_errors())

return Utils
```

Excellent! Now we have all our loads protected, and we finally have our own
error-proof armored shoes ready.

### Final touches to utils

If we may, let's now do a little refactoring to group the module loading into a
single function (excluding `loaders` of course) and separate the steps of
loading `utils`, which would only load the `loaders`, from the loading of the
other utility modules. The code explains itself better:

```lua
---A collection of custom helper functions.
---@class Utils
---@field config UtilsConfig
---@field custom UtilsCustom
---@field helpers UtilsHelpers
local Utils = {}

-- Load the loaders. (executed only during the first `require("utils")` call)
local ok, loaders = pcall(require, "utils.loaders")
if not ok then
  vim.cmd("edit " .. NeovimPath .. "/lua/utils/loaders.lua")
  error(string.format("Can't load 'utils.loaders':\n\n%s\n", loaders))
end

Utils.load = loaders.load_config
Utils.check_errors = loaders.check_errors

-- Load the utils modules.
function Utils.load_utils()
  Utils.config = Utils.load("utils.config")
  Utils.custom = Utils.load("utils.custom")
  Utils.helpers = Utils.load("utils.helpers")

  assert(loaders.check_errors())
end

return Utils
```

**_Beautiful._**

With this, we must remember that now, in addition to using `require("utils")`,
we must initialize the module with `require("utils").load_utils()`.

It's worth mentioning that with this design approach, the `load_utils` function
could easily be modified to add parameters for loading our utilities in the
future. For example, we could add a profiler, start a DAP session, or simply
avoid loading certain modules in specific contexts.

Now, we just need to adjust our `init.lua` and enjoy:

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

## 8. Conclusions

I wish I had something like this in the early stages of my _Neovim adventures_.
But nevertheless, when these days I'm adding something to my setup and feel like
I know what I'm doing, this little module comes to the rescue and reminds me
that even a self-named level 20 paladin can roll a 1 at the worst moments. In
those situations, it's nice to have done the homework and have a resilient
system that protects us from our own cognitive clumsiness. At those moments,
when the module arises and saves the day, every time I thought to myself, "Oh,
this is the kind of thing I really enjoy about using Neovim".

If we're concerned about the performance impact of this approach on our config,
I ran a simple comparison between `require` and `utils.load`. After 20 Neovim
startups, the average difference in time was 0.55ms. To put that in perspective,
if troubleshooting an issue normally takes an extra 30 seconds without the
fallback settings (and of course, that would _feel_ like much more if you keep
pressing `ñ` to open the command line instead of `:`), the time saved by fixing
a single error equals the load time of 54.545 Neovim executions. In that
analysis, of course, the time spent building the module and going through all
this reading is invaluable pure joy.

Well, I hope it has been an interesting read and that some of the ideas
presented here will be useful, especially for those who, fearful of ruining
their configuration, refrain from exploring the options offered by this
fantastic "editor".

Good luck!

---

If you are curious, I use this technique in my own configuration. You can check
it out [here](https://github.com/polirritmico/neovim.conf). It's essentially the
same as what I've shown here, but with some additions like handling lazy specs
and debugging capabilities.

