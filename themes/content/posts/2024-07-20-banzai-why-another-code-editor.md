---
type: post
title: "banzai! why another code editor?"
date: "2024-07-20T00:00:00-03:00"
tags: [dev,editors]
showTableOfContents: true
---

My first editor for code was Microsoft Notepad, why? Because Internet Explorer on Win95 used to show page source there, so I like to edit HTML and batch scripts using it, then FrontPage Express (to have syntax highlighting support) and later notepad++.

When finally started to use Linux and later OpenBSD choose vi, yes vi, not iMproved version, emacs is not available by default, used vi for notes, shell scripting then some C, found funny the `~` marks, buffer manipulation and shortcuts, never was a advanced user, or have complex `.vimrc`, or problems to `:q!`.

### editors time line
- 1976: emacs, vi
- 1983: notepad
- 1989: pico
- 1991: vim
- 1999: gedit
- 2001: kate
- 2003: notepad++
- 2005: geany
- 2008: sublime
- 2011: kakoune
- 2014: atom
- 2015: neovim, vscode
- 2019: lite
- 2020: lite-xl
- 2021: helix
- 2023: zed

Things changed when I started to work as professional programmer and was introduced to IDEs, using Delphi 7 and Visual Basic 6, start to debug using breakpoints, pure magic, skyrocket productivity. Then started to work as webdeveloper with PHP and Ruby, back to text editors until Java (using Eclipse and Netbeans) for while.

### moving from zed
>_Every program attempts to expand until it can support LLMs. Those programs which cannot so expand are replaced by ones which can._

(modified James Zawinski law from 1995 to 2022)

The year is 2024, tried to use almost all mainstream code editors, you name it, now most of time I use Zed, and obliged to use Xcode and Android Studio for mobile development, you know.

Tried Lapce, Helix and I was pretty happy with Zed for while, started to use it at 0.160.0 before it get support for Linux, now I get updates every time... Some update messed with my plugin config, change my theme, I dont care about social, AI thing, I just want a reliable editor that works, manage resources efficiently, simpler and better than vscode, tried to build Zen for Linux once, but why I need postgres and docker to build a code editor?!
The Cargo file has more than 500 lines, more than 150 external deps from crates... Forget it, just prefered to wait for Linux official support...

After some time noted huge memory consuption, some leak overtime with couple small files opened. So, they prefer add AI/social integration features than support more platforms or fix this [issues](https://github.com/zed-industries/zed/issues/18673#issuecomment-2408663493)...?!

Zed with Windows support will probably compete with VScode in couple years, maybe become developer editor _de facto_, but these issues for me is just unacceptable...Good luck then, I'm back with my old plan to maintain my own editor like many other devs... You will move to another home, quit your job, buy another laptop, even change your OS, but probably will continue to use the same code editor...

As inspiration...
- Salvatore Sanfilippo (aka antirez) created [kilo](https://antirez.com/news/108)
- Rob Pike created [acme](http://acme.cat-v.org/)
- John Romero created [TEd the editor that shipped over 30 games](https://www.gamedeveloper.com/design/classic-tools-retrospective-john-romero-talks-about-creating-ted-the-tile-editor-that-shipped-over-30-games)

## banzai editor
[![banzai](/img/banzai.png)](https://en.wikipedia.org/wiki/Top_Gear_(video_game))

[_This is how you live._](https://cyberpunk.fandom.com/wiki/Mizutani_Hozuki_%22Hoseki%22)

### lite editor
Years ago in 2020 started to use [lite](https://github.com/rxi/lite) written by rxi, Lua lightweight editor using SDL rendering in C, a delight experience, love it, but miss some things, little clunky, buggy, not have time to fix it, some time later it was forked by Francesco Abbate and became [lite-xl](https://github.com/lite-xl) project.
Appreciate many things, utf8 support, watchdir, the plugins ecosystem flurish, but not liked the UI changes, how depedendencies are handled, the idea of a plugin manager etc... So, decided to use lite again, as a start, with some influence, changes and assets from lite-xl...

### just works™
How much time you need to setup your dev environment? I hate unattended updates, or that one which mess with current configs, incompatible changes argh...
just want portable binary, couple megabytes, need couple disk kilobytes to save editor assets, your config, session, maybe plugins... like helix, you have some lsp server installed? nice, will try to use it...

### defaults
I always use more than one machine, with different processors archtectures and OS, I'm early-adopter, like to try new stuff from time to time, so I just give up to maintain dotfiles or adapt config files,
prefer simplicity and portability over customizations, optinated defaults from the apps, use them as they come.

### gui vs tui
If you're a developer these days certanily will use a browser and terminal emulator to handle a package manager, infra tooling and git at least.
For me doesn't make sense tweak terminal editor to appear like gui one, as said had good experience with gui editors (zed, textmate, sublime) for while, and SDL comes with acceleration support.
I'm using vi/m less and less, editing remote configs eventualy, now everything are running in containers, can be debugged remotely.

### low learning curve
Never understand the cult behind VI/m, why people still using hjkl if we have arrow keys in any modern computer? Why maintain this antropological cult of ADM-3A terminal keyboard layout/shortcuts?
I can understand the flow of not use a mouse, used qutebrowser for a while, but as Carmack says it's "civil war re-enactment". My idea is mantain a low learning curve like notepad or web browser so you can customize and introduce advanced editing overtime or even use _vim-mode_ plugin as you wish.

### minimalism
I like _zen mode_ for editors or minimalist designs like ElementaryOS code, no buttons, sidebars, visual distractions as low as possible, just you and code.
But need to have tree view for files, I manage too many different projects, different assets, like to have an overview across it or them. Decided to maintain all configuration in one file, using lua like neovim not json, keep things simple and powerful...

### zen of banzai
Considering all my experience, essential things that won't change over time I can summarize as mantra
what I expect and deserve from code editor...

- insertion latency and memory footprint lower as possible
- must adapt to evironment not create another one
- customization must be possible not rule
- no external depedendencies
- portability must be easy
- one configuration file
- low learning curve
- no social features
- no auto-updates

### banzai
_banzai_ is japanese cheer of enthusiasm or triumph, meaning "10k years" of long life, I hope to use it until _the end of times_. Decided to rewrite the _launcher/renderer_ in Zig to bring all desired simplicity and portability, will be much easier to maintain and improve over time... Not a fork of lite or lite-xl, will be like a _little brother_, anyone is welcome to try it, test, submit fixes, but if you miss any feature not included will be better to fork or start your own journey, [banzai!!!](https://github.com/dgv/banzai)


