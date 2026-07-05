Quick dissasembly to assist with  debugging broken board https://www.vogons.org/viewtopic.php?p=1429483#p1429483

What a weird bios this Sanyo is. Some highlights:
- uses some 286P Opcodes IDA doesnt like, bios refused to disassemble in 286P mode 😮 no idea wtf going on, switched to 386P to continue.
- very weird code. For example instead of just calling Int15 its doing a song and dance routine of manually loading IVT_SYSTEM_SERVICES_offset (0x54) into BX and calling 'call dword ptr [bx]'.
- Main bios internal routines use some very [weird jump trampoline](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L11959) jumping [all over the place](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L12676), it feels like pointless obfuscation
- bonkers jumptable trampolines. I still dont understand the second kind they used in SETUP 😀
- bios fully supports HDDs, but as most early bioses cant define custom ones
- ram test routine is laughable, its pretty much ram presence check not ram test
- no rom checksum check
- uses [3Dh prefix](https://github.com/raszpl/SANYO_BIOS/blob/0c52d055dd58969eb9034630cffee7366591ca06/Sanyo%20MBC-17PLUS%202.03.BIN.lst#L12177) trick to disarm subsequent instruction, lodsb with 3Ch prefix becomes inert "cmp al, 0ACh" allowing multiple callers to reuse/abuse same routine by calling into middle of valid instruction. Clever at first, until you realize there is over 8KB of free space left in this 32KB rom lol
