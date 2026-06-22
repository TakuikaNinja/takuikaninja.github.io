---
title: "Tambo, the Dumbo Sound Driver"
date: 2026-06-22 14:37:29 +1200
tags: NES Famicom music chiptune devlog 6502 assembly Tambo
---

Why I've been writing my own NES/FC sound driver.

The driver/replayer source and demo songs can be found [here](https://github.com/TakuikaNinja/Tambo).

## FamiTracker isn't Game-Ready

Let's not beat around the bush here. It's fairly well-known in the homebrew scene that FamiTracker's current sound driver is rather bloated and inefficient for use in game contexts (to the point where it is one reason behind a driver rewrite). With the exception of MML-driven sound drivers such as NSD.Lib and Pently, most homebrew sound drivers limit themselves to a particular subset of FT features (such as removing effect column support) and tack on a sound effect system.

Take GGSound and Sabre. These two primarily operate by using macros stored in FT instruments - these are essentially repeatable sequences of volume levels, duty cycle/noise mode values, and note/pitch offsets.

One thing I couldn't help but notice with these sound drivers is that they often present software-driven sound design as the only choice. Volume envelopes? Process them in software. Arpeggios and pitch bends? Process them in software. Do this for up to 4 channels (note: FT doesn't support instrument macros for DPCM), and the execution time adds up quickly. Use too many instruments with long macros, and the song size grows like crazy.

Because these sound drivers often provide basic means of converting sound data from FT text exports, they also generally lack support for optimisations such as finite loops (e.g. repeat a sequence of patterns X times) and note transposition out of the box.

I think it's about time we explored other approaches instead of clinging to paradigms laid out by existing music tooling.

## Hardware-driven Sound Design

So if software-driven sound design exists, then that implies the existence of hardware-driven sound design, right? What would that sound like? Well, I'll just point to the early era of first-party titles for some examples:

Gyromite
<iframe width="560" height="315" src="https://www.youtube.com/embed/f1YmeZs91bs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Wrecking Crew
<iframe width="560" height="315" src="https://www.youtube.com/embed/wGS_kq3Lsos" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/-xAOecKb1eA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Gumshoe
<iframe width="560" height="315" src="https://www.youtube.com/embed/HkozwFFgebA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/S9T9w_Jd5ts" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

The benefits of this style are as follows:
- Hardware envelopes are quite suitable for noise percussion or plucky notes.
- Length counter acts as an automatic note cut, so there's usually no need to store these in the song data.
- Hardware sweep can be configured for basic pitch bends (like drums) or for more complex "fake arpeggio" sequences, especially when combined with the hardware envelope.
- Triangle has an additional linear counter, which can be abused for growl sounds like in some Sunsoft games.

The limitations are as follows:
- Volume envelope is either decaying from 15 to 0, looping that decay, or set to a constant volume.
- Hardware sweep is pulse only.
- Duty cycle/noise mode and the above effects can only be set once per note (because setting the high period/length counter byte is what triggers most of them), so no instrument macros whatsoever. (DPCM is surprisingly unaffected by this)

Of course, I can't talk about limited sound drivers without bringing up the soundtrack from the ROM hack called Layla - The Iris Missions:

<iframe width="560" height="315" src="https://www.youtube.com/embed/MNV8cBLgNmU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/ESTXZEN7Yis" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

SupperTails66 did an excellent job wrangling a sound driver which (I think) only has basic software fade-out envelopes and arp macros.

## Working on Tambo

I trialled this hardware-driven sound design in "APU Dance" using FamiTracker as a way of getting out of a chiptune hiatus:

<iframe width="560" height="315" src="https://www.youtube.com/embed/uk5kF894bVQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Yeah, all those "arps" and pitch bends are done with the hardware sweeps.

The satisfaction I had after finishing that song motivated me to write my own sound driver tailored to this style of sound design.

It's called [Tambo](https://github.com/TakuikaNinja/Tambo) (田んぼ, Japanese for rice field) because:
- It rhymes with "dumbo", referencing the dumbness of the driver.
- The 田 is a reference to Hip Tanaka, whom along with Yukio Kaneoka was responsible for the music which inspired me to pursue this style of sound design.

So what makes this a "dumbo" sound driver, then? Well, have a glance at the note data format:
```
apu_dance_pulse2_pattern0:
	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $b1, AS3, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 1, $16, $b1, AS3, $09 << 3
	.byte 1, $16, $b1, C4, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $b1, C4, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $b1, C4, $09 << 3

	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $b1, GS3, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 1, $16, $b1, GS3, $09 << 3
	.byte 1, $16, $b1, GS3, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $b1, GS3, $09 << 3
	.byte 2, $80, $81, C3, $07 << 3
	.byte 2, $16, $c1, FS3, $09 << 3
	.byte 0
```

Each "row" is formatted as follows:
- duration in tracker rows (0 = end pattern)
- register 0 (e.g. volume/duty)
- register 1 (e.g. hardware sweep)
- pulse/triangle note lookup or register 2 (e.g. noise period/mode)
- register 3 (e.g. length counter)

There are some minor differences between channels but the key thing here is that this format is barely above the level of a compressed register log - there's no notion of instruments at all here. All the driver has to do to process a new note for each channel is: read 5 bytes, perform a note lookup if necessary, then write the register values to the APU. SFX follow a very similar format, with the only difference being that they can pick a channel to play on.

An obvious problem with this format is the data size - 5 bytes for every note adds up quickly. I implemented the following commands to help composers address this:
- pattern jump (i.e. for infinite loops)
- finite loop (2 are provided to accomplish 1 level of nesting)
- note transposition (add a signed value to a running offset)
- end song (mainly used for terminating unused channels, since tracks must define data for all 5 of them)

The way these commands are distinguished from regular pattern addresses is simple: commands use addresses lower than $8000 (start of PRG-ROM in most mappers) and use the low byte as an ID. Some commands use the high byte of the address as a parameter, while jumps use an additional address to specify the jump target. Here's an example for the triangle channel which uses everything except "end song":

```
apu_dance_triangle:
	.word apu_dance_blank_pattern
	
	.word CMD::SET_LOOP1 | (12 << 8)
apu_dance_triangle_A:
	.word apu_dance_triangle_pattern0
	.word apu_dance_triangle_pattern1
	.word CMD::LOOP_JUMP1, apu_dance_triangle_A
	
	.word apu_dance_triangle_pattern0
	.word apu_dance_triangle_pattern2
	
	.word CMD::TRANSPOSE | (4 << 8) ; +4 semitones
	.word CMD::SET_LOOP1 | (1 << 8)
apu_dance_triangle_B:
	.word CMD::TRANSPOSE | ((128 - 2) << 8) ; -2 semitones
	.word CMD::SET_LOOP2 | (3 << 8)
@inner:
	.word apu_dance_triangle_pattern0
	.word apu_dance_triangle_pattern1
	.word CMD::LOOP_JUMP2, @inner
	.word CMD::LOOP_JUMP1, apu_dance_triangle_B
	
	.word CMD::JUMP, apu_dance_triangle
```

The above lets me get away with only defining 3 unique patterns for the triangle channel. Nifty, isn't it?

Tambo also has the following features:
- Note cut/rest support for pulse & triangle (though reducing their usage is recommended)
- Pitch/tempo adjustments based on the TV system (so PAL and "Dendy" consoles don't sound terribly off)
- Automatic linear counter trill setting for triangle
- DPCM support

## Audio Examples

I've been posting some audio snippets on Mastodon:

<blockquote class="mastodon-embed" data-embed-url="https://oldbytes.space/@TakuikaNinja/116731131069093389/embed" style="background: #FCF8FF; border-radius: 8px; border: 1px solid #C9C4DA; margin: 0; max-width: 540px; min-width: 270px; overflow: hidden; padding: 0;"> <a href="https://oldbytes.space/@TakuikaNinja/116731131069093389" target="_blank" style="align-items: center; color: #1C1A25; display: flex; flex-direction: column; font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', Roboto, sans-serif; font-size: 14px; justify-content: center; letter-spacing: 0.25px; line-height: 20px; padding: 24px; text-decoration: none;"> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="32" height="32" viewBox="0 0 79 75"><path d="M63 45.3v-20c0-4.1-1-7.3-3.2-9.7-2.1-2.4-5-3.7-8.5-3.7-4.1 0-7.2 1.6-9.3 4.7l-2 3.3-2-3.3c-2-3.1-5.1-4.7-9.2-4.7-3.5 0-6.4 1.3-8.6 3.7-2.1 2.4-3.1 5.6-3.1 9.7v20h8V25.9c0-4.1 1.7-6.2 5.2-6.2 3.8 0 5.8 2.5 5.8 7.4V37.7H44V27.1c0-4.9 1.9-7.4 5.8-7.4 3.5 0 5.2 2.1 5.2 6.2V45.3h8ZM74.7 16.6c.6 6 .1 15.7.1 17.3 0 .5-.1 4.8-.1 5.3-.7 11.5-8 16-15.6 17.5-.1 0-.2 0-.3 0-4.9 1-10 1.2-14.9 1.4-1.2 0-2.4 0-3.6 0-4.8 0-9.7-.6-14.4-1.7-.1 0-.1 0-.1 0s-.1 0-.1 0 0 .1 0 .1 0 0 0 0c.1 1.6.4 3.1 1 4.5.6 1.7 2.9 5.7 11.4 5.7 5 0 9.9-.6 14.8-1.7 0 0 0 0 0 0 .1 0 .1 0 .1 0 0 .1 0 .1 0 .1.1 0 .1 0 .1.1v5.6s0 .1-.1.1c0 0 0 0 0 .1-1.6 1.1-3.7 1.7-5.6 2.3-.8.3-1.6.5-2.4.7-7.5 1.7-15.4 1.3-22.7-1.2-6.8-2.4-13.8-8.2-15.5-15.2-.9-3.8-1.6-7.6-1.9-11.5-.6-5.8-.6-11.7-.8-17.5C3.9 24.5 4 20 4.9 16 6.7 7.9 14.1 2.2 22.3 1c1.4-.2 4.1-1 16.5-1h.1C51.4 0 56.7.8 58.1 1c8.4 1.2 15.5 7.5 16.6 15.6Z" fill="currentColor"/></svg> <div style="color: #787588; margin-top: 16px;">Post by @TakuikaNinja@oldbytes.space</div> <div style="font-weight: 500;">View on Mastodon</div> </a> </blockquote> <script data-allowed-prefixes="https://oldbytes.space/" async src="https://oldbytes.space/embed.js"></script>

<blockquote class="mastodon-embed" data-embed-url="https://oldbytes.space/@TakuikaNinja/116776299512006014/embed" style="background: #FCF8FF; border-radius: 8px; border: 1px solid #C9C4DA; margin: 0; max-width: 540px; min-width: 270px; overflow: hidden; padding: 0;"> <a href="https://oldbytes.space/@TakuikaNinja/116776299512006014" target="_blank" style="align-items: center; color: #1C1A25; display: flex; flex-direction: column; font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', Roboto, sans-serif; font-size: 14px; justify-content: center; letter-spacing: 0.25px; line-height: 20px; padding: 24px; text-decoration: none;"> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="32" height="32" viewBox="0 0 79 75"><path d="M63 45.3v-20c0-4.1-1-7.3-3.2-9.7-2.1-2.4-5-3.7-8.5-3.7-4.1 0-7.2 1.6-9.3 4.7l-2 3.3-2-3.3c-2-3.1-5.1-4.7-9.2-4.7-3.5 0-6.4 1.3-8.6 3.7-2.1 2.4-3.1 5.6-3.1 9.7v20h8V25.9c0-4.1 1.7-6.2 5.2-6.2 3.8 0 5.8 2.5 5.8 7.4V37.7H44V27.1c0-4.9 1.9-7.4 5.8-7.4 3.5 0 5.2 2.1 5.2 6.2V45.3h8ZM74.7 16.6c.6 6 .1 15.7.1 17.3 0 .5-.1 4.8-.1 5.3-.7 11.5-8 16-15.6 17.5-.1 0-.2 0-.3 0-4.9 1-10 1.2-14.9 1.4-1.2 0-2.4 0-3.6 0-4.8 0-9.7-.6-14.4-1.7-.1 0-.1 0-.1 0s-.1 0-.1 0 0 .1 0 .1 0 0 0 0c.1 1.6.4 3.1 1 4.5.6 1.7 2.9 5.7 11.4 5.7 5 0 9.9-.6 14.8-1.7 0 0 0 0 0 0 .1 0 .1 0 .1 0 0 .1 0 .1 0 .1.1 0 .1 0 .1.1v5.6s0 .1-.1.1c0 0 0 0 0 .1-1.6 1.1-3.7 1.7-5.6 2.3-.8.3-1.6.5-2.4.7-7.5 1.7-15.4 1.3-22.7-1.2-6.8-2.4-13.8-8.2-15.5-15.2-.9-3.8-1.6-7.6-1.9-11.5-.6-5.8-.6-11.7-.8-17.5C3.9 24.5 4 20 4.9 16 6.7 7.9 14.1 2.2 22.3 1c1.4-.2 4.1-1 16.5-1h.1C51.4 0 56.7.8 58.1 1c8.4 1.2 15.5 7.5 16.6 15.6Z" fill="currentColor"/></svg> <div style="color: #787588; margin-top: 16px;">Post by @TakuikaNinja@oldbytes.space</div> <div style="font-weight: 500;">View on Mastodon</div> </a> </blockquote> <script data-allowed-prefixes="https://oldbytes.space/" async src="https://oldbytes.space/embed.js"></script>

(The above is a port of "[Hi-Score Party](https://youtu.be/R8sEIPu98zc&t=129)")

## What's Next

I'm planning on doing the following for Tambo:
- Testing, debugging, optimisation passes
- More demo songs
- Polished player interface

I'll obviously be doing other things alongside this long-term project and writing about them, so look forward to those.

