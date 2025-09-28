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


## r2 basics

### Change address

Many of the r2 commands are applied to current address by default. We can change
it by using the `s` (seek) command:
```
[0x00010538]> s main
[0x000107e4]>
```

### Apply command to address

We can apply a command to a given address by using the `@`
operator:
```
[0x00010538]> pdf @ main
            ; DATA XREF from entry0 @ 0x10568(r)
┌ 172: int main (int argc, char **argv); // noreturn
│ `- args(r0, r1) vars(8:sp[0x4..0x40])
│           0x000107e4      00482de9       push {fp, lr}
│           0x000107e8      04b08de2       add fp, var_4h
.......
```

You can read this command as "disassemble function at main address".


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

### List binary sections

In order to list the sections of a binary, we can use:
- `rabin2 -S <file>`
- `iS` in r2

```
$ rabin2 -S /bin/ls
nth paddr          size vaddr         vsize perm flags type        name
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000000      0x0 0x00000000      0x0 ---- 0x0   NULL
1   0x00000318     0x1c 0x00000318     0x1c -r-- 0x2   PROGBITS    .interp
2   0x00000338     0x20 0x00000338     0x20 -r-- 0x2   NOTE        .note.gnu.property
3   0x00000358     0x24 0x00000358     0x24 -r-- 0x2   NOTE        .note.gnu.build-id
.....
```

```
[0x000061d0]> iS
nth paddr          size vaddr         vsize perm flags type        name
―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00000000      0x0 0x00000000      0x0 ---- 0x0   NULL
1   0x00000318     0x1c 0x00000318     0x1c -r-- 0x2   PROGBITS    .interp
2   0x00000338     0x20 0x00000338     0x20 -r-- 0x2   NOTE        .note.gnu.property
3   0x00000358     0x24 0x00000358     0x24 -r-- 0x2   NOTE        .note.gnu.build-id
.....
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

## Code

### List functions

We can list the detected functions with `afl` command:
```
[0x00010538]> afl
0x000104c0    1     12 sym.imp.read
0x000104b4    1     12 sym.imp.printf
0x000104cc    1     12 sym.imp.__stack_chk_fail
0x000104f0    1     12 sym.imp.__libc_start_main
0x00010538    1     56 entry0
0x0001057c    1     28 sym.call_weak_fn
0x000105a0    1     32 sym.deregister_tm_clones
0x000105cc    1     44 sym.register_tm_clones
0x00010604    1     36 entry.fini0
0x0001062c    1      4 entry.init0
0x000108a8    1      8 sym._fini
0x00010690   13    328 sym.getprompt
0x000107e4    4    172 main
0x00010630    3     84 sym.print_warning
0x00010494    1     12 sym._init
0x000104a0    1     16 fcn.000104a0
```
Here we have 3 columns:
- Address: The first column indicates the address of the function inside the
  binary.
- Basic blocks count: The number of basic blocks in the function.
- Name: The name of the symbol. Note that those starting by `sym.imp` are
  imported functions.

We can also list the functions without the imports by filtering those:
```
[0x000107e4]> afl | grep -v sym.imp
0x00010538    1     56 entry0
0x0001057c    1     28 sym.call_weak_fn
0x000105a0    1     32 sym.deregister_tm_clones
0x000105cc    1     44 sym.register_tm_clones
0x00010604    1     36 entry.fini0
0x0001062c    1      4 entry.init0
0x000108a8    1      8 sym._fini
0x00010690   13    328 sym.getprompt
0x000107e4    4    172 main
0x00010630    3     84 sym.print_warning
0x00010494    1     12 sym._init
```

### Disassemble function

We can use the `pdf` command at a function address to disassemble the function:
```
[0x00010538]> pdf @ main
            ; DATA XREF from entry0 @ 0x10568(r)┌ 172: int main (int argc, char **argv); // noreturn
│ `- args(r0, r1) vars(8:sp[0x4..0x40])
│           0x000107e4      00482de9       push {fp, lr}
│           0x000107e8      04b08de2       add fp, var_4h
│           0x000107ec      18d04de2       sub sp, sp, 0x18
│           0x000107f0      10000be5       str r0, [var_10h]           ; 0x10 ; 16 ; argc
│           0x000107f4      14100be5       str r1, [var_14h]           ; 0x14 ; 20 ; argv
.......
```

Additionally, if we are just interested in the opcodes at a specific offset, we
can disassemble them with `pd` function:
```
[0x00010538]> pd 3 @ main+16
│           0x000107f4      14100be5       str r1, [var_14h]           ; 0x14 ; 20 ; argv
│           0x000107f8      18200be5       str r2, [var_18h]           ; 0x18 ; 24
│           0x000107fc      8c309fe5       ldr r3, obj.__stack_chk_guard ; [0x20eb0:4]=0
```


### Decompile function

We need to install the ghidra decompiler:
```
r2pm -U
r2pm -ci r2ghidra
```

Then we can use the `pdg` command to decompile a function:
```
[0x00001060]> pdg @ main

ulong main(int param_1,int64_t param_2)

{
    int iStack_14;
    int iStack_10;
    int iStack_c;
    
    iStack_10 = 0;
    iStack_14 = 0;
    if (1 < param_1) {
        iStack_14 = sym.imp.atoi(*(param_2 + 8));
    }
    for (iStack_c = iStack_14; iStack_c < 10; iStack_c = iStack_c + 1) {
        iStack_10 = iStack_10 + iStack_c;
    }
    sym.imp.printf("Total: %d\n",iStack_10);
    return 0;
}

```


## Data

### Print hexdump

We can print hexdump and values with `px` family command:
```
[0x000061d0]> px 32 @ str.main
- offset -  DADB DCDD DEDF E0E1 E2E3 E4E5 E6E7 E8E9  ABCDEF0123456789
0x0001a6da  6d61 696e 0000 0100 0000 0100 0000 0100  main............
0x0001a6ea  0000 0000 0000 0000 0000 0000 0000 0200  ................
```

Print half words (2 bytes) with `pxh` or `pxH`:
```
[0x000061d0]> pxh 32 @ str.main
0x0001a6da  0x616d 0x6e69 0x0000 0x0001 0x0000 0x0001 0x0000 0x0001  main............
0x0001a6ea  0x0000 0x0000 0x0000 0x0000 0x0000 0x0000 0x0000 0x0002
................
[0x000061d0]>
[0x000061d0]> pxH 16 @ str.main
0x0001a6da 0x616d case.0x485f.127+835
0x0001a6dc 0x6e69 fcn.00006e20+73
0x0001a6de 0x0000 section.
0x0001a6e0 0x0001 section.+1
0x0001a6e2 0x0000 section.
0x0001a6e4 0x0001 section.+1
0x0001a6e6 0x0000 section.
0x0001a6e8 0x0001 section.+1
```

Print words (4 bytes) with `pxw` or `pxW`::
```
[0x000061d0]> pxw 32 @ str.main
0x0001a6da  0x6e69616d 0x00010000 0x00010000 0x00010000  main............
0x0001a6ea  0x00000000 0x00000000 0x00000000 0x00020000  ................
[0x000061d0]>
[0x000061d0]> pxW 16 @ str.main
0x0001a6da 0x6e69616d
0x0001a6de 0x00010000 fcn.00010000
0x0001a6e2 0x00010000 fcn.00010000
0x0001a6e6 0x00010000 fcn.00010000
```

Print quadwords (8 bytes) with `pxq` or `pxQ`:
```
[0x000061d0]> pxq 32 @ str.main
0x0001a6da  0x000100006e69616d  0x0001000000010000   main............
0x0001a6ea  0x0000000000000000  0x0002000000000000   ................
[0x000061d0]>
[0x000061d0]> pxQ 32 @ str.main
0x0001a6da 0x000100006e69616d 
0x0001a6e2 0x0001000000010000 
0x0001a6ea 0x0000000000000000 section.
0x0001a6f2 0x0002000000000000 
```

### Strings
#### Search strings in binary

We can retrieve all the strings in data sections by using:
- `rabin2 -z <file>`
- `iz` in r2

```
[0x000061d0]> iz | head
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x0001a650 0x0001a650 11  12   .rodata ascii dev_ino_pop
1   0x0001a6c8 0x0001a6c8 10  11   .rodata ascii sort_files
2   0x0001a6d3 0x0001a6d3 6   7    .rodata ascii posix-
3   0x0001a6da 0x0001a6da 4   5    .rodata ascii main
4   0x0001a790 0x0001a790 10  11   .rodata ascii ?pcdb-lswd
5   0x0001a7a0 0x0001a7a0 65  66   .rodata ascii # Configuration file for dircolors, a utility to help you set the
6   0x0001a7e2 0x0001a7e2 72  73   .rodata ascii # LS_COLORS environment variable used by GNU ls with the --color option.
```

Resource:
- [Advent of Radare: 04 - Searching strings](https://www.radare.org/advent/04.html)

#### Print string by address

We can use the `ps` command to print a C string (null terminated) in a given
address:
```
[0x000061d0]> ps @ str.shell
shell
```


