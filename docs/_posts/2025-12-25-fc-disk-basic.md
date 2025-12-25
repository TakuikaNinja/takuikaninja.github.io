---
title: "Family BASIC on the FDS, from the 80's"
date: 2025-12-25 13:15:53 +1300
tags: Famicom FDS 6502 assembly hacking BASIC archival
---

## Preface

This is a first-hand recollection of how an unofficial Famicom Disk System (FDS) port of Family BASIC was archived from period Japanese magazines. 

For technical details and Disk BASIC itself, please refer to the following links:

- [Archived Disk BASIC creation tools](https://archive.org/details/fc-disk-basic) (does not contain Disk BASIC itself!)
- [Disk BASIC recreation](https://github.com/TakuikaNinja/FC-DiskBASIC)
- [NESdev Forum discussion](https://forums.nesdev.org/viewtopic.php?t=25171)

## Rediscovering Japan's Famicom hacking scene

Picture this. You're discussing on the NESdev forums about using the FDS timer IRQ for OS-style time-slice operations. You half-jokingly mention that a [Disk Operating System for the FDS would benefit from a port of Family BASIC](https://forums.nesdev.org/viewtopic.php?p=291467#p291467). The official revisions of Family BASIC were all confined to cartridges with battery-backed RAM or cassette tape save functionality. The ability to keep programs on disks would have been a god-send for BASIC programmers at the time, not to mention the hardware upgrades provided by the RAM adapter! Then two months later, you come across a [Japanese blog](http://gameidiri.cocolog-nifty.com/blog/2007/02/basic_f283.html) about putting Family BASIC onto a disk. If you felt nerd-sniped by that, then you'd be in the same boat as myself in January-March of 2024. 

A bit of history since us westerners were obviously insulated from this - During the 80's and early 90's, there was a thriving Famicom hacking scene in Japan. Piracy was an unavoidable motivator of course, but this should not be conflated with genuine developer interest. As you might be able to imagine, Family BASIC and the FDS was their bread and butter. Magazines at the time documented some bizzare and impressive modifications of Famicom hardware/software, including but not limited to:

- Cartridge dumping hardware (they already knew about [Pachicom's developer rant](https://tcrf.net/Pachi_Com_(NES)#Y.S..27_Rant))
- Custom controllers with turbo functionality
- Family BASIC with CHR-RAM for Kanji display
- A MIDI cartridge interface
- An RS232 cartridge interface (this one doesn't use the expansion port)
- A native assembler/disassembler/monitor package
- YM2413 FM sound expansion for the FDS (the wavetable technically has FM, but sure)
- YM2149F PSG sound expansion for the FC (what a *gimmick!*)
- Disabling the FDS drive's write protections for "dubbing" purposes
- Family BASIC V2.1A on FDS ("Disk BASIC")

Upon finding out all of this, I was curious to know if there was a way to obtain Disk BASIC in the modern age. Unfortunately, the resources I initially found never documented the programs nor the patches needed to make a working disk version. The only Yahoo Auction listing I could find for the disk itself expired with an unreasonably high price. I decided to post my findings on the forums to see if anyone else knew of it. 

In September 2024, marioboy replied on the forum thread with a link to a Japanese video by Ootakeke (おおたけけ) covering the [I2 Disk BASIC Generator Kit](https://www.youtube.com/watch?v=drIvX5ijUtQ). This is a piece of software for I2's Souseiki Fammy device, which as the name implies, generates a copy of Disk BASIC by dumping a retail cartridge to disk and automatically applying the required patches. I checked the creator's channel page and found an [older video](https://www.youtube.com/watch?v=8XWxWEZ2NaY) on the same topic. 

Although the videos revealed how the graphics tool and disk saving worked, they didn't result in any useful leads. (Unlicensed companies such as I2 and Hacker International also made attempts to turn the FDS into an open development platform. Their stories and products are less relevant for Disk BASIC, however.) The thread laid dormant for some time. 

## The patches were there this whole time?

January 2025. I decided to do an online search for Disk BASIC like I had been doing every now and then. One unfamiliar website catches my eye. I opened the page and was shocked - it contained [Japanese instructions](http://cmpslv3.stars.ne.jp/Konjo/027/027.htm) to create a disk image for Disk BASIC, along with the full patch list! I then found out that this was written by Enri, a Japanese blogger/researcher I was already familiar with from their documentation of the FC/FDS. "How long has this page been up for?" I thought as I checked the Wayback Machine for its oldest hit. It listed a hit for January 1st 2024 - it was up for at least a year, possibly even longer. 

I just had to follow the instructions to make a copy of Disk BASIC. It booted up and ran just fine in an emulator. I made a [CA65 repository](https://github.com/TakuikaNinja/FC-DiskBASIC) to automate the creation process, and uploaded a YouTube video showcasing the program. 

<iframe width="512" height="480" src="https://www.youtube.com/embed/NyzgAresfmc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

There was just one problem - disk saving, arguably its most important feature, was missing. 

## PLEASE BA DISK CARD

The lack of disk saving in the version I finally found left me unsatisfied. Enri's page claimed that these instructions originally came from Famicom Hacking Manual Vol. 2. Perhaps Vol. 3 had an article for adding disk saving? I already knew someone online who had a copy but was understandably unwilling/unable to share scans. You see, the publisher (Sansai Books) is still active and currently sells digital editions of their books exclusively in Japan. Articles from Famicom Hacking Manual have been reissued in some of their recent publications, so it's sensible to assume that they don't want scans of it being shared around willy-nilly on the internet (Japan is particularly strict about copyright). 

Some time later in April 2025, I received a comment on the YouTube video. The commenter (InfiniteEnd) said they were going to purchase copies of Famicom Hacking Manual to assist with the archival process. I couldn't really refuse, so I directed them to the forums and provided the page numbers I had found from contents pages online. They scanned the pages and shared them with me in private. I confirmed that Vol. 3 indeed added disk saving, and went to work reassembling everything. 

In case you didn't know how program listings in magazines worked at the time: Type-in machine code programs were often provided in the form of binary hex dumps printed across one or more pages. You would have to manually assemble them with a hex editor program, usually one running on the target system itself. The disk programs used to create Disk BASIC were expected to be created using an unlicensed FDS hex editor called Tonkachi Editor (famous for bundling one of the earliest hacks of Super Mario Bros.). As far as I know, it only provides a controller interface for inputting hex values. 

I obviously didn't want to suffer through that, so I used a modern PC hex editor called [ImHex](https://imhex.werwolv.net/) instead. The process was still tedious but I managed to replicate the entire Disk BASIC creation process with the aid of an emulator. During the process, I found some errors in the listings. This was a known issue mentioned on the first Japanese blog I found, so I corrected what I thought were things that would have reasonably been fixed in official corrections. Low and behold, disk saving worked! 

<iframe width="512" height="480" src="https://www.youtube.com/embed/SPrCRbijkr8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

I updated the repository to support disk saving in May 2025, with the previous version moved to a separate branch for safekeeping. I also uploaded the fixed creation tools to the [Internet Archive](https://archive.org/details/fc-disk-basic) in June 2025, along with the patches and instructions. 

## Closing thoughts

I know what you're thinking by now: "All this effort for a port of Family BASIC V2.1A, and not V3.0?" I can only say that it should *theoretically* be possible to port V3.0 in the same manner, though you'll likely have to ensure the built-in BASIC programs can never be called afterwards (they get removed during the process). This wasn't the point of this archival project, however. I believe that Disk BASIC on the Famicom is an impressive feat by the early Famicom hacking scene which shouldn't be diminished. They managed to bring the FC/FDS on par with other 8-bit home computers of the era (MSX, SC-3000, etc.) when Nintendo refused to do so. It's a relic from an often overlooked part of the Famicom's history. 

This story of archiving native FC/FDS development tools isn't over, however. Things such as the native assembler/disassembler/monitor package and I2's Souseiki Fammy, ILine-PC, and their software are yet to be fully archived. If you or someone you know owns one of these things, please reach out on the NESdev forums or Discord server so we can assist with the archival/research process. 

## Next time...

Have you ever wanted to dump NROM carts to FDS disks? How about playing games with true slowdown instead of spamming the pause button? Next time, I'll be going over the current state of research on the Souseiki Fammy. 

