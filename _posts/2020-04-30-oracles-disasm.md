---
layout: post
title: The 5-Year Anniversary of Oracles-Disasm
tags: oracles
---

On this day, 5 years ago, I made my first commit in the
[oracles-disasm](https://github.com/stewmath/oracles-disasm) project; a full
disassembly for "The Legend of Zelda: Oracle of Ages and Seasons" for the
Gameboy. I've never talked about it much publicly outside of a few tweets, so
I thought this would be a good time to tell the world about it.

## Overview

In a nutshell, the purpose of this project is to reconstruct something
resembling the original source code for Oracle of Ages and Seasons (both games).
Of course, I don't have - or want - the original source code. (Somebody
apparently leaked the Pokemon gen 1/2 source code lately, which is a can of
worms I wouldn't want to deal with.) But, as a game which was written in
assembly - no C compilers involved - it's entirely possible to construct
a reasonable approximation of how the source might have looked. This primarily
involves documenting code, as shown below.

{% include image.html url="/images/oracles-disasm-before-documentation.png"
description="Before: What the heck does this code do?" %}

<br>

{% include image.html url="/images/oracles-disasm-after-documentation.png"
description="After: Aha, this is the code which saves your file!" %}

An important thing to emphasize is that it's not just a "disassembly", in the
sense that it's not limited to documentation. It can also be _re-assembled_ back
into the 2 respective games, which is a vital point; some other video game
disassemblies focus solely on documentation without also being buildable.

Of course, just because it's possible, doesn't mean it's easy. When I first
started out, I had no comments, no variable names to work with. Everything
needed to be documented from scratch. Every variable, every function, needed to
be reverse-engineered and given a meaningful name by me. Every address reference
and function call needed to be replaced with said names. (This last part could
have been partially automated, but it's provably impossible for machines to do
this perfectly.)

This work isn't completely done, but it's close. I would estimate it's over 90%
done for Oracle of Ages, and perhaps 70% done for Oracle of Seasons. (The
disassembly is a single repository which builds both games, since they share
a lot of code.)

## Inspiration

This project was inspired by the
[pokered](https://github.com/pret/pokered/) project, which, similarly, is
a disassembly of Pokemon Red and Blue versions. The pokemon hacking community
has also done the same with
[Pokemon Gold/Silver](https://github.com/pret/pokegold),
[Crystal](https://github.com/pret/pokecrystal), and even
[Pinball](https://github.com/pret/pokepinball), among
[many others](https://github.com/pret). They've even started decompiling C code
for the [3rd gen games](https://github.com/pret/pokeruby), while _still
recompiling correctly bit-for-bit_, which is insane.

Several years ago, I made a project with the pokered repository; a [colorization
hack](http://www.romhacking.net/hacks/1385/) for Pokemon Red and Blue, with help
from others in the community. This was a one-off project, and I'm not
particularly into pokemon hacking. But I very much liked working with the
"pokered" disassembly, and decided the Zelda Oracle games deserved the same
treatment.

## Motivation

So, why do this ungodly amount of work, you ask? Reverse-engineering an entire
game, even a gameboy game, is no small task. Well, there are a few reasons:

* Reverse-engineering is fun!
* I get to discover new glitches and unused content. There isn't much there, but
  every once in a while I find something neat, like this unused enemy, which is
  an upgraded version of the "goponga flower" enemies!

{% include image.html url="/images/oracles-disasm-goponga-flower.png" %}

* A complete disassembly makes it possible to _insert data anywhere_, because it
  can rely on the assembler to recalculate pointers. Without a disassembly or
  source code, it is only possible to _modify data_ or _append data_; patches
  are annoying to make as a result, and this is one reason ASM hacking is seen
  as intimidating.
* The above point, plus the obvious benefit of documentation, makes ROM hacking
  (modding) far easier.

Mainly, my motivation boils down to simplifying the process of ROM hack creation
\- I want to see people design new games with the Zelda Oracles engine.  When it
comes to hacking games, a disassembly is obviously useful, but I'd argue that
it's particularly important for a Zelda game.

For some games - like, say, Super Mario World - this kind of approach might be
overkill. The excellent [Lunar Magic](https://fusoya.eludevisibility.org/lm/)
editor allows one to design new Mario games without writing a single line of
code (not that this has stopped those who are determined).

But a Zelda game has more specific needs. In order to create new and interesting
puzzles and mechanics, writing some code (or at least scripts) is a necessity.
Rather than trying to hide the complexity of the code with a fancy editor,
I want to expose the code as painlessly as possible. This means documenting it
and making it easy to modify.

A tool called [ZOLE](https://github.com/stewmath/ZOLE-4.5/releases), written by
[Lin](http://github.com/lin20), already exists for Oracles rom hacking. It uses
the more traditional approach of editing the ROM directly. I've used it a fair
bit and even made updates for it. It's a decent starting point for ROM hacks,
but it becomes clear very quickly that a very large number of events are
hardcoded and difficult to modify, and new ones are not particularly easy to
create. The disassembly is meant to be a basis for ROM hacking which can deal
with these issues as flexibly as possible.

{% include image.html url="/images/oracles-disasm-zole.png"
description="Screenshot of ZOLE." %}

Aside from proof-of-concept things like my [variable-width font
hack](http://www.romhacking.net/hacks/2934/), no complete hacks have been made
with the disassembly yet. But it's had an indirect effect; the [Oracles
Randomizer](https://github.com/jangler/oracles-randomizer/releases) by Jangler,
which shuffles around all of the obtainable items in the game, used the
documentation from the disassembly to greatly speed up development time. That
being said, there's still a major component which needs to be finished if
I really want to make ROM hacking easy...

## Level creation

Lately I've been focusing on the disassembly's companion software,
[LynnaLab](https://github.com/stewmath/lynnalab). It's all good and well to be
able to hack the code, but you can't make a good ROM hack without being able to
create new dungeons too. This is what LynnaLab is for; it's a GUI program which
allows you to design levels by editing the disassembly's files. This should
allow us to replace ZOLE entirely.

{% include image.html url="/images/oracles-disasm-lynnalab.png"
description="You can create or edit rooms using LynnaLab..." %}

<br>

{% include image.html url="/images/oracles-disasm-lynnalab-ingame.png"
description="...Then rebuild the disassembly, and open the game!" %}

The level-editing software is "secondary", though, in that it's built on top of
the disassembly; it just edits data that's already exposed, and documented
(mostly), in the disassembly. As a result, LynnaLab will probably be missing
a few bells and whistles, such as - perhaps - the ability to edit Link's
starting location. It's something I might add eventually, but such a thing could
be trivially changed by editing the (well-documented) data in the disassembly,
so it's not a priority.

## Recent developments

The disassembly has been largely a one-man job through its 5-year run. In just
the last few days, though, a user by the name of Dan has been helping out - in
particular, he's been working on the oft-neglected Seasons codebase,
particularly by refactoring code from the Ages codebase and re-using it. The
games share a lot of code, after all.

Dan also made a [fork of the Oracles
Randomizer](https://github.com/vinheim3/oracles-randomizer/releases) which
randomizes entrance and exit locations, making for crazy nonsensical playthoughs
of the games. I hear he has even bigger plans in store...

{% include image.html url="/images/oracles-disasm-rando.png"
description="Wait, this isn't supposed to be here..." %}

## Next steps

It's been 5 years, but I feel that the pieces are falling into place. The
disassembly is close to complete \- at least for Ages - while essential features
for LynnaLab, such as editing warps, are being worked on now; once that's
ready, all I will need to do is create tutorials to show people how to use it
all. Things should be in a stable state soon, and then I hope that we'll begin
seeing some Zelda Oracles romhacks.

I'll update this blog when I start making said tutorials, but if you'd like to
follow developments on this, I'm quite active on [this discord
server](https://discord.gg/wCpPPNZ), as are a few other fellow Oracles hackers.
