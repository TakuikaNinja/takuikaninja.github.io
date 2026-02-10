---
layout: default
title: "Family BASIC Typeins"
permalink: /family-basic-typeins/
---

# {{ page.title }}

This page hosts Family BASIC typeins archived from period publications. Shorter listings are presented in code blocks here and longer listings are hosted in external repositories.

## Family BASIC V2

V2.0A & V2.1A. Compatibility with V3.0 is not guaranteed.

### Disassembler/Assembler/Monitor

Three typeins by Norihito Ushijima (牛島憲人) from Famicom Hacking Manual (ファミコン改造マニュアル) Vol. 3 which can disassemble 6502 machine code, assemble 6502 code (bytecode preview only), and view/edit memory contents.

[Repo](https://github.com/TakuikaNinja/FamilyBASIC-DAM)

![Disassembler screen](https://github.com/TakuikaNinja/FamilyBASIC-DAM/raw/main/disassembler/FC-D.ASM.png)
![Assembler screen](https://github.com/TakuikaNinja/FamilyBASIC-DAM/raw/main/assembler/FC-ASM.png)
![Monitor Screen](https://github.com/TakuikaNinja/FamilyBASIC-DAM/raw/main/monitor/FC-MONITOR_V2.png)

### CHR-RAM Mod

A CHR-RAM modification of Family BASIC by Saburou Tama (多摩三郎) was documented in Backup Utilisation Techniques (バックアップ活用テクニック) Part 4. The hardware modification replaces the sprite pattern table with CHR-RAM, allowing for custom graphics while keeping the text characters (background pattern table) in CHR-ROM. The CHR-ROM data was intended to be dumped to cassette tape beforehand so it could be reloaded later on. Typeins for the SHARP X1 were also provided to enable the loading, editing, and exporting of this data.

TODO:
- SHARP X1 BASIC programs?

#### Kanji Display

This typein accompanied the hardware modification article to demonstrate the custom graphics.

```BASIC
10 CLEAR &H7700
20 RESTORE 200
30 FOR I=0TO 41
40 READ A$:A=VAL("&H"+A$)
50 POKE &H7700+I,A
60 NEXT
70 GOTO 300
80 '
200 DATA A9,0B,85,63              'LDA #0B       STA 63
210 DATA A5,63,D0,FC              'LDA 63        BNE FC
220 DATA AD,80,77,8D,06,20        'LDA 7780      STA 2006
230 DATA AD,81,77,8D,06,20        'LDA 7781      STA 2006
240 DATA AD,82,77,8D,07,20        'LDA 7782      STA 2007
250 DATA A5,9D,8D,05,20           'LDA 9D        STA 2005
260 DATA A5,69,8D,05,20           'LDA 69        STA 2005
270 DATA A9,1E,8D,01,20,60        'LDA 69   STA 2001   RTS
280 '
300 SPRITE ON
310 DEF SPRITE 0,(0,1,0,0,0)=CHR$(0)+CHR$(1)+CHR$(2)+CHR$(3)
320 DEF SPRITE 1,(0,1,0,0,0)=CHR$(4)+CHR$(5)+CHR$(6)+CHR$(7)
330 SPRITE 0,180,200
340 SPRITE 1,200,200
350 '
360 POKE &H7780,0
370 RESTORE 500
380 FOR I=0 TO &H7F
390 POKE &H7781,I
400 READ A$:A=VAL("&H"+A$)
410 POKE &H7782,A
420 CALL &H7700
430 NEXT
440 END
450 '
500 DATA 41,2F,01,01,87,44,04,07,00,00,00,00,00,00,00,00
510 DATA 10,FE,10,10,FC,44,44,FC,00,00,00,00,00,00,00,00
520 DATA 20,27,20,4F,40,41,82,8C,00,00,00,00,00,00,00,00
530 DATA 40,FC,40,FE,A0,10,08,06,00,00,00,00,00,00,00,00
540 DATA 01,01,FF,80,80,1F,00,00,00,00,00,00,00,00,00,00
550 DATA 00,00,FE,02,02,F0,20,40,00,00,00,00,00,00,00,00
560 DATA 00,01,FF,01,01,01,01,07,00,00,00,00,00,00,00,00
570 DATA 80,00,FE,00,00,00,00,00,00,00,00,00,00,00,00,00
```

![Kanji display](/assets/images/fcbasic-kanji-display.png)

#### CHR-ROM Dumping

This typein dumps the two pattern tables into separate tape files. 

```BASIC
10 CLEAR &H7700
20 RESTORE 200
30 FOR I=0 TO 52
40 READ A$:A=VAL("&H"+A$)
50 POKE &H7700+I,A
60 NEXT
70 GOTO 400
80 '
200 DATA A9,0C,85,63          'LDA #0C   STA 63
210 DATA A5,63,D0,FC          'LDA 63    BNE FC
220 DATA AD,80,77             'LDA 7780
230 DATA 8D,06,20             'STA 2006
240 DATA AD,81,77             'LDA 7781
250 DATA 8D,06,20             'STA 2006
260 DATA A2,00                'LDX #00
270 DATA AD,07,20             'LDA 2007
280 DATA AD,07,20             'LDA 2007
290 DATA 9D,00,07             'STA 0700,X
300 DATA E8,D0,F7             'INX        BNE F7
310 DATA A5,9D,8D,05,20       'LDA 9D     STA 2005
320 DATA A5,69,8D,05,20       'LDA 69     STA 2005
330 DATA A9,0B,85,63          'LDA #0B    STA 63
340 DATA A5,63,D0,FC          'LDA 63     BNE FC
350 DATA 60                   'RTS
360 '
400 FOR I=0 TO 1
410 POKE &H500,4              '04=ASC
420 POKE &H501,ASC("F"),ASC("O"),ASC("N"),ASC("T"),0,13
430 POKE &H512,2,1            '0102 Bytes
440 POKE &H514,&HFE,6         'Start=06FE
450 POKE &H516,0,7
460 CALL &HB4FC
470 FOR J=0 TO 15 :PRINT J
480 POKE &H6FE,0
490 POKE &H6FF,J
500 IF J=15 THEN POKE &H6FF,&HFF
510 POKE &H7780,I*16+J
520 POKE &H7781,0
530 CALL &H7700
540 CALL &HB50D
550 NEXT:NEXT
```

#### CHR-RAM Loading

Note that this only loads one tape file, since the hardware modification only uses CHR-RAM for one of the pattern tables. The article demonstrates this by loading the X1's character set into CHR-RAM, then running `CGEN1` to map the background pattern table to it. (CTRL+D to revert the mapping)

The original listing erroneously used `&H7F8x` in lines 430/440 instead of the `&H778x` addresses used elsewhere. This has been corrected in the listing below.

```BASIC
10 CLEAR &H7700
20 RESTORE 200
30 FOR I=0 TO 52
40 READ A$:A=VAL("&H"+A$)
50 POKE &H7700+I,A
60 NEXT
70 GOTO 400
80 '
200 DATA A9,0C,85,63          'LDA #0C   STA 63
210 DATA A5,63,D0,FC          'LDA 63    BNE FC
220 DATA AD,80,77             'LDA 7780
230 DATA 8D,06,20             'STA 2006
240 DATA AD,81,77             'LDA 7781
250 DATA 8D,06,20             'STA 2006
260 DATA A2,00                'LDX #00
270 DATA BD,00,07             'LDA 0700,X
280 DATA AD,07,20             'STA 2007
300 DATA E8,D0,F7             'INX        BNE F7
310 DATA A5,9D,8D,05,20       'LDA 9D     STA 2005
320 DATA A5,69,8D,05,20       'LDA 69     STA 2005
330 DATA A9,0B,85,63          'LDA #0B    STA 63
340 DATA A5,63,D0,FC          'LDA 63     BNE FC
350 DATA 60                   'RTS
360 '
400 CALL &HB430
410 FOR I=0TO 15
420 CALL &HB401
430 POKE &H7780,I
440 POKE &H7781,0
450 CALL &H7700
460 NEXT
```

## Disk BASIC

Disk BASIC is an unofficial Famicom Disk System (FDS) port of Family BASIC V2.1A, originally documented as a manual process in Backup Utilization Techniques Part 8 and Famicom Hacking Manual Vol. 2 & 3. I2 would later release the Disk BASIC Generator Kit for their [Souseiki Fammy](/2026/02/10/i2-souseiki-fammy) to automate the process and provide additional features.

Please visit [this post](/2025/12/25/fc-disk-basic) for more information on Disk BASIC.

### Character Editor

A graphics editor for Disk BASIC, primarily targeting the 16x16 sprite characters. Data can be saved to/loaded from cassette tape.

[Repo](https://github.com/TakuikaNinja/FC-DiskBASIC-CharEditor)

![Character editor menu](https://github.com/TakuikaNinja/FC-DiskBASIC-CharEditor/raw/main/img/menu.png)
![Character editor screen](https://github.com/TakuikaNinja/FC-DiskBASIC-CharEditor/raw/main/img/editor.png)

### Creation Tools

These typeins were provided in the Disk BASIC article to dump the Family BASIC cartridge data to tape format, in order to load them into a custom FDS program which creates Disk BASIC.

#### CHR-ROM Dumping

This is a modification of the typein from the CHR-RAM mod listed above. The CHR data is saved as a single file instead of two separate ones.

```BASIC
10 CLEAR &H7700
20 RESTORE 200
30 FOR I=0 TO 52
40 READ A$:A=VAL("&H"+A$)
50 POKE &H7700+I,A
60 NEXT
70 GOTO 400
80 '
200 DATA A9,0C,85,63          'LDA #0C   STA 63
210 DATA A5,63,D0,FC          'LDA 63    BNE FC
220 DATA AD,80,77             'LDA 7780
230 DATA 8D,06,20             'STA 2006
240 DATA AD,81,77             'LDA 7781
250 DATA 8D,06,20             'STA 2006
260 DATA A2,00                'LDX #00
270 DATA AD,07,20             'LDA 2007
280 DATA AD,07,20             'LDA 2007
290 DATA 9D,00,07             'STA 0700,X
300 DATA E8,D0,F7             'INX        BNE F7
310 DATA A5,9D,8D,05,20       'LDA 9D     STA 2005
320 DATA A5,69,8D,05,20       'LDA 69     STA 2005
330 DATA A9,0B,85,63          'LDA #0B    STA 63
340 DATA A5,63,D0,FC          'LDA 63     BNE FC
350 DATA 60                   'RTS
360 '
400 '
410 POKE &H500,4              '04=ASC
420 POKE &H501,ASC("F"),ASC("O"),ASC("N"),ASC("T"),0,13
430 POKE &H512,2,1            '0102 Bytes
440 POKE &H514,&HFE,6         'Start=06FE
450 POKE &H516,0,7
460 CALL &HB4FC
470 FOR I=0 TO 31:PRINT I;
480 POKE &H6FE,0
490 POKE &H6FF,I
500 IF I=31 THEN POKE &H6FF,&HFF
510 POKE &H7780,I
520 POKE &H7781,0
530 CALL &H7700
540 CALL &HB50D
550 NEXT
```

#### PRG-ROM Dumping

This typein dumps the PRG-ROM data.

```BASIC
10 POKE &H500,1            'ファイルの種類はマシン語
20 POKE &H501,ASC("B")     'ファイルネーム
30 POKE &H502,ASC("A")
40 POKE &H503,ASC("S")
50 POKE &H504,ASC("1"),0,13
60 POKE &H512,&HEF,&H5F    '8000Hから4000Hバイトセーブする
70 POKE &H514,0,&H80
80 POKE &H516,0,0
90 CALL &HB4FC:CALL &HB50D
```

Translation of Japanese comments (`'` is the shorthand for `REM`):
- ファイルの種類はマシン語 = file type is machine language
- ファイルネーム = file name
- 8000Hから4000Hバイトセーブする = save 4000H bytes of data starting from 8000H

The third comment lists an incorrect data size. The POKE statement on that line (60) has 0x5FEF in little endian, so this is actually 0x5FF0 bytes between 0x8000-0xDFEF. This error is safe to ignore as comments are typically omitted when typing in listings.

## Family BASIC V3

`SORRY NOTHING` - *Super Boy I* (SMS)


