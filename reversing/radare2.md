# Radare2

Radare2 or r2 is a reversing engineering framework created by
[pancake](https://infosec.exchange/@pancake).

## Installation

You can install it from its repository which also includes installation
instructions:

- [Radare2](https://github.com/radareorg/radare2)

## Tutorials

Here are some great resources to learn how to use r2:

- [A journey into Radare 2 – Part 1: Simple
  crackme](https://www.megabeets.net/a-journey-into-radare-2-part-1/) by Megabeets
- [A journey into Radare 2 – Part 2:
  Exploitation](https://www.megabeets.net/a-journey-into-radare-2-part-2/) by Megabeets
- [Radare2 Book](https://book.rada.re/)
- [Advent of Radare](https://radare.org/advent/)

## Executable inspection

We can use r2 or rabin2 to inspect an executable file and extract information of
different parts. You can think of rabin2 as an objdump on steroids.

The advantage of using `rabin2` over `r2` is that we avoid to load the whole
binary into r2, which is faster.

### Retrieve general binary information

To retrieve general information about a binary we can use one of the following
commands:
- `rabin2 -I <file>`
- `r2 -q -c i <file>`

Here is an example of the former that avoids to load the binary into r2:
```shell
$ rabin2 -I ntdll.dll 
arch     x86
baddr    0x180000000
binsz    2029120
bintype  pe
bits     64
canary   true
injprot  false
retguard false
class    PE32+
cmp.csum 0x001f3ccd
compiled Sun Aug  3 18:20:27 2025
crypto   false
dbg_file ntdll.pdb
endian   little
havecode true
hdr.csum 0x001f3ccd
guid     0AF9B92CB2856CFB0C54EC42CE8E33771
laddr    0x0
lang     c
linenum  false
lsyms    false
machine  AMD 64
nx       true
os       windows
overlay  true
cc       ms
pic      true
relocs   false
signed   true
sanitize false
static   true
stripped false
subsys   Windows CUI
va       true
```

### Extract binary exports

To get binary imports we can use:
- `rabin2 -E <file>`
- `r2 -q -c iE <file>`

Here is an example of `rabin2 -E`:
```shell
$ rabin2 -E ntdll.dll
nth  paddr      vaddr       bind   type size lib       name                                                  demangled
――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
8    0x0007ef10 0x18007fb10 GLOBAL FUNC 0    ntdll.dll Ordinal_8
9    0x0003f640 0x180040240 GLOBAL FUNC 0    ntdll.dll A_SHAFinal
10   0x00040470 0x180041070 GLOBAL FUNC 0    ntdll.dll A_SHAInit
11   0x000404b0 0x1800410b0 GLOBAL FUNC 0    ntdll.dll A_SHAUpdate
12   0x000dfbe0 0x1800e07e0 GLOBAL FUNC 0    ntdll.dll AlpcAdjustCompletionListConcurrencyCount
13   0x00070b30 0x180071730 GLOBAL FUNC 0    ntdll.dll AlpcFreeCompletionListMessage
14   0x000dfc10 0x1800e0810 GLOBAL FUNC 0    ntdll.dll AlpcGetCompletionListLastMessageInformation
15   0x000dfc30 0x1800e0830 GLOBAL FUNC 0    ntdll.dll AlpcGetCompletionListMessageAttributes
...
```
