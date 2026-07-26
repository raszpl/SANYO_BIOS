# Sanyo MBC-17PLUS 2.03SE BIOS reverse engineering

Quick dissasembly to assist with debugging of broken board https://www.vogons.org/viewtopic.php?p=1429483#p1429483

Sanyo BIOS 2.03SE Boot prompt:
```
ROM BIOS Version 2.03SE
CPU Clock 12.5MHz ZERO WAIT
BASE MEMORY SIZE      640K BYTE
EXTENDED MEMORY SIZE  3072K BYTE
```

Sanyo BIOS 2.03SE setup screen:
```
                    ***** SYSTEM INFORMATION *****      Ver 2.00

                              Current Data        New Data

     F1   Date (MM-DD-YYYY)     07-26-2026     :
     F2   Time (HH:MM:SS)        07-28-44      :

     F3   Drive Type  A:       3.5" (1.44MB)   :
     F4               B:       Not Installed   :

     F5   Fixed Type  C:            47         :
     F6               D:       Not Installed   :

     F7   Memory   Base            640 Kb      :
     F8            Extended       3072 Kb      :

     F9   Video Adapter        Enhanced        :
								  Adapter      :

     [W]  Write CMOS
     ESC  Exit

<Help>-------------------------------------------------------------------------
   [F1 - F9] Select function            [W] Write data
   [ESC] Quit
```

What a weird bios! Some highlights:
- uses some 286P Opcodes IDA doesnt like, bios refused to disassemble in 286P mode 😮, switched to 386P to continue.
- very weird code. For example instead of just calling Int15 its doing a song and dance routine of manually loading IVT_SYSTEM_SERVICES_offset (0x54) into BX and calling 'call dword ptr [bx]'.
- ~~Main bios internal routines use some very [weird jump trampoline](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L11959) jumping [all over the place](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L12676), it feels like pointless obfuscation.~~ Turns out its a bytecode VM Sanyo build in order to save space. A lot of BIOS functions are written in bytecode and slowly interpreted, remnant from earlier boards with smaller EPROMs. I didnt attempt to decode the bytecode yet.
- bonkers jumptable trampolines. ~~I still dont understand the second kind they used in SETUP 😀~~ Second kind is a [Switch statement](https://github.com/raszpl/SANYO_BIOS/blob/0e039aa5ffb0ba905850f5220d7662ef9ca7195b/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L12714) :)
- bios fully supports HDDs, but as most early bioses cant define custom ones
- ram test routine is laughable, its pretty much ram presence check not ram test
- no rom checksum check
- uses [3Dh prefix](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L12177) trick to disarm subsequent instruction, lodsb with 3Ch prefix becomes inert "cmp al, 0ACh" allowing multiple callers to reuse/abuse same routine by calling into middle of valid instruction. Clever when used on boards with 8KB ROMs, but tlis 286 platform shipped with over 8KB of free space left in this 32KB rom.

# Sanyo MBC-17PLUS 2.03SE source code
Dissasemble progress 100%, annotation ~80%, bytecode VM 0% decoded. [Listing](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst), [IDA C style Header file](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.h). [IDA 9.3 Free i64 database](https://github.com/raszpl/SANYO_BIOS/raw/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.i64).
