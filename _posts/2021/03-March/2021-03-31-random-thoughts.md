---
layout: post
title: Random Thoughts and Things
subtitle: What's that dripped from your ear?
tags: [osx, shell]
published: true
---

So looking to up my nerdiness some more (I know - how much higher can it go?), I'm starting to try and CLI more, specifically moving to alacritty (instead of iTerm2), and using tmux and vi more.

## Alacritty

> Alacritty is a modern terminal emulator that comes with sensible defaults, but allows for extensive configuration. By integrating with other applications, rather than reimplementing their functionality, it manages to provide a flexible set of features with high performance. The supported platforms currently consist of BSD, Linux, macOS and Windows.

This is a new terminal is pretty slick and configurable from a yaml file.  Read [more here](https://github.com/alacritty/alacritty).

I decided to try it cause I watch too many YouTube videos, and was bored that Saturday afternoon.  I like it, its growing on me, but isn't as 'OSX native' as you might like -- and it's missing things like split screens, multiple tabs, etc.

Enter tmux.

## Tmux

>tmux is a terminal multiplexer: it enables a number of terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached.

[Tmux](https://github.com/tmux/tmux) allows for 'sessions' that you start, and can detach/reattach to (like `screen`), but also has a plethora more functionality and customization for different keys, plugins and theming.  I've done some basic changes, like remapping the `prefix` to CTRL-a instead of CTRL-b, and a few more.  I also finally started putting my dotfiles up on [Github](https://github.com/pdaugavietis/dot-files).

Of course, since we're on the CLI and editing files, we should use `vi`.

## VIM (or NeoVIM)

Instead of just plain vim, I installed NeoVIM and prospered.  Nvim has a whole bunch of extra functionality, but also moves away from the traditional `.vimrc` file and into the `.config/nvim/**` world of configuration.  I also started using VIM plugins and way more than the default, out-of-the-box configuration (again, dotfiles are on [Github](https://github.com/pdaugavietis/dot-files)).  You

## Base 16 Themes

We might be on the command line, but we still like to be pretty!  So we're using the Base 16 theme packs for alacritty and vim, and putting them together with [`alacritty-colorscheme`](https://github.com/toggle-corp/alacritty-colorscheme) to the set the schemes for both alacritty and nvim in the same command.

```bash
git clone git@github.com:aaron-williamson/base16-alacritty.git ~/.aaron-williamson/base16-Alacritty
ln -s ~/.aaron-williamson/base16-Alacritty ~/.config/alacritty/colors
```

```bash
# plugin install
Plug 'chriskempson/base16-vim'
# then reference the installed colors.vim
```

```bash
alacritty-colorscheme -V apply base16-classic-dark.yml
```
