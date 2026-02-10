---
title: "I2 would like to talk about the Souseiki Fammy"
date: 2026-02-10 15:43:19 +1300
tags: Famicom FDS 6502 assembly hacking BASIC archival I2
---

Pun fully intended.

This is about the I2 Souseiki Fammy, an unlicensed cartridge middleman device capable of interfacing between Famicom cartridges and FDS disks.

- [NESdev Wiki article](https://www.nesdev.org/wiki/Souseiki_Fammy) (mostly complete)
- [Reverse-engineered KiCad schematic](https://github.com/edorfaus/souseiki-fammy-kicad) by edorfaus
- [NESdev Forum discussion](https://forums.nesdev.org/viewtopic.php?t=26064)

## A kit to generate Disk BASIC?

This story began during my search and archival of [Disk BASIC](/2025/12/25/fc-disk-basic). As previously noted, many resources covering Disk BASIC mention a device called the Souseiki Fammy (創世機ファミー) sold by I2 (アイ・ツー). You might know I2 as the company that sold an unlicensed FDS hex editor called Tonkachi Editor, which famously bundled one of the earliest hacks of Super Mario Bros. (commonly called Tonkachi Mario). What you might not know is that they sold a variety of unlicensed FDS software/hardware and provided documentation (manuals, magazines) for FC/FDS programming: Kosodate Gokko for backing up disks, Tonkachi Editor for disk file management/editing/disassembly, and Hokusai/Jingorou for graphics editing. I2 was also responsible for the ILine-PC (Iライン-PC), a device which connects the FDS RAM adapter's expansion port to a typical Japanese computer's printer/parallel port for cross-development purposes. ([Image source](https://www.nesworld.com/article.php?system=nes&data=neshardware_fds))

![A crusty jpeg photo of the ILine-PC and Kodakara-kun disk software.](https://www.nesworld.com/pics/fdscopy.jpg)

The Souseiki Fammy apparently had a piece of software called the "Disk BASIC Generator Kit", which would automatically dump and patch a Family BASIC V2 cart to work as a standalone FDS disk. That sounds nifty, don't you think? No more typing in BASIC programs and hex dumps from magazine articles! It even adds disk saving and 4KiB of extra program memory when used with the Fammy!? You could probably use this to create native development tooling for the system. 

So of course I had to find whatever information I could for this thing. Ootakeke thankfully had plenty of videos on the topic:

- [Fammy overview & ROM-QD for 256K+64K](https://www.youtube.com/watch?v=WxhiE1M3baI) (Donkey Kong)
- [ROM-QD for 256K+64K](https://www.youtube.com/watch?v=z66Kbno-0y8) (F1 Race)
- [Disk BASIC Generator Kit](https://www.youtube.com/watch?v=8XWxWEZ2NaY) (creation)
- [Disk BASIC Generator Kit](https://www.youtube.com/watch?v=drIvX5ijUtQ) (creation & disk saving)

Some more internet sleuthing yielded a list of known Fammy software:

- ROM-QD for 256K+64K - dumps NROM cartridges to a disk format playable on the Fammy.
- B.B RAM Plus - dumps/writes battery-backed RAM contents for cartridges without PRG-RAM protections.
- Disk BASIC Generator Kit - converts a Family BASIC v2 cartridge to a standalone disk, with support for disk saving.
- Slow Plus for ROM - applies slow-down to cartridge games.
- Slow Plus for QD - applies slow-down to disk games.
- Slow Plus for ROM-QD - applies slow-down to disks created using ROM-QD.

Here's a Famitsu advert for the Fammy, complete with Kanji typos ([Image source](https://x.com/oroti_famicom/status/1803589725124718843)):
![Famitsu advert for the Souseiki Fammy.](https://forums.nesdev.org/download/file.php?id=28920&sid=12e4237a360d4454b63495ff19a7329f&mode=view)

Further hardware expansions were announced for the Fammy but unfortunately never came to market. Perhaps the interest in I2's products or FC/FDS hacking in general was waning at this point? Disk software copyright years and magazine articles/adverts date the Fammy to around 1988. ([Image source](https://k1ilove.yu-nagi.com/diskto14.html))

![Advert announcing a "RAM Pack" hardware expansion](https://forums.nesdev.org/download/file.php?id=30229&t=1&sid=28a4deb043e15fdb5a9a3e5e64d3a17c)

Unfortunately, I couldn't find much more in terms of documentation and dumped software for the Fammy. I'm talking about obscure magazine ads, a single dump of ROM-QD, and a single emulator capable of booting it (NintendulatorNRS). English-speaking owners of the Fammy are naturally quite rare. (I could only find one owner on the Famicom World forums, which I still haven't been able to sign up for...) How on Earth was I or any other English-speaking person going to gain access to a working Fammy?

## One Zorched Fammy

November 2025. I received a tip-off from mrmxy regarding a Mercari listing for a Souseiki Fammy. It had the Fammy, four disks covering five out of the known software, and a manual for the Slow Plus software. Even with the Disk BASIC Generator Kit missing, it was well worth a shot.

As much as I would've liked to front the cost myself, I had (and still have) budget issues and international shipping to worry about. Thus, I relayed the info to a few other NESdev members in private to see if anyone else was willing to obtain it instead. In the end, the item was kindly bought by Zorchenhimer. He does a lot of FC R&D as well, with the [Fukutake StudyBox](https://zorchenhimer.com/studybox/) being his primary niche. (Funnily enough, I had previously indirectly convinced him to obtain a rare 1st-generation StudyBox tape. I would again like to extend my apologies and gratitude for both purchases.)

<img src="/assets/images/sf-transparent-unit.png" alt="Famicom Disk System with the Souseiki Fammy plus NROM cart." width="25%"/>

(Photo courtesy of Zorchenhimer.)

Much of the reverse-engineering work was done during Zorchenhimer's streams on Twitch/YouTube. Zorchenhimer dumped the disks using the FDSStick, took photos of the Fammy's board, and disassembled the software (he very much loved the self-modifying code and inline routine parameters). edorfaus used the board photos to [recreate the schematic in KiCad](https://github.com/edorfaus/souseiki-fammy-kicad) and analyse its circuitry. I [translated a scan](https://docs.google.com/document/d/1wnXzFQccC6ueQ1HYYhoUdl8C6BnPaEqQMsadsViRFM8/edit?usp=sharing) of the Slow Plus manual and assisted with the software disassembly, since the FDS was my domain of knowledge. We worked together to determine how the Fammy controls the memory mapping between the cartridge and FDS (it uses discrete logic, which was extra fun to figure out). A [disk port of the Apple WozMon](https://github.com/TakuikaNinja/fdsmon) was even used to probe the hardware in real-time.

A couple fun moments were had during the process, like the time when Slow Plus was used on Super Mario Bros. 2 (aka The Lost Levels): [Twitch clip](https://www.twitch.tv/zorchenhimer/clip/EvilTentativeRatNomNom-ZYnTlwhQHIjWRWi_)

(Twitch embedding sucks, BTW)

## Discrete Logic Madness

I just want to highlight the impressive memory mapping capabilities I2 achieved with the Fammy here. What you're about to read is all done using discrete logic, i.e. 74xx series logic ICs and a single 8 KiB SRAM chip.

The Fammy has two cartridge slots, and a toggle switch on the front of the unit labelled Disk/ROM. This controls whether the Famicom can access the slot for the FDS RAM adapter or the ROM cartridge. The Fammy maps a write-only register at 0x4C00-0x4DFF which can flip the switch's meaning from Disk mode to ROM mode and vice versa. This is the main feature which allows ROM-QD to dump ROM cartridge contents to disk format. The software boots in Disk mode, then temporarily switches to ROM mode in order to read the ROM cartridge for dumping. There are additional bits which control fine-grained handling of cartridge signals, but they aren't important for this explanation.

Now, the Fammy's SRAM mapping sounds confusing until you understand what its software needs to do in order to dump cartridge contents or apply slow-down. By default, one 4 KiB half of the 8 KiB SRAM is always mapped to 0x5000-0x5FFF. It doesn't use 0x6000 onwards because 0x6000-0x7FFF is mapped to either the RAM adapter's PRG-RAM or the ROM cartridge's SRAM (B.B RAM Plus uses the latter). So where does the remaining 4 KiB go? Well, one of the bits in the 0x4C00-0x4DFF register controls whether or not a mirror of the Fammy's SRAM is also mapped to 0xE000-0xFFFF, replacing the FDS BIOS ROM. This is the crux of running ROM-QD disks on the Fammy without modifying the ROM cartridge's code - disks created using ROM-QD bundle a startup program which sets this SRAM mapping mode and moves the final 8 KiB of the original game's PRG data to where it belongs before running the game.

So how is slow-down achieved by those Slow Plus disks when the Twitch clip doesn't have any pause button spam? Simple - you hijack the NMI handler (fired once every frame) to conditionally exit without letting the game's NMI handler code run. To do that, the NMI vector at 0xFFFA needs to be modified to point to a custom handler which *sometimes* allows jumping to the original NMI handler. Thus, the memory range containing the NMI vector needs to be writable, yet there still needs to be memory somewhere to store a custom NMI handler without occupying any memory used by the game. The Fammy solves this problem by providing another bit in the 0x4C00-0x4DFF register to control whether or not the Fammy's mirrored SRAM is mapped to 0xF000-0xFFFF instead of 0xE000-0xFFFF. Does the mapping make sense now? One 4 KiB half of the SRAM is mapped to 0x5000-0x5FFF to store the custom NMI handler, and the other 4 KiB half is mirrored/mapped to 0xF000-0xFFFF to allow modification of the NMI vector. The Slow Plus disks all follow the same strategy of booting in Disk mode, copying the 4 KiB area containing the NMI vector into the lower half of the Fammy's SRAM, modifying the NMI vector (+ making the custom NMI handler in the SRAM's upper half jump to it), then setting the SRAM mapping mode to replace the area it just copied before loading/running the game.

Again, this is all handled using discrete logic and custom disk software. I think I2 deserves some praise for the engineering work on this thing. 

## Closing thoughts

I'm very glad that we know a lot more about what the Souseiki Fammy can do and how it works now. Hopefully, the improved documentation and newly dumped software will result in better emulator support down the line. All that's left to find now is the Disk BASIC Generator Kit... And maybe the ILine-PC stuff, too.

There are still many mysteries left regarding I2, such as their fate after the Famicom/FDS era. If there's any documentation from the period that might shine a light on their activity, I'd love to know.

## Next time...

You might want to look out for something on 2026-02-21, the 40th anniversary of the FDS...

