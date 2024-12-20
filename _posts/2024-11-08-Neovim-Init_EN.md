---
title: '(WIP) Neovim Adventures: Let''s start from the init'
date: '2024-11-08 20:02 -0300'
permalink: /posts/Neovim-Init
lang: en
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: First chapter in Neovim Adventures.
media_subpath: /assets/img/neovim/
image: nvim.png
---
# Neovim Adventures I (WIP)

## Objectives and Generalities

I have been using [Neovim](https://neovim.io/) almost exclusively for editing
text and code in my system for some years now. Throughout this time I have been
accumulating some recipes, concepts and ideas (my own, copied and adapted) that
have led me to a fairly stable and flexible organization of my development
environment along with a series of criteria that I want to share through this
series of articles.

Not all of these ideas have survived, the vast majority in fact I have
eliminated them as they have been relegated for months to unused code and thus
not organically integrated into my workflow. However, learning and exploring the
possibilities that Neovim offers to materialize them allows to extract a really
useful learning when facing new developments. Sharing this learning and its
conclusions I think can be something quite interesting, especially for those who
are just starting with their own _NeoVimtura_. Moreover, what may seem useless
to one may well be the key or turning point for another.

Generating didactic material that addresses these points and above all with a
little more depth as to the why of things instead of just making a tutorial or
recipe to follow, is another of the main objectives of these articles. I am
interested in conveying a deeper approach to the development of our tools,
trying to contribute to the _"Why do I want to do this?"_ or the more
fundamental _"Is this the best way to approach the problem?"_, because at the
end of the day, _taking the edge off_, adjusting such a central piece for all of
us who work with text on a daily basis can potentially bring benefits that we
will enjoy on a daily basis. Obviously, I do not aspire to reach a conclusion
that is valid for every use case, but at least to be able to raise questions
that I consider more fundamental than the ones that I have already discussed.

### Route plan. What and how.

_A priori_, the series will be composed of the following articles:

> 1. Start and guide (this article).
> 2. Keys and criteria Organization and error handling
> 3. Utilities and customization

### To whom

If you are reading this I assume two things. First, that **you are
interested** -_even entertained_- in fine-tuning your working tools, like a
carpenter sharpening and honing the edge of his planer until he finds the
perfect sound and chip. And secondly, I assume that **you don't want to waste
time reinventing the wheel**. You don't want to waste hours or even _days_ on
problems already solved and within reach of a function, flag or parameter just
to save a few minutes of reading.

The series will be mainly focused towards:

- Neovim users with a little experience, curiosity or who have already taken
  their first steps with the editor and want to put together for the first time
  their own configuration or a more definitive one.
- Those who have already written their configuration but are not convinced
  (encapsulation problems, performance, behavior, etc.).
- Enthusiasts who are always looking for ways and ideas for optimization or just
  different points of view.
- Novice users who are still shipwrecked in the sea of overwhelming abundance
  that nvim offers on how to solve things.

### Prior Knowledge

Specifically the series will assume some prior knowledge of the following items:

- **Basic functions:** Switching modes, typing text, saving, exiting, etc. For
  completeness I will mention them, but to learn that I suggest `:Tutor`.
- **Vim moves:** Yes, editing a string using `ci` is beautiful, but too much has
  already been written about it.
- **Vimscript:** Even in the days of Vim I never liked it and always found it
  too specific. To this day this feeling remains and I'll just say that I'd
  rather learn Lisp and switch to Emacs than get into it (thanks Lua).
- **Programming concepts:** I will use programming concepts such as function,
  variable, tables, API, etc., but at the same time I will try to explain what I
  mean for those more distant or new to coding. Otherwise, Lua is a pretty
  simple language, and even friendly in my opinion, so it's a perfect
  opportunity to get a little bit into programming.
- **Windows stuff:** Sorry, I haven't used that operating system for years and I
  think that if PCs came with Linux by default from the stores, computing in
  general would be in a healthier state.

Notwithstanding the above points, I will try to include help boxes and _tips_
that provide certain information that I consider valuable or useful to handle on
these topics.

### Methodology

It will depend on the content, but it could be limited to the following points:

- Ask ourselves questions as users and developers.
- Ask meta-questions (questioning whether or not we are asking the right
  question).
- Find the correct nomenclature and explore what the Neovim documentation tells
  us about it.
- Review the possibilities.
- Write the code.
- Conclusions.

With this I hope to contribute to a more refined notion of how and when to build
our own solutions, that these are specifically adapted to our requirements (or
what we think we need), and of course that this does not degrade performance or
take away flexibility from the configuration code.

With these general clarifications, let's proceed to the core of this chapter:
The start, our `init.lua`.

**Let's get to work!**

---

## Init

_Back to the beginning_

To begin this series of adventures, I think it is best to do it twice, that is,
to learn about the directory and file structure that Neovim proposes, and at the
same time, to learn how to use the documentation to avoid having to open the web
browser every time we don't know how to deal with a specific situation. The more
time we spend in Neovim, the faster we will become familiar with its flow,
terminology; and learning to navigate between **windows** and **buffers** is
something that the sooner we become familiar with the better.

<!-- prettier-ignore-start -->
> **It is important to nuance our learning process**: A piece of advice that I
> personally found very useful for a successful transition to Neovim and that I
> will always try to share, is not to completely discard the editor we are
> already familiar with while we are in this learning state.
>
> If after a while working in Neovim, we are already tired or frustrated and
> everything feels uphill, or if we suddenly have to solve situations
> immediately or respond to deadlines; the best thing to do in these cases is to
> go back to the familiar and continue learning at another time. In this way we
> avoid completely valid frustrations and improve our ability to understand when
> we really want to learn.
>
> In contrast to other editors, Neovim assumes a completely different paradigm,
> so it can be quite overwhelming at times to tackle everything at once.
{: .prompt-warning }
<!-- prettier-ignore-end -->

Then, startup our favorite terminal ([Konsole](https://konsole.kde.org/) in my
case) and run Neovim:

```sh
$ nvim
```

<!-- prettier-ignore-start -->
> When we see a shell command preceded by the sign `$` or (`%`), by convention
> it means that we can use it with any user without having permissions problems.
> On the contrary, when we see that it begins with `#`, it means that it is
> expected to have superuser or `root` permissions.
{: .prompt-tip }
<!-- prettier-ignore-end -->

I assume we don't have any configuration files or directories. In the future,
one way to temporarily reopen Neovim in this default state is by `nvim --clean`.
Very useful especially when something doesn't work and we want to know if the
cause of the problem is in our configuration or not.

<!-- prettier-ignore-start -->
![Default interface](default-nvim.png)
_The empty interface, just like the beauty of a natural stone containing all
shapes and sculptures_.
<!-- prettier-ignore-end -->

Well, with the editor now open, let's help ourselves and see how to navigate
through the help.

## Help and Windows

To open the Neovim help we have to open the `command-line` and execute the
`help` command.

<!-- prettier-ignore-start -->
> In vim/nvim terminology, running `:help` means:
>
> Open the `command-line` (in **normal mode**, press the `:` key).
> Type the command `help` (optionally we can type just `h`).
> Press `<CR>` (the `enter` key).
{: .prompt-info }
<!-- prettier-ignore-end -->

Remember that Neovim is a **modal editor**, this means that the keyboard and all
the software will change its operation based on the mode we are in. In this
case, by default we are in the _normal mode_ to which we can easily return at
almost any time by pressing `<Esc>`. Finally, with the above instructions we
should have opened the help window (`<F1>` also works).

And just like that, we are already faced with the first challenge: **Do I have
to read the documentation while occupying only half of the screen?** Well, no!
Solving this type of situation is precisely what this series is all about. So
before solving it, I invite you to reflect a little on this point.

<!-- prettier-ignore-start -->
![Help!](help.png)
_Such a waste of space..._
<!-- prettier-ignore-end -->

The key question here is **What do we want to do?**, and the answer is not
obvious as it may seem: _"I want the help screen to take up the whole screen and
not just half of it, so let's set that up"_. Well, yes and no. This is the first
[x-y trap](https://xyproblem.info/) we face. What we really want is not for the
help **window** to take up the whole screen, but to be able to read the
documentation comfortably.

Rhetoric! Some may say not without a hint of reason, but the change of focus is
important. _It is the context that determines our comfort_, and we are just
getting into this tool. Do we have enough context to question this decision?
What were the developers thinking when they decided to go for this default
behavior? We don't know. Will full-screen help be comfortable in the most common
use cases? We don't know either. Not at least not while we are in this
familiarization process and certainly not when we barely know how to organize
and move between windows.

Therefore our first instinct, that of _"I'm going to set the help window to open
full screen by default"_, obeys the same principle of premature optimization,
and therefore let's consider it a mistake at this point or at least an
uninformed decision. Perhaps it is indeed the right setting, but we lack
sufficient exploration to determine this.

Changing this default behavior may be more of a waste of time than the benefit
or convenience we are looking for. A future _phantom setting_ that we now find
logical, but is potentially one of those options that knowing how to navigate
and manage windows can tend to hinder rather than help or accommodate.

So how do we fix it? We don't. Simply, having the focus on the help (or the
focus on the _window_ that has the help _buffer_) we press `<C-w>o` to make the
selected window the only one on the screen.

<!-- prettier-ignore-start -->
> **NeoVim/Vim-style keystroke typing:** \*\* \*\*NeoVim/Vim-style keyboard
> shortcuts
>
> `<C-w>o` means to press the `ctrl+w` keys, followed by the `o` key. As a
> mnemonic aid, we can understand the `<C-w>` as _"window stuff"_ (**w**indows)
> and the `o` key as **o**pen (open).
{: > .prompt-info }
<!-- prettier-ignore-end -->

Now, with the documentation in full screen, we are left to figure out how to
find the documentation on how to configure Neovim. So let's try to find this
section with `:help configuration`.

Oh, what the heck is `lsp-handler-configuration`? Well, it's obviously not what
we're looking for.

Traditionally, terminal applications use mechanics that follow certain
historical usage patterns. In this case we are facing precisely one of them,
inside the `command-line`, we can use `<Tab>` to autocomplete our command and
its arguments. In this case if we type only the first `h` of `help` and press
`<Tab>`, Neovim will **autocomplete** the command. When there is more than one
possibility it will open a menu where we can choose the option we want and press
`<Tab>` again to **expand** the selection.

In this way, we can see that by searching for `conf` we have many possibilities
to choose from such as `:conf`, `config`, `confirm`, etc. Let's try
`:help config`. Ah, now we are talking, this does make sense.

This is one of the points that will become easier as we become more familiar
with Neovim. familiarize ourselves with Neovim, since at first we have no notion
of the terminology used in the manual. terminology used in the manual. Later on
we will be able to use **Telescope**, **Fzf** or other alternatives to help us
with this, but for the moment we can already read in the help the for the moment
we can already read in the help the following message (edited):

> A file containing initialization commands is generically called a "vimrc" or
> config file...
>
> The config file is located at:
>
> - Unix: `~/.config/nvim/init.lua`
> - Windows: `~/AppData/Local/nvim/init.lua`
> - XDG: `$XDG_CONFIG_HOME/nvim/init.lua`

OK, let's generate this file. But how do we close this help window? Of course,
but taking advantage of the need, let's dig a little bit into the windows.

## Moving between windows

We might intuit that what is showing each window is _the file_, but this is not
quite true. Neovim/Vim operate under an abstraction called **buffer**. As soon
as we finish

When there is more than one window on the current screen, we can change the
focus with `<C-w>` plus the window address (`h`, `j`, `k` or `l`).

## Paths

In Unix terminology, `~` represents the home directory of our user. For example,
if our user's name is `foo`, then `~/` represents the `/home/foo/` path on our
file system. Also, it is common on Linux systems to have by default the
`XDG_CONFIG_HOME` environment variable pointing to `~/.config/`. We can check
where it points to by opening a terminal and checking the output of the command:

```sh
$ echo "$XDG_CONFIG_HOME"
```

Ergo, Neovim is going to look for our `init.lua` file in
`/home/<username>/.config/nvim/init.lua`. Let's create it:

```sh
$ cd
$ mkdir .config/nvim
$ cd .config/nvim
$ touch init.lua
```

To edit!

```sh
$ nvim init.lua
```

Here we have our glorious configuration file:

<!-- prettier-ignore-start -->
![Empty file in Neovim](base-init.png)
Our beautiful blank page! Look at it! So full of possibilities!
<!-- prettier-ignore-end -->

Now let's see if Neovim is reading or not the file through the classic _Hello
world!_ (Yes, _Hello world_ to configure our editor). We use the following code:

```lua
-- (Comments begin with --)

-- The following function allows us to notify the user with a message:
vim.notify("Hello world!")
```

<!-- prettier-ignore-start -->
> A good source of information about the Lua language can be found -of course-
> in the documentation: `:help lua`.
{: .prompt-tip }
<!-- prettier-ignore-end -->

The code is entered by switching from `normal-mode` to `insert-mode` with the
`i` key and once written we return to normal mode with `<Esc>`. To save the
changes and close the current Neovim instance, we can press `:qw` or `<ZZ>`.

After reopening Neovim we have this:

![Hello world](hello-world.png)

Perfect. Everything is working. Once checked let's clean the file.

> If at any time we want to see again this or any message of the current session
> we can use the command `:messages` or `:mess`.

<!-- prettier-ignore-start -->
> Lua offers the `print` function that we can use instead of `vim.notify`,
> however, I prefer the latter because it allows to add message levels
> (warnings, errors) and according to the documentation it can be replaced by a
> custom provider.
{: .prompt-info }
<!-- prettier-ignore-end -->

## Our init.lua

How to organize our configuration is a subject that we will deal with in detail
later on, however in general terms we can mention two options:

1. We handle everything in a single `init.lua`.
2. We separate the different scopes of our configuration into multiple files.

Both strategies offer benefits and difficulties, it is mostly a personal
decision, but I believe based on my experience working on different code bases;
that sooner or later a single file almost always becomes difficult to maintain
and organize. Perhaps the most plausible exception is that our configuration has
already reached an important point of stability and at the same time, it does
not have so much length in lines of code.

Anyway, I think that in this case the correct strategy (or perhaps the most
likely to prove correct) is to divide and group our _setup_ based on files (or
modules) of functionality. This division as in any code, is a challenge of
subtle implications and therein lies its difficulty, but _a lot of water has
passed under this bridge_, so we can apply the solution that many have found in
this regard, and it is precisely the one we will apply: **divide our
configuration into multiple files**. In addition, browsing through the various
files of a project in Neovim is a joy compared to other IDEs, especially if we
enhance it with some _plugins_ (although the default possibilities are also very
good).

We therefore believe that the following organizational proposal is the best way
forward:

```
nvim/
├── init.lua
└── lua
    └── config
        ├── mappings.lua
        └── settings.lua
```

And for neovim to "read" the `mappings.lua` and `settings.lua` files we use the
`require` function inside our `init.lua`:

```lua
require("config.settings")
require("config.mappings")
```

`require` is a Lua function that operates under the **modules** abstraction. A
Lua module is basically a `.lua` file that Lua reads through the `require`
function. After reading the file, it executes its code and stores the result in
memory so that its functions, variables or settings are available for use or
reuse.

In the above case, `settings` and `mappings` are our modules and `config` would
be a container directory that we use for code organization.

I think the meaning of each of both modules is self-explanatory, but in any
case, the idea is that inside `settings` we add all the Neovim specific options
such as columns with relative numbers, default text width, status bar options,
etc.; and inside `mappings` we add everything related to shortcuts and keyboard
configurations.

We will review their content in the next release.

---

## Conclusions

We have made little progress, but at the same time covered many elements. Future
installments will have much more code. For now, by way of conclusion let's list
the key ideas we have covered:

- Opening Neovim documentation or help.
- Commands related to windows (`<C-w>`):
  - `<C-w>o` to enlarge a split window (with focus on the window we want to
    enlarge).
  - `<C-w>` plus the direction of the window to focus to change the "window
    focus".
- Help through the `command-line` (`:help <search>`).
- Configuration file `:h config`.
- Function `require`.

