# tsam

tsam is a program that interacts with tmux panes using the SAM editor

<img width="1418" height="836" alt="2026-04-25-133451_1418x836_scrot" src="https://github.com/user-attachments/assets/74fb2930-f6a2-4788-8bcb-e6e32b125f2c" />

## Getting started

To run this program, you need a few set of tools installed and in you `PATH`.

- (**Required**) [_tmux(1)_](https://github.com/tmux/tmux) of course.

- (**Required**) a [_sam(1)_](https://9fans.github.io/plan9port/man/man1/sam.html) compatible
  text editor, specifically, it could theoretically be _any_ editor, maybe even X editors, the only
  required assumptions is that it has a `-d` flag, to be ran in headless mode
  (a la [_ed(1)_](https://9fans.github.io/plan9port/man/man1/ed.html)), and that it accepts multiple
  files as arguments (through xargs). Preferrably it should not run formatters or other similar
  entire file modifications, since it could cause some issues with the sent-keys detection.
  Anyways, if you want a _sam_, you can grab it from [deadpixi's sam](https://github.com/deadpixi/sam),
  which is what this is tested against and also my favorite pick.

- (**Required**) [_Just_](https://github.com/casey/just). I will write this as a `POSIX` shell script
  sometime, but for now it was way easier to prototype it in `just`.

- (_Optional_) [rlwrap](https://github.com/hanslub42/rlwrap). If you are going to use _sam_, this is
  a no-brainer for you, i've already setup the command and wrapping in the `tsam` script,
  so if you already have it installed, there are few reasons not to use it, since without it you can't
  even navigate through the command. But you may turn it off simply by setting the `rlwrap := '...'`
  line in the script to something like `rlwrap := ''`, and it's off! (don't comment the line though,
  it won't work).

## Features

- Searching for text in a specific pane:

```
> X/1_/ 0/pts/ +- p
/dev/pts/3
```

- Executing `sam` commands over multiple panes:

```
> X 2p
~$ tsam
/dev/pts/3
/dev/pts/5
```

- Sending keys (frail, works in a very specific way):

The current implementation saves two versions of each pane, one under `/tmp/tsam-vts` and the other
under `/tmp/tsam-vts-ro`, the way sending keys works is by diffing the two versions,
(with `/tmp/tsam-vts` being the one you modify), grepping the `^+` lines and simply sending them
literally through `tmux send-keys`. The reason you must append text strictly to `$` (EOF) or a blank
line is to avoid mixing with the other text in the terminal
(since it's hard to tell what commands you really want to send if your line contains e.g.
'C-l "ls" Enter~$ tsam 2>/dev/null').

All of this also means that you must save the files. Feel free to suggest alternatives!

```
> X/1/ $- a/C-l "ls" Enter/
> X/1/ w
/tmp/tsam-vts/1_sh: #592
> ^D
```

> Note: I used `$-` to avoid pesky warnings from sam about the file not ending on a newline. This is
handy, and for this purpose I also included a safeguard in the source code to guarantee that `$-` is
always a blank line, you can treat is a `command buffer`.

- ...And many more. `y`, `g`, `v`, `Y` and others work as expected.

## TODO

There are a few major design things to think about, and I couldn't even gather my thoughts on it
by now, but here are some ideas:

- [ ] Implement a `POSIX` shell script. This is the easiest, the script is `just` a wrapper :-).
- [ ] Redesign send-keys functionality. I'll be thinking on this one, but it's good enough for now.
- [ ] Killing panes. This could be based on deletion, i need a way to index which files were kept open.

feel free to send me feature requests.
