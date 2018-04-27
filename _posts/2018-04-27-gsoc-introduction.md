---
layout: post
title: Google Summer of Code 2018&#58; Boldly going where one team has gone before
tags: scummvm
---

This summer, I've been accepted into Google Summer of Code to work on the [ScummVM](https://www.scummvm.org/) project. My task: to reverse-engineer [Star Trek: 25th Anniversary](https://en.wikipedia.org/wiki/Star_Trek:_25th_Anniversary_(computer_game)) and rewrite the game's code in C++ as part of ScummVM's framework.

Why did I apply to ScummVM? Well, to me, it sounded a lot more interesting than
traditional coding projects; I get a kick out of picking apart a game's code (as I've been
doing with [Oracle of Ages](https://github.com/drenn1/ages-disasm) for the last 3 years),
so this should be an enjoyable summer for me.

I'm not quite starting from scratch - an engine with some basic file handling and image
display was started by [clone2727](https://clone2727.blogspot.ca/) several years ago - but
the vast majority of the work remains to be done. I got started on this while working on
my proposal, and implemented the sprite and text systems in the ScummVM engine (see
below).

{% include image.html url="/images/gsoc-introduction-textsystem.png" %}

The task of rewriting the game is complicated by the fact that the game appears to have
a great deal of hardcoded logic; unlike more "sophisticated" engines which have their own
scripting languages, everything here appears to be done with x86 assembly. On one hand,
I'll need to rewrite all of this logic, which is a lot of work. On the other hand, all of
the game's logic will be documented when it's done, so bugfixes will be easier to
implement after the fact.

I'll be posting weekly updates here as I proceed, at least once the "coding phase" starts
(May 14th). My work can be found in [this
repository](https://github.com/Drenn1/scummvm/tree/startrek).

Big thanks to Google and to the ScummVM team for selecting me.
