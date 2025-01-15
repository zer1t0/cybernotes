# WinDbg

## Installation

You can [install WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)
from the [Microsoft Store](https://apps.microsoft.com/detail/9pgjgd53tn86). 

## Symbols

An important part to make things easier when debugging/reversing is to have as
much symbols as you can get. In Windows, symbols for binaries can be found in
[.pdb](https://en.wikipedia.org/wiki/Program_database) files.

### Loading symbols into WinDbg

In my experience if you install the latest WinDbg from the Microsoft Store, it
automatically retrieves the symbols for you for the loaded modules, so it should
work nicely.

On the contrary, the short answer is:
```
.sympath srv*C:\symbols*https://msdl.microsoft.com/download/symbols
```
This will indicate WinDbg to retrieve symbols from Microsoft symbol server and
store them into `C:\symbols`. You may need to apply `.reload` after changing
symbols path.

This being said, the long answer starts by checking the current symbols path,
which you can with the
[.sympath](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-sympath--set-symbol-path-)
meta command without arguments:
```
0:000> .sympath 
Symbol search path is: srv*
Expanded Symbol search path is: cache*;SRV*https://msdl.microsoft.com/download/symbols

************* Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       srv*
```
In this case the symbol path is
`cache*;SRV*https://msdl.microsoft.com/download/symbols`. The `cache*` directive
indicates that that symbols are cached in the default cache directory of the
system, and `SRV*https://msdl.microsoft.com/download/symbols` indicates that
those symbols that are not in the cache will be retrieved from the Microsoft
public symbol server (https://msdl.microsoft.com/download/symbols ). 

> You can find more information about the symbols path syntax in [Symbol path
> for Windows debuggers]( https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-path).

Once you have checked the symbol path, you may want modify it, that can also be
done, as stated in the short answer, with `.sympath`. Here are some examples
with different syntax:

- `.sympath C:\symbols` : Symbols will be retrieved from `C:\symbols`
  directory.
- `.sympath srv*C:\symbols*https://msdl.microsoft.com/download/symbols`: Symbols
  will be retrieved from Microsoft symbols server and cached into `C:\symbols`.
- `.sympath cache*C:\symbols;srv*https://msdl.microsoft.com/download/symbols`:
  Similar to previous example, but if add more symbol servers, all will be
  cached in `C:\symbols`.

And here is a command example:
```
0:000> .sympath srv*C:\symbols*https://msdl.microsoft.com/download/symbols
Symbol search path is: srv*C:\symbols*https://msdl.microsoft.com/download/symbols
Expanded Symbol search path is: srv*c:\symbols*https://msdl.microsoft.com/download/symbols

************* Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       srv*C:\symbols*https://msdl.microsoft.com/download/symbols
```

As we have said before, after applying `.sympath`, in case symbols are not
reloaded, you may need to use the `.reload` command.

### Download symbols for an offline machine

> Based on [Fix Your (Offline)
Symbols](https://www.osr.com/nt-insider/2015-issue1/fix-offline-symbols/).

In case you want to download symbols for a machine without internet connectivity
you can follow the next steps.

In the non-internet machine you must use
[symchk](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/using-symchk)
(it comes with the [Debugging Tools for
Windows](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)).

The following command will create a manifest.txt file that includes the
definition of symbols that required to download:
```
symchk /om manifest.txt /ie <executable-for-which-we-want-symbols> /s c:\symbols
```

To speed up the search for symbols (that is what `symchk` does) you can create
an empty folder with `mkdir C:\symbols`, so `symchk` fails faster.

Once you have the `manifest.txt` file, you must copy it to a machine with
internet connection and passed it as argument to `symchk` with the following
command:
```
symchk /im manifest.txt /s SRV*C:\Symbols*http://msdl.microsoft.com/download/symbols
```

This will download the symbols to the `C:\Symbols` folder, that you can copy to
the machine without internet connection. 

Finally you have the symbols in your isolated machine. In WinDbg remember to set
`C:\symbols` as source in the symbol path. You can do it with:
```
.sympath C:\symbols
```

## About Commands

In these notes you will find many different commands of WinDbg explained. The
intention focused on explaining common situations with WinDbg, so the commands
are not explained exhaustively, but you can refer to the
[official documentation for WinDbg
commands](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/debugger-commands). 
Moreover, just let you know that there different command types:
- **Regular Commands**: Commands that affect the debuggee (program being debugged).
- **Meta commands**: Also called *dot commands* because they always start with a
  dot, like `.sympath` or `.reload`. They are related to configuration of the debugger
  itself, not the debuggee.
- **Extension commands**: Commands provided by extensions. These commands start
  with "!".
  
## Registers

You can view register values with the *Registers View* or the [r
command](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/r--registers-). Here
is an example:

```
0:000> r
rax=0000000000000000 rbx=00007ffe37b75910 rcx=00007ffe37aed994
rdx=0000000000000000 rsi=0000000000000010 rdi=00000032e3311000
rip=00007ffe37b207a0 rsp=00000032e35fefa0 rbp=0000000000000000
 r8=00000032e35fef98  r9=0000000000000000 r10=0000000000000000
r11=0000000000000246 r12=0000000000000040 r13=0000000000000000
r14=00007ffe37b75800 r15=00000218f9e80000
iopl=0         nv up ei pl zr na po nc
cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
ntdll!LdrpDoDebuggerBreak+0x30:
00007ffe`37b207a0 cc              int     3
```

You can also see just one register with `r`:
```
0:000> r rsp
rsp=00000032e35fefa0
```

Another possibility is to use the evaluation command `?`:
```
0:000> ? rsp
Evaluate expression: 218563080096 = 00000032`e35fefa0
```

In order to set a registry value, the `r` command can be used (again):
```
0:000> r rax=1
```

## Disassemble

In order to dissasemble functions and instructions, you have some commands that
starts with `u*` (for Unassemble), the most raw command is
[u](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/u--unassemble-),
that will print the dissasembly from the provided memory address.

Here is an example with `NtAllocateVirtualMemory` function:
```
0:000> u ntdll!NtAllocateVirtualMemory
ntdll!NtAllocateVirtualMemory:
00007ffb`cdfed2d0 4c8bd1          mov     r10,rcx
00007ffb`cdfed2d3 b818000000      mov     eax,18h
00007ffb`cdfed2d8 f604250803fe7f01 test    byte ptr [SharedUserData+0x308 (00000000`7ffe0308)],1
00007ffb`cdfed2e0 7503            jne     ntdll!NtAllocateVirtualMemory+0x15 (00007ffb`cdfed2e5)
00007ffb`cdfed2e2 0f05            syscall
00007ffb`cdfed2e4 c3              ret
00007ffb`cdfed2e5 cd2e            int     2Eh
00007ffb`cdfed2e7 c3              ret
```

Moreover, in order to dissasemble a function, you may find the
[uf](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/uf--unassemble-function-)
command more appropiate.

Here is the example for the very same `NtAllocateVirtualMemory` function:
```
0:000> uf ntdll!NtAllocateVirtualMemory
ntdll!NtAllocateVirtualMemory:
00007ffb`cdfed2d0 4c8bd1          mov     r10,rcx
00007ffb`cdfed2d3 b818000000      mov     eax,18h
00007ffb`cdfed2d8 f604250803fe7f01 test    byte ptr [SharedUserData+0x308 (00000000`7ffe0308)],1
00007ffb`cdfed2e0 7503            jne     ntdll!NtAllocateVirtualMemory+0x15 (00007ffb`cdfed2e5)  Branch

ntdll!NtAllocateVirtualMemory+0x12:
00007ffb`cdfed2e2 0f05            syscall
00007ffb`cdfed2e4 c3              ret

ntdll!NtAllocateVirtualMemory+0x15:
00007ffb`cdfed2e5 cd2e            int     2Eh
00007ffb`cdfed2e7 c3              ret
```

As you may notice, the `uf` command differentiates the branches of the
function. This can be nice but in some (rare) occasions, a function just jumps
into another and the output can be confuse.

This is the case when dissasembly `NtQuerySystemTime` with `uf`:
```
0:000> uf ntdll!NtQuerySystemTime
ntdll!RtlQuerySystemTime:
00007ffe`37ac6f80 488b04251400fe7f mov     rax,qword ptr [SharedUserData+0x14 (00000000`7ffe0014)]
00007ffe`37ac6f88 488901          mov     qword ptr [rcx],rax
00007ffe`37ac6f8b eb02            jmp     ntdll!RtlQuerySystemTime+0xf (00007ffe`37ac6f8f)  Branch

ntdll!RtlQuerySystemTime+0xf:
00007ffe`37ac6f8f 33c0            xor     eax,eax
00007ffe`37ac6f91 c3              ret

ntdll!NtQuerySystemTime:
00007ffe`37aee020 e95b8ffdff      jmp     ntdll!RtlQuerySystemTime (00007ffe`37ac6f80)  Branch
```

As you may notice, it shows parts of another function `RtlQuerySystemTime`
because `NtQuerySystemTime` just jumps directly into it.

## Breakpoints

The commands started by *b* are related to breakpoints.

The most simple action, setting a breakpoint on a memory address can be done
with the 
[bp](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/bp--bu--bm--set-breakpoint-)
(BreakPoint) command:
```
0:000> bp HelloWorld!main
```

Appart from `bp`, the `bu` (Breakpoint unresolved) can be used to set
breakpoints into symbols that weren't resolved yet.

Once set, breakpoints can be listed with
[bl](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/bl--breakpoint-list-)
(Breakpoint List):
```
0:000> bl
     1 e Disable Clear  00007ff6`2caf1920  [C:\Users\user\source\repos\HelloWorld\HelloWorld\HelloWorld.c @ 25]     0001 (0001)  0:**** HelloWorld!main
```

## Derreference pointers

You can derreference pointers with the `poi` function. It works like the **ptr*
operator in C. Here is an example:

```
0:000> dp rsp L1
000000c6`240ff718  00007ff6`2caf2539
0:000> u poi(rsp) L1
HelloWorld!invoke_main+0x39 [D:\a\_work\1\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl @ 79]:
00007ff6`2caf2539 4883c448        add     rsp,48h
```

The first command shows the first value in the stack, which is the return
address. Then, the second command derreferences the return address to read the
instruction. 


## List loaded libraries

The
[lm](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/lm--list-loaded-modules-)
(list modules) command allows to list the loaded libraries. It also indicates if
the library symbols were already loaded.

```
0:000> lm
start             end                 module name
00007ff6`a2cb0000 00007ff6`a2cd6000   HelloWorld C (private pdb symbols)  C:\ProgramData\Dbg\sym\HelloWorld.pdb\B9C97404AB58457C8C397E86E75C7798a\HelloWorld.pdb
00007ffb`87890000 00007ffb`87aaf000   ucrtbased   (deferred)             
00007ffb`97000000 00007ffb`9702b000   VCRUNTIME140D   (deferred)             
00007ffb`c8b20000 00007ffb`c8bb0000   apphelp    (deferred)             
00007ffb`cbba0000 00007ffb`cbe96000   KERNELBASE   (pdb symbols)          C:\ProgramData\Dbg\sym\kernelbase.pdb\DFC28314D88FE97BC24F88F5A77E9F731\kernelbase.pdb
00007ffb`cd210000 00007ffb`cd2cd000   KERNEL32   (pdb symbols)          C:\ProgramData\Dbg\sym\kernel32.pdb\B07C97792B439ABC0DF83499536C7AE51\kernel32.pdb
00007ffb`cdf50000 00007ffb`ce148000   ntdll      (pdb symbols)          C:\ProgramData\Dbg\sym\ntdll.pdb\1669C503FDE3540E0A2FBE91C81204361\ntdll.pdb
```

## Resources

- [WinDbg docs](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/)
- [WinDbg - the Fun Way: Part 1](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-1-2e4978791f9b)
- [Fix Your (Offline) Symbols](https://www.osr.com/nt-insider/2015-issue1/fix-offline-symbols/)
- [Symbols for Windows Debugging](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols)
- [Learn WinDbg](http://www.windbg.xyz/)
