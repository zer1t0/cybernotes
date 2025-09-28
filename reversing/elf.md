# ELF

The ELF format is the one used by GNU/Linux programs.

We can inspect an ELF file with [objdump](https://www.man7.org/linux/man-pages/man1/objdump.1.html) or [readelf](https://www.man7.org/linux/man-pages/man1/readelf.1.html) programs.

## Headers

- ELF header: Information about the type of ELF file.
- Program header table: Information to load the program in memory.
- Section header table: Information of the sections.

### ELF Header

The ELF header contains information about the type and contents of the current
file. It is located at the beginning of the file and has the following
structure:
```c

#define EI_NIDENT 16

typedef struct {

    // Starts with magic value: "\x7fELF"
    // e_ident[EI_CLASS] indicates the bits of the ELF file: 32 or 64.
    unsigned char e_ident[EI_NIDENT];

    // Indicates the type of ELF file: executable, relocatable (.o),
    // shared (.so) or core file.
    uint16_t      e_type;

    // Architecture supported: x86, ARM, etc.
    uint16_t      e_machine;

    // File version
    uint32_t      e_version;

    // Entry point address
    ElfN_Addr     e_entry;

    // Program header offset. Zero if no pogram header.
    ElfN_Off      e_phoff;

    // Section header offset. Zero if no section header.
    ElfN_Off      e_shoff;

    // For flags, but unused.
    uint32_t      e_flags;

    // ELF header size in bytes
    uint16_t      e_ehsize;

    // Size of entry in program header table
    uint16_t      e_phentsize;

    // Number of program header entries
    uint16_t      e_phnum;

    // Size of entry in section header table
    uint16_t      e_shentsize;

    // Number of section header entries
    uint16_t      e_shnum;

    // Index of the section header entry that is associated with the string
    // table
    uint16_t      e_shstrndx;
} ElfN_Ehdr;
```

We can read the ELF header of a binary with `readelf -h`:
```
$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x61d0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          149360 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

### Program header

The program header (also known as program header table) is a table that contains
information relative to loading process of the ELF file, so it is only
meaningful (and present) in executable and shared files. It contains the
segments to load but also other information like the interpreter to use for
dynamic loading or if the stack is executable.

Here is the struct of each entry in program header table in ELF of 32 bits:
```c
typedef struct {
    uint32_t   p_type;
    Elf32_Off  p_offset;
    Elf32_Addr p_vaddr;
    Elf32_Addr p_paddr;
    uint32_t   p_filesz;
    uint32_t   p_memsz;
    uint32_t   p_flags;
    uint32_t   p_align;
} Elf32_Phdr;
```

Here is an example of program header with `readelf -l`:
```
$ readelf -l /bin/ls

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x61d0
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000036c0 0x00000000000036c0  R      0x1000
  LOAD           0x0000000000004000 0x0000000000004000 0x0000000000004000
                 0x0000000000015759 0x0000000000015759  R E    0x1000
  LOAD           0x000000000001a000 0x000000000001a000 0x000000000001a000
                 0x0000000000008ed0 0x0000000000008ed0  R      0x1000
  LOAD           0x00000000000232b0 0x00000000000232b0 0x00000000000232b0
                 0x0000000000001310 0x00000000000025f8  RW     0x1000
  DYNAMIC        0x0000000000023d98 0x0000000000023d98 0x0000000000023d98
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  NOTE           0x0000000000000358 0x0000000000000358 0x0000000000000358
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  GNU_EH_FRAME   0x000000000001ef7c 0x000000000001ef7c 0x000000000001ef7c
                 0x00000000000009fc 0x00000000000009fc  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x00000000000232b0 0x00000000000232b0 0x00000000000232b0
                 0x0000000000000d50 0x0000000000000d50  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .text .fini
   04     .rodata .eh_frame_hdr .eh_frame
   05     .init_array .fini_array .data.rel.ro .dynamic .got .got.plt .data .bss
   06     .dynamic
   07     .note.gnu.property
   08     .note.gnu.build-id .note.ABI-tag
   09     .note.gnu.property
   10     .eh_frame_hdr
   11
   12     .init_array .fini_array .data.rel.ro .dynamic .got
```

### Section header

The section header (also known as section header table) is a table that contain
the information of each section, like its location, of the binary file.

```c
typedef struct {
    // Section name
    uint32_t   sh_name;
    uint32_t   sh_type;
    uint32_t   sh_flags;
    Elf32_Addr sh_addr;
    Elf32_Off  sh_offset;
    uint32_t   sh_size;
    uint32_t   sh_link;
    uint32_t   sh_info;
    uint32_t   sh_addralign;
    uint32_t   sh_entsize;
} Elf32_Shdr;
```

We can read the section header of an ELF file with `readelf -S`:
```
$ readelf -S /bin/ls
There are 31 section headers, starting at offset 0x24770:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000020  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000358  00000358
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000037c  0000037c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003a0  000003a0
       00000000000000b8  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           0000000000000458  00000458
       0000000000000be8  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000001040  00001040
       00000000000005d9  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           000000000000161a  0000161a
       00000000000000fe  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          0000000000001718  00001718
       00000000000000d0  0000000000000000   A       7     2     8
  [10] .rela.dyn         RELA             00000000000017e8  000017e8
       0000000000001560  0000000000000018   A       6     0     8
  [11] .rela.plt         RELA             0000000000002d48  00002d48
       0000000000000978  0000000000000018  AI       6    25     8
  [12] .init             PROGBITS         0000000000004000  00004000
       0000000000000017  0000000000000000  AX       0     0     4
  [13] .plt              PROGBITS         0000000000004020  00004020
       0000000000000660  0000000000000010  AX       0     0     16
  [14] .plt.got          PROGBITS         0000000000004680  00004680
       0000000000000030  0000000000000008  AX       0     0     8
  [15] .text             PROGBITS         00000000000046b0  000046b0
       000000000001509e  0000000000000000  AX       0     0     16
  [16] .fini             PROGBITS         0000000000019750  00019750
       0000000000000009  0000000000000000  AX       0     0     4
  [17] .rodata           PROGBITS         000000000001a000  0001a000
       0000000000004f7a  0000000000000000   A       0     0     32
  [18] .eh_frame_hdr     PROGBITS         000000000001ef7c  0001ef7c
       00000000000009fc  0000000000000000   A       0     0     4
  [19] .eh_frame         PROGBITS         000000000001f978  0001f978
       0000000000003558  0000000000000000   A       0     0     8
  [20] .init_array       INIT_ARRAY       00000000000232b0  000232b0
       0000000000000008  0000000000000008  WA       0     0     8
  [21] .fini_array       FINI_ARRAY       00000000000232b8  000232b8
       0000000000000008  0000000000000008  WA       0     0     8
  [22] .data.rel.ro      PROGBITS         00000000000232c0  000232c0
       0000000000000ad8  0000000000000000  WA       0     0     32
  [23] .dynamic          DYNAMIC          0000000000023d98  00023d98
       00000000000001f0  0000000000000010  WA       7     0     8
  [24] .got              PROGBITS         0000000000023f88  00023f88
       0000000000000050  0000000000000008  WA       0     0     8
  [25] .got.plt          PROGBITS         0000000000023fe8  00023fe8
       0000000000000340  0000000000000008  WA       0     0     8
  [26] .data             PROGBITS         0000000000024340  00024340
       0000000000000280  0000000000000000  WA       0     0     32
  [27] .bss              NOBITS           00000000000245c0  000245c0
       00000000000012e8  0000000000000000  WA       0     0     32
  [28] .gnu_debugaltlink PROGBITS         0000000000000000  000245c0
       0000000000000049  0000000000000000           0     0     1
  [29] .gnu_debuglink    PROGBITS         0000000000000000  0002460c
       0000000000000034  0000000000000000           0     0     4
  [30] .shstrtab         STRTAB           0000000000000000  00024640
       000000000000012f  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

## Sections

- **.got**: Contains the addresess of external variables.

- **.got.plt**: Contains the addresses of external functions (like the ones from
  libraries). This one is written in runtime when a external function needs to
  be resolved (lazy approach) or when the program starts in case Full RELRO is
  applied.

- **.plt**: It contains jump instructions using the values stored in
  *.got.plt* section that allow to jump to an external function when its
  address was resolved or jump to the routine to resolve the function address in
  case function was not resolved yet. This section also stores the stubs to
  resolve external functions addresses (that invokes the loader to resolve such
  functions).

## Inspecting ELF

We can inspect an ELF file with [objdump](https://www.man7.org/linux/man-pages/man1/objdump.1.html) or [readelf](https://www.man7.org/linux/man-pages/man1/readelf.1.html) programs, as well
as with another tools like the radare2 or libraries such [pyelftools](https://github.com/eliben/pyelftools).

### Check if ELF is static or dynamic

In order to check if an ELF binary is dynamic, we need to search for the
`INTERP` entry in the program header, that indicates the dynamic linker used by
the program. Here is an example:
```
$ readelf -l /bin/ls | grep INTERP -A 2
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

In case no `INTERP` section is found, it is a static ELF (make sure that is an
executable file).

### Listing ELF sections

We can obtain the sections of a PE file with the commands `objdump -h` or
`readelf -S`:

```
$ objdump -h /bin/ls

/bin/ls:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .interp       0000001c  0000000000000318  0000000000000318  00000318  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.gnu.property 00000020  0000000000000338  0000000000000338  00000338  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.gnu.build-id 00000024  0000000000000358  0000000000000358  00000358  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .note.ABI-tag 00000020  000000000000037c  000000000000037c  0000037c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .gnu.hash     000000b8  00000000000003a0  00000000000003a0  000003a0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .dynsym       00000be8  0000000000000458  0000000000000458  00000458  2**3
....
```

```
$ readelf -S /bin/ls
There are 31 section headers, starting at offset 0x24770:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000020  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000358  00000358
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000037c  0000037c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003a0  000003a0
       00000000000000b8  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           0000000000000458  00000458
       0000000000000be8  0000000000000018   A       7     1     8
....
```

### Search an ELF function offset

```
$ readelf --syms /usr/lib/arm-linux-gnueabihf/libc.so.6 | grep system
  1040: 00044a5c    44 FUNC    WEAK   DEFAULT   12 system@@GLIBC_2.4
```

```
$ objdump --dynamic-syms /usr/lib/arm-linux-gnueabihf/libc.so.6 | grep system
0013c558 g    DF .text	000000a0 (GLIBC_2.4)  svcerr_systemerr
00044a5c  w   DF .text	0000002c  GLIBC_2.4   system
00044a5c g    DF .text	0000002c  GLIBC_PRIVATE __libc_system
```

### Search an ELF string offset

```
$ strings --all --radix=x /usr/lib/arm-linux-gnueabihf/libc.so.6 | grep /bin/sh
 161fcc /bin/sh
```


## Resources

- [man elf](https://man.archlinux.org/man/elf.5.en)
