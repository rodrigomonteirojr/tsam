# tsam

tsam is a program that interacts with tmux panes using the SAM editor

<img width="1418" height="836" alt="2026-04-25-133451_1418x836_scrot" src="https://github.com/user-attachments/assets/74fb2930-f6a2-4788-8bcb-e6e32b125f2c" />

## Getting started

In order to run this program, you need a few sets of tools installed and in your `PATH`.

- (**Required**) a running [_tmux_](https://github.com/tmux/tmux) session.

- (**Required**) a [_sam(1)_](https://9fans.github.io/plan9port/man/man1/sam.html) compatible
  text editor. In theory, you could use _any_ editor, maybe even X editors, the only
  required assumptions are that there is a `-d` flag, to be run in headless mode
  (a la [_ed(1)_](https://9fans.github.io/plan9port/man/man1/ed.html)), and that the editor accepts
  multiple filenames as arguments (through xargs).
  Preferrably, the editor should not run formatters or other similar
  significant file modifications, since this could cause some issues with the sent-keys detection.
  Anyways, if you want a copy of _sam_, you can grab it from [deadpixi's sam](https://github.com/deadpixi/sam),
  which is what this is tested against and also my favorite pick.

- (**Required**) [_Just_](https://github.com/casey/just). I will write this as a `POSIX` shell script
  sometime, but for now it was way easier to prototype it in _just_.

- (_Optional_) [rlwrap](https://github.com/hanslub42/rlwrap). If you are going to use _sam_, this is
  a no-brainer for you. As such, I've already setup the command and wrapping in the `tsam` script.
  Without it, you won't be able to navigate through the command line,
  though you can still write commands.
  You may disable the wrapper at any time by setting `rlwrap := ''` inside the script.
  (don't comment that line though, it won't work).

Given the above requiremented are met, you are ready to execute `./tsam`. You may place the script
anywhere in your filesystem, preferrably in a location referenced by your `PATH`.

## Introduction

> Note: the following assumes you are running `./tsam` from inside a _tmux_ session.

> *: If rlwrap is enabled, you may see a '> ' prompt show up.

When you first execute the script, you will be presented with the following output:

```
-. /tmp/tsam-vts/0_sh
```

this means the program successfully executed [_sam(1)_](https://9fans.github.io/plan9port/man/man1/sam.html), and you are ready to run commands on your panes.

To verify all your panes loaded correctly, type: '_n_ ⏎'. You should see the something like:

```
-. /tmp/tsam-vts/0_sh
-  /tmp/tsam-vts/1_tail
-  /tmp/tsam-vts/2_cat
-  /tmp/tsam-vts/3_vi
...
```

where filenames match the format 'ID_COMMAND'. Keep in mind they are merely a **snapshot**
of the state of the panes as you executed the program.
> This means you can't analyze **live** data just yet, but it's a planned feature.

Although they are snapshots, they are not read-only. You may modify "pane-files" as you wish,
saving one of them will send positive changes as keys to the corresponding pane.
Here is a simple example:

```
~ $ tsam
n ⏎
-. /tmp/tsam-vts/0_sh
-  /tmp/tsam-vts/1_vi
$ a/tsam!/ ⏎
w ⏎
/tmp/tsam-vts/0_sh: ?warning: last char not newline
#38
^D
tsam!~ $ tsam!
```

> Note: the last line looks weird, this is because the keys were sent right after the program
was finished. Before my shell could take back control, the keys were already "typed" into the pty.
As soon as the shell came back, it correctly received the keys and displayed them after the prompt.

This example demonstrates a _tsam_ session. Consisting of the following commands:

- `n`: Previously mentioned, displays open "pane-files" (and any manually opened files);
- `$ a/tsam!/`: this is also a full command, consisting of an _address_ and an _instruction_;
- `w`: This is an instruction for the editor to save the current active file;
- ^D: This is just Ctrl + d, a common way to exit many command line programs, sends 'EOF'.

The only mystery left is the elusive `$ a/tsam!/` command, let's disect it:

- `$`: this is an _address_, in this case it means simply the EOF (end-of-file),
  which means the next command will operate right at the end of the file;
- `a`: this is an _instruction_, it means "append". It simply appends its _argument_ right after
  whatever address we are operating in;
- `/tsam!/`: this is what we called the _argument_ in the last item. Everything inside `/` is
  passed as input to the specified _instruction_, a literal '/' may be written as: '\/'.

This manual is a work-in-progress. For now, you may read these resources:

- A proper intro: [sam-language](https://ratfactor.com/papers/sam-language)
- The original _sam_ paper: [**The Text Editor**](https://doc.cat-v.org/plan_9/4th_edition/papers/sam/)
- Useful links: [sam.cat-v.org](http://sam.cat-v.org/)

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
- [ ] Processing live data. This could involve usage of pipe-pane, it's in my list for an update.

feel free to send me feature requests.
