---
layout: post
title: The Strange Case of the Obfuscated(?) Zelda ROM
tags: oracles
---


Yesterday, there was a huge leak of Nintendo assets. Particularly, source code
and prototypes have been leaked for a huge number of games. These include source
code for Star Fox, Star Fox 2, Yoshi's Island, Mario Kart, and more recently
some N64 games including Super Mario 64 and Ocarina of Time, and more...

But, of particular interest to me, is source code and prototype leaks of Link's
Awakening DX.

I have mixed feelings about this. This already happened to the first
4 generations of Pokemon games; they had their source code leaked some time ago.
I mused in an earlier post that I hope such a thing never happens to the Zelda
Oracle games, because of my work on my
[oracles-disasm](https://github.com/drenn1/oracles-disasm) project.  I don't
want there to be a chance of the project being contaminated with information
stolen from Nintendo's (or Capcom's in this case) actual source code.

I have similar feelings about Link's Awakening, since I've made some
contributions to the [LADX
disassembly](https://github.com/drenn1/oracles-disasm), though I'm not a main
contributor. As such I won't be looking at the leaked source code, and I really
hope the LADX disassembly can continue in a relatively "clean" way.

On the other hand... even if I'm not looking at the source code, there are some
interesting prototype ROMs circulating around of the early development of Link's
Awakening DX. Do I have a moral problem looking at these in the same way as
looking the source code? ...If I do, it's not as strong as my curiosity in this
case. I'm sorry, Nintendo. I am weak to this.

## The dmzel series

There are a large number of ROMs, but so far I've been focusing the "dmzel"
series. There are 5 files: "dmzel", and then "dmzel2" up to "dmzel5".

Before getting first-hand access, I saw speedrunner
[NitrozSR](https://www.twitch.tv/nitrozsr) do a speedrun of dmzel5. It looks
almost exactly like the original black&white release v1.1; DX-exclusive features
such as the color dungeon have not been added yet, and it seems to have all the
same glitches - even the speedrun's Arbitrary Code Execution setup made
specifically for the OG black&white version works! The only difference is...
there's color.

{% include image.html url="/images/obfuscated-ladx-dmzel5-title.png"
description="Note the lack of 'DX' in the title, despite the completeness of the
color" %}

<br>

{% include image.html url="/images/obfuscated-ladx-dmzel5-beach.png" %}

I think it's rather nice to look at! Of course, not all screens look as nice,
as it's clearly not complete. There also don't seem to be very many graphical
changes, such as the changes made to grass in the DX version. Also, certain
effects are missing, such as any "flashing" palettes including Link taking
damage or sword charging.

It's pretty stable, aside from one problem where saving & quitting resets the
game to think it's in monochrome mode, which obviously breaks the color
palettes. RAM address "DD00" is set to "1" when running on a GBC, but this gets
reset to 0 upon saving & quitting, causing the breakage.

{% include image.html url="/images/obfuscated-ladx-dmzel5-palettes-broken.png" 
description="Broken palettes in dmzel5 after a save&quit" %}

dmzel2-4 are similar. I haven't looked too deeply into them, but they seem to
just be earlier versions of dmzel5; for example, dmzel2 and dmzel3 don't display the
menu cursor correctly. And, curiously, you start with the Tail Key in dmzel2
through dmzel4.

{% include image.html url="/images/obfuscated-ladx-dmzel3-menu.png"
description="dmzel3 menu: cursor is missing, you start with the tail key, and
the right side is clearly not finished" %}

dmzel2 has a timestamp of 9 days before dmzel5. I could believe that timeline,
but their timestamps are all in July 1998, placing this less than 6 months
before the final release of Link's Awakening DX. I guess that's possible? It
seems a bit fast to me, but then again, the DX version was simply an update, not
a new game.

There is just one problem with dmzel2 through dmzel5: they don't work on
accurate emulators; only on inaccurate emulators such as VBA. This is because of
a problem with their headers. Each Gameboy ROM contains a header which specifies
what hardware they are meant to run on. In particular, each cartridge contains
hardware called the "[Memory Bank
Controller](https://gbdev.gg8.se/wiki/articles/Memory_Bank_Controllers)" (MBC),
which controls the way in which the cartridge interfaces with the gameboy.

The ROMs state that they use MBC1:

{% include image.html url="/images/obfuscated-ladx-dmzel5-file.png"
description="(Thank you to whoever added Gameboy ROM support to the 'file' command!)" %}

This is peculiar. While it's normal for monochrome Gameboy games to use MBC1,
supposedly, all Gameboy Color games are supposed to use the newer MBC5, at least
if they use the Gameboy Color's double-speed mode.  Apparently MBC1 wasn't
designed for the higher clock rate, although I'm unsure if this was a problem in
practice.

Anyway, the point is - the header is lying. It should be using MBC5, not MBC1.
Changing address 0x147 in the ROM from value 0x03 to value 0x1b fixes the issue.
Since MBC5 is backwards compatible with MBC1, it may be the case that VBA and
other emulators are simply emulating MBC5 all the time, which is why it works on
those emulators.

Of course, the header is only an indicator of the underlying hardware in use. If
Nintendo's debugging hardware used an MBC5, they wouldn't have noticed if they
simply forgot to update the ROM header with that information - it is only of any
benefit to emulators. This might be the reason why the MBC field in the header
is incorrect. (Although I'm unsure how Nintendo's official emulator handled
this. It was apparently part of the leak, but I'd rather not touch sketchy EXE
files...)

## dmzel1

This is where things get weird.

There is a file named "dmzel". I will refer to it as "dmzel1" so as not to
confuse it with others in the series.

dmzel1 appears at first glance to be just another Gameboy ROM file. But it
doesn't work at all. Upon closer inspection, something is very strange: the ROM
is 4 megabytes large. All the others are only 1 megabyte large, just like the
final release!

So I took a closer look. Naturally, I found a gameboy header around address
0x100, easily identfiable by the game's title "ZELDA":

{% include image.html url="/images/obfuscated-ladx-dmzel1-address-00100.png" %}

And again at address 0x4100, which was positively strange:

{% include image.html url="/images/obfuscated-ladx-dmzel1-address-04100.png" %}

And after that it re-occurs at intervals of 0x10000 bytes...

{% include image.html url="/images/obfuscated-ladx-dmzel1-address-10100.png" %}

{% include image.html url="/images/obfuscated-ladx-dmzel1-address-20100.png" %}

At this point I'm screaming at my monitor "Why are there _65 Gameboy headers in
a single file_!?!?" And why is each one spaced exactly 0x10000 bytes apart, with
the exception of the one at 0x4000?

My initial conclusion was that, _for some reason_, the beginning of 65 partial
ROMs were concatenated into a single file. But this wasn't quite correct.

While investigating, I split the file into 64 segments of 0x10000 bytes each,
such that each segment started with one of the gameboy headers. What I found was
that, when I did a binary diff on any 2 segments, there were mostly identical...
except for addresses 0x4000-0x7fff, which were always _radically different_.

I won't get into too much detail, but, a segment of size 0x4000 - such as
addresses 0x4000-0x7fff - corresponds to a single "bank" of data in a normal
Gameboy ROM. It can be thought of as the unit into which all Gameboy ROMs are
divided into. When I observed each of the 64 segments, I found that each
corresponded closely to a specific "bank" in the "dmzel2" ROM (I assumed this
would be the most closely related ROM). And indeed, the first segment - and only
the first segment - contained a Gameboy Header (the one at address 0x4000 in the
original dmzel ROM).

Let's recap. There are 64 segments of size 0x10000 each. Each segment contains
a single unique bank of size 0x4000. The first segment contains a header. 64
banks of size 0x4000 adds up to... 1 megabyte? The expected size for a ROM?

So... I guess I should... concatenate them together??????

This produced a working ROM. Just need to apply the MBC fix like the other dmzel
files, and...

{% include image.html url="/images/obfuscated-ladx-dmzel1.png"
description="I cannot believe that worked." %}

In any case, dmzel1 seems to be what it says on the tin: an even earlier prototype
in the "dmzel" series. It is dated 5 days before dmzel2, which I find
believable. It seems that they haven't even attempted to color the majority of
sprites yet (as seen with Tarin), and attempting to Save&Quit actually crashes
the game. Although the game _does_ save before crashing, so that's good.

### Futher investigation into the "extra" data

I have absolutely no idea why the ROM was stored in this way. In a nutshell, if
you remove 3/4ths of the data in the original "dmzel" file, you get a working
ROM. But the remaining 3/4ths of the file seem to serve no purpose at all.
I looked into it further, and didn't find any answers as to why it may be stored
in this way, but I noticed a few things.

For each segment of size 0x10000 in the "dmzel" file, the first 0x4000 bytes are
identical. That is, bytes 0x0000-0x3fff, 0x10000-0x13fff, 0x20000-0x23fff, etc.
are all identical. This also corresponds exactly to bank 0 of the extracted,
working ROM (bytes 0x4000-0x7fff of dmzel1).

The bytes from offsets 0x4000-0x7fff (ie. 0x14000-0x17fff, 0x24000-0x27fff) all
differ from each other, as explained above; they form the working ROM when
concatenated together.

Bytes from offset 0x8000-0xbfff are similarly identical to one another. They
appear to correspond to offsets 0x30000-0x33fff in the extracted ROM (bank
0x0c). However, compared to the extracted ROM, they are not identical. Luckily,
these consist of graphics, so we can see the difference. It seems like the
graphics have just been shuffled around a bit, not really changed in
a significant way.

{% include image.html url="/images/obfuscated-ladx-dmzel1-gfx-compare-1.png" %}

{% include image.html url="/images/obfuscated-ladx-dmzel1-gfx-compare-2.png"
description="Top: dmzel1 unused data. Bottom: dmzel1 functional rom." %}

Lastly, there are the offsets between 0xc000 and 0xffff. This data appears to
differ in each segment, but only in a few bytes at the end. Also, if you split
it into two halves (compare 0xc000-0xdfff with 0xe000-0xffff) it seems that the
majority of the data is repeated between them.

There was only one part of the data I could comprehend, which was this:

{% include image.html url="/images/obfuscated-ladx-dmzel1-map-layout.png" %}

A rather striking pattern - it's a room layout. If "FF" is considered to mean
"out of bounds", then the dimensions become 10x8, which fits a room's dimensions
perfectly. So I drew this layout in
[LALE](https://www.romhacking.net/utilities/966/), and found that it was the
entrance to Dungeon 3. Assuming I copied it correctly, it appears to correspond
exactly to the black & white version's layout. (The DX version has 2 different
grass tiles.)

{% include image.html url="/images/obfuscated-ladx-dmzel1-d3-entrance.png"
description="Note that palettes are not accurate to dmzel1." %}

This seems to be the only room layout in there, or at least the only one stored
in such an obvious way. In total, it is repeated 128 times in the dmzel1 file
(twice in each of the 64 segments). Why the dungeon 3 entrance? Beats me!

Ultimately I'm still completely baffled as to why this file is arranged in the
way it is. Offsets 0x0000-0x3fff and 0x8000-0xbfff mostly correspond to data
that can be found in the functional ROM. Offsets 0xc000-0xffff do not, at all.
(Room layouts are not normally stored in such an obvious way). There may still
be interesting data in this latter segment, but I'm not sure how to interpret
any of it. (If I had to guess, perhaps it also contains collision and object
data for the D3 entrance screen.)

There is another file like this. It is called "zel1.bin", and it's dated even
before "dmzel". But, even stranger, it is 2.1 megabytes large. I haven't been
able to get it to run...

## Hashes

There were a number of duplicate "dmzel" files. I think they were all the same,
but here are the MD5 hashes, just to be safe. (MBC fix not applied to any of
these.)

65f2bb47a3da496b25f26f601823fc83  dmzel<br>
993737f05c0ad5c106079121c61e467f  dmzel2<br>
1174bd8ecfd6ff22e01fa754708b1e77  dmzel3<br>
da559597564df006edb68bf68a586a1a  dmzel4<br>
1672e1168be72a0483e6b11ff717e542  dmzel5[.com]

And the working ROM extracted from dmzel:

7afd2ead1b7b6abf6340442a5fc59390  dmzel-extracted.gbc
