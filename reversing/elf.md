# ELF

The ELF format is the one used by GNU/Linux programs.

We can inspect an ELF file with [objdump](https://www.man7.org/linux/man-pages/man1/objdump.1.html) or [readelf](https://www.man7.org/linux/man-pages/man1/readelf.1.html) programs.

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
