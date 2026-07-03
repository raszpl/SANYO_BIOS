Quick dissasembly to assist with  debugging broken board https://www.vogons.org/viewtopic.php?p=1429483#p1429483

What a weird bios this Sanyo is. Some highlights:
- uses some 286P Opcodes IDA doesnt like, bios refused to disassemble in 286P mode 😮 no idea wtf going on, switched to 386P to continue.
- very weird code. For example instead of just calling Int15 its doing a song and dance routine of manually loading IVT_SYSTEM_SERVICES_offset (0x54) into BX and calling 'call dword ptr [bx]'.
- bonkers jumptable trampolines. I still dont understand the second kind they used 😀
- bios fully supports HDDs, but as most early bioses cant define custom ones
- ram test routine is laughable, its pretty much ram presence check not ram test
- no rom checksum check
