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
which you can with the [.sympath](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-sympath--set-symbol-path-) meta command without arguments:
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
public symbol server (<https://msdl.microsoft.com/download/symbols>). 

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

### Open file where a symbol resides

You can just use the [.open](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-open--open-source-file-) command with the `-a` parameter:
```
.open -a <address>
```

Example:
```
.open -a HelloWorld!main
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
  
## Data

### Registers

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

### Pseudo-registers

- [Pseudo-Register Syntax](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/pseudo-register-syntax)

Pseudo-registers are like variables that WinDbg supports, there are many
predefined and some to allow the user to store its own (integer) values. Every
pseudo-register starts with the symbol `$`. An example of predefined
pseudo-register is `$peb` (that indicates the PEB address) or `$ra` (indicates
the return address). A complete list of predefined/automatic pseudo-registers
can be found in [Automatic Pseudo-Registers](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/pseudo-register-syntax#automatic-pseudo-registers).

```
0:000> ? $ra
Evaluate expression: 140735759137834 = 00007fff`98ee3c2a
```

### The @ symbol

When using a register or pseudo-register you can prepend the `@`
symbol. This indicates the debugger that the next token is a register or
pseudo-register, and avois searching through the whole list of symbols
available.

The following example returns the exact same value, but if you tried to execute
it, you will notice that using `@$ra` will increase the speed of response:
```
0:000> ? $ra
Evaluate expression: 140735759137834 = 00007fff`98ee3c2a
0:000> ? @$ra
Evaluate expression: 140735759137834 = 00007fff`98ee3c2a
```

### Search strings

We can search string patterns with the [s](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/s--search-memory-) command. We will need
to specify the pattern along with a memory range to search the
string.

Here is an example searching an ASCII string:
```
0:000> s -a 775e0000 L200000 "NtQuery"
776f6144  4e 74 51 75 65 72 79 41-74 74 72 69 62 75 74 65  NtQueryAttribute
776f615a  4e 74 51 75 65 72 79 41-75 78 69 6c 69 61 72 79  NtQueryAuxiliary
776f617b  4e 74 51 75 65 72 79 42-6f 6f 74 45 6e 74 72 79  NtQueryBootEntry
776f6191  4e 74 51 75 65 72 79 42-6f 6f 74 4f 70 74 69 6f  NtQueryBootOptio
```

And a example searching an Unicode string:
```
0:000> s -u 775e0000 L200000 "ntdll"
775e8430  006e 0074 0064 006c 006c 002e 0064 006c  n.t.d.l.l...d.l.
7770d2a8  006e 0074 0064 006c 006c 002e 0064 006c  n.t.d.l.l...d.l.
7770d364  006e 0074 0064 006c 006c 002e 0064 006c  n.t.d.l.l...d.l.
```

Here is an example searching all ASCII strings with a minimum length
of 30:
```
0:000> s -[l30]sa 775e0000 L200000
775e5210  "HEAP: Free Heap block %p modifie"
775e5230  "d at %p after it was freed"
775e6418  " !"#$%&'()*+,-./0123456789:;<=>?"
775e6438  "@abcdefghijklmnopqrstuvwxyz[\]^_"
```

Similar example as above but with Unicode strings with a minimum
length of 30:
```
0:000> s -[l30]su 775e0000 L200000
775e1ab4  "㹌睟㹠睟㺄睟㺨睟㻘睟㻸睟㼘睟㼴睟㽐睟㽨睟㾐睟㾰睟㿘睟㿸睟䀘睟䁀睟"
775e1af4  "䁠睟䂀睟䂜睟堘睞䃈睟䃔睟䃤睟䃴睟䄈睟䄠睟䄸睟䅐睟䅤睟䅰睟䆌睟䆸睟"
775e1b34  "䇜睟䇼睟䈌睟䈴睟䉔睟䉤睟䊄睟䊰睟䊼睟䋔睟䋤睟䋼睟䌠睟䌴睟䍌睟䍔睟"
775e1b74  "䍤睟䎄睟䎘睟䎰睟䏈睟"
775e4860  "\Registry\Machine\System\Current"
775e48a0  "ControlSet\Control\MUI\UILanguag"
```

### Display string

We can display an ascii string with [da](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/d--da--db--dc--dd--dd--df--dp--dq--du--dw--dw--dyb--dyd--display-memor) command:
```
0:000> da 0x776f6144
776f6144  "NtQueryAttributesFile"
```

And a Unicode string with [du](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/d--da--db--dc--dd--dd--df--dp--dq--du--dw--dw--dyb--dyd--display-memor) command:
```
0:000> du 775e8430
775e8430  "ntdll.dll"
```

### Derreference pointers

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


### Traversing Windows List Entries

> Based on [WinDbg — the Fun Way: Part 1 - Arrays and Lists](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-1-2e4978791f9b#814d)

Is commom for Windows to use lists to link structures with the type
`LIST_ENTRY`, which is defined in `winnt.h`:

```c
typedef struct _LIST_ENTRY {
    struct _LIST_ENTRY *Flink;
    struct _LIST_ENTRY *Blink;
} LIST_ENTRY,*PLIST_ENTRY,*RESTRICTED_POINTER PRLIST_ENTRY;
```

And we can traverse the lists with the
`Debugger.Utility.Collections.FromListEntry` that can be used with the [dx
command](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/dx--display-visualizer-variables-). Let's
see a practical example.

As example of a list we have the modules loaded by the process, which are linked
by the PEB. Each of this modules is described by a `LDR_DATA_TABLE_ENTRY` which
has the following definition:

```c
typedef struct _LDR_DATA_TABLE_ENTRY {
    PVOID Reserved1[2];
    LIST_ENTRY InMemoryOrderLinks;
    PVOID Reserved2[2];
    PVOID DllBase;
    PVOID Reserved3[2];
    UNICODE_STRING FullDllName;
    BYTE Reserved4[8];
    PVOID Reserved5[3];
    __C89_NAMELESS union {
      ULONG CheckSum;
      PVOID Reserved6;
    };
    ULONG TimeDateStamp;
} LDR_DATA_TABLE_ENTRY,*PLDR_DATA_TABLE_ENTRY;
```

As we can guess, the modules are linked by the `InMemoryOrderLinks` field. This
is important to know since it will allow us to resolve the correct address of
the structure from the list.

So, in order to list the loaded modules, we can use the following command:
```
dx Debugger.Utility.Collections.FromListEntry(((ntdll!_PEB*)@$peb)->Ldr->InLoadOrderModuleList, "ntdll!_LDR_DATA_TABLE_ENTRY", "InLoadOrderLinks")
```

Let's explain it by parts. First, we have the `dx` command that interprets the
[LINQ
expression](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/using-linq-with-the-debugger-objects). Then
the `Debugger.Utility.Collections.FromListEntry` will traverse the list. We
indicate to `FromListEntry` that the list starts in
`((ntdll!_PEB*)@$peb)->Ldr->InLoadOrderModuleList`, being `@$peb` the address of
the Process Environment Block (PEB) and `Ldr->InLoadOrderModuleList` the field
that has the list head.  Then each list entry is casted to
`ntdll!_LDR_DATA_TABLE_ENTRY` type, which has the list links 
in the `InLoadOrderLinks` field (so it is necessary substract the field offset
to obtain a pointer to the beginning of each entry).

Here is the command (with some additions to extract the module name):
```c
0:000> dx Debugger.Utility.Collections.FromListEntry(((ntdll!_PEB*)@$peb)->Ldr->InLoadOrderModuleList, "ntdll!_LDR_DATA_TABLE_ENTRY", "InLoadOrderLinks").Select(e => (e.BaseDllName.Buffer).ToDisplayString("sub"))
Debugger.Utility.Collections.FromListEntry(((ntdll!_PEB*)@$peb)->Ldr->InLoadOrderModuleList, "ntdll!_LDR_DATA_TABLE_ENTRY", "InLoadOrderLinks").Select(e => (e.BaseDllName.Buffer).ToDisplayString("sub"))
    [0x0]            : notepad.exe
    [0x1]            : ntdll.dll
    [0x2]            : KERNEL32.DLL
    [0x3]            : KERNELBASE.dll
    [0x4]            : GDI32.dll
    [0x5]            : win32u.dll
    [0x6]            : gdi32full.dll
    [0x7]            : msvcp_win.dll
    [0x8]            : ucrtbase.dll
    [0x9]            : USER32.dll
    [0xa]            : combase.dll
    [0xb]            : RPCRT4.dll
    [0xc]            : shcore.dll
    [0xd]            : msvcrt.dll
    [0xe]            : COMCTL32.dll
```

And we got the modules. The `Select(e =>
(e.BaseDllName.Buffer).ToDisplayString("sub"))` expression indicates that for
each entry `e` it will take the field `BaseDllName.Buffer` that contains a
unicode string (`wchar_t*`) that we display with `ToDisplayString("sub")`.

### Listing memory maps

We can get a list of memory regions with the [`!address` command](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-address).

Here is an example:
```
0:000> !address

                                     
Mapping file section regions...
Mapping module regions...
Mapping PEB regions...
Mapping TEB and stack regions...
Mapping heap regions...
Mapping page heap regions...
Mapping other regions...
Mapping stack trace database regions...
Mapping activation context regions...

        BaseAddress      EndAddress+1        RegionSize     Type       State                 Protect             Usage
--------------------------------------------------------------------------------------------------------------------------
+        0`00000000        0`7ffe0000        0`7ffe0000             MEM_FREE    PAGE_NOACCESS                      Free       
+        0`7ffe0000        0`7ffe1000        0`00001000 MEM_PRIVATE MEM_COMMIT  PAGE_READONLY                      Other      [User Shared Data]
+        0`7ffe1000        0`7ffec000        0`0000b000             MEM_FREE    PAGE_NOACCESS                      Free       
+        0`7ffec000        0`7ffed000        0`00001000 MEM_PRIVATE MEM_COMMIT  PAGE_READONLY                      <unknown>  [..........Q82i..]
+        0`7ffed000       6c`6d200000       6b`ed213000             MEM_FREE    PAGE_NOACCESS                      Free       
+       6c`6d200000       6c`6d3b8000        0`001b8000 MEM_PRIVATE MEM_RESERVE                                    <unknown>  
        6c`6d3b8000       6c`6d3b9000        0`00001000 MEM_PRIVATE MEM_COMMIT  PAGE_READWRITE                     PEB        [1f04]
        6c`6d3b9000       6c`6d3bd000        0`00004000 MEM_PRIVATE MEM_COMMIT  PAGE_READWRITE                     TEB        [~0; 1f04.1efc]
...stripped...
+     7ff9`30860000     7ff9`30861000        0`00001000 MEM_IMAGE   MEM_COMMIT  PAGE_READONLY                      Image      [KERNELBASE; "C:\Windows\System32\KERNELBASE.dll"]
      7ff9`30861000     7ff9`3099b000        0`0013a000 MEM_IMAGE   MEM_COMMIT  PAGE_EXECUTE_READ                  Image      [KERNELBASE; "C:\Windows\System32\KERNELBASE.dll"]
      7ff9`3099b000     7ff9`30b1e000        0`00183000 MEM_IMAGE   MEM_COMMIT  PAGE_READONLY                      Image      [KERNELBASE; "C:\Windows\System32\KERNELBASE.dll"]
      7ff9`30b1e000     7ff9`30b23000        0`00005000 MEM_IMAGE   MEM_COMMIT  PAGE_READWRITE                     Image      [KERNELBASE; "C:\Windows\System32\KERNELBASE.dll"]
      7ff9`30b23000     7ff9`30b5f000        0`0003c000 MEM_IMAGE   MEM_COMMIT  PAGE_READONLY                      Image      [KERNELBASE; "C:\Windows\System32\KERNELBASE.dll"]
...stripped...
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

### Show calls of a function

Sometimes in order to understand relation between functions, can be useful to
only display the calls instructions instead of all of the disassemble. You can
do this with the `/c` option of the `uf` command. Here is an example of 
`LoadLibraryW` calls:

```
0:000> uf /c KERNELBASE!LoadLibraryW
KERNELBASE!LoadLibraryW (76281930)
  KERNELBASE!LoadLibraryW+0xc (7628193c):
    call to KERNELBASE!LoadLibraryExW (762519a0)
```

From this command you can deduce that `LoadLibraryW` is just a wrapper around
`LoadLibraryExW`.

## Execution

### Debugger mode

WinDbg debugger has two modes: [Source Mode](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-in-source-mode) and [Assembly Mode](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-in-assembly-mode). 

The debugger behaves differently based on the mode it is working on. You should
check the WinDbg docs for all the differences, but for example, in source mode,
commands like [t](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/t--trace-) (Step In) or [p](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/p--step-) (Step Over) will execute a line of
code, whereas in assembly mode those same commands will execute just an assembly
instruction. It is important to be aware of the mode you are in order to
identify how the debugger will behave.


You can change the mode with the [l](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/l---l---set-source-options-) command. To set the Assembly Mode you
can use `l-t`:
```
0:000> l-t
Source options are 0:
    None
```

And then to set Source Mode you can use `l+t`:
```
0:000> l+t
Source options are 1:
     1/t - Step/trace by source line
```

### Execution controls

There are several commands that allows to control the execution of the debugged
process.

- [g](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/g--go-) (Go) : Continues until an event makes it stop, like a breakpoint, an
  exception, or the end of the program.
- [gu](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/gu--go-up-) (Go up/Step Out) : Continues until the current function returns.

- [p](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/p--step-) (Ste**p**/Step Over): Executes a single instruction or line of code (depends
  on the debugger mode), without entering functions.

- [t](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/t--trace-) (Trace/Step In): Executes a single instruction or line of cod (depends
  on the debugger mode) and enters into a function when called.

### Breakpoints

The commands started by *b* are related to breakpoints.

The most simple action, setting a breakpoint on a memory address can be done
with the [bp](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/bp--bu--bm--set-breakpoint-) (BreakPoint) command:
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

### Time Travel Debugging

Time Travel Debugging or TTD allows us to record a program execution
and then inspect it in the debugger, going forward and backwards in
the execution flow.

- [Time Travel Debugging: Getting started with TTD](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/time-travel-debugging-overview#getting-started-with-ttd)

## Modules

A module is a library or the main exe of a process.

You can list the loaded modules with the
[lm](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/lm--list-loaded-modules-)
(list modules) command. It also indicates if the library symbols were already loaded.

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

Here you can see that the `HelloWorld` program loads the modules `ucrtbased`,
`VCRUNTIME140D`, etc. Note that some modules/libraries can be loaded lazily when
the process is running (by using `LoadLibrary` or similar), so may not all the
modules that the process are listed when the process starts.

### Discover to which module a symbol belongs

If you want to know to which module an specific symbol belongs, a trick you can
use is to invoke the
[!lmi](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-lmi)
command. Here is an example with the `LoadLibraryA` function:

```
0:000> !lmi LoadLibraryA
Loaded Module Info: [loadlibrarya] 
         Module: KERNELBASE
   Base Address: 76140000
....truncated....
```
You can see that the `LoadLibraryA` function belongs to the `KERNELBASE` module.

### List all symbols of a module

We can use the [`x` command](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/x--examine-symbols-) in combination with the help of a wildcard to
list all the symbols that a module contains.

For example, to list all available symbols in `ntdll`:
```
x ntdll!*
```

```
0:000> x ntdll!*
00007ffa`9f9b60d0 ntdll!RtlpHpVaMgrRangeCreate (void)
00007ffa`9fa09dd0 ntdll!RtlFindLastBackwardRunClear (void)
00007ffa`9f9f52b0 ntdll!EtwDeliverDataBlock (void)
.....strip.....
```

## Scripting


### Command Script files

The most basic way of scripting in WinDbg is to run files with gdb
commands. This can be accomplished with the [`$<`, `$><`, `$$<`, `$$><`,
`$$>a<`](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-----------------------a---run-script-file-)
command family, make sure to check the docs to understand the differences
between each command.

Imagine a file named hello.txt with the following contents:
```
.echo Hello World
.echo Bye World
```

You can execute its commands with:
```
$$<"C:\hello.txt"
```

In this case each command will be executed separately:
```
0:000> $$<"C:\hello.txt"
0:000> .echo Hello World
Hello World
0:000> .echo Bye World
Bye World
```

If you want to execute all you can use `$$><:
```
0:000> $$><"C:\hello.txt"
Hello World
Bye World
```
 
Some samples of Windbg file scripts:
- [WinDbg_Scripts](https://github.com/yardenshafir/WinDbg_Scripts) by yardenshafir
 
### Script Providers

Additionally to the common script files, WinDbg allows to register script
providers that allow to create WinDbg in other languages, which is usually more
powerful. 

We can list the available script providers with the [.scriptproviders](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/-scriptproviders--list-script-providers-)
command:
```
0:000> .scriptproviders
Available Script Providers:
    NatVis (extension '.NatVis')
    JavaScript (extension '.js')
```

### Javascript Scripting
One of the most used script engines is Javascript, that comes by default in
the latest WinDbg versions.

- [JavaScript Debugger Scripting](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/javascript-debugger-scripting)
- 2017/12/01 [Debugger data model, Javascript & x64 exception handling](https://doar-e.github.io/blog/2017/12/01/debugger-data-model/) by
  Axel "0vercl0k" Souchet
- [WinDbg Javascript reference](https://github.com/hugsy/defcon_27_windbg_workshop/blob/main/windbg_cheatsheet.md#windbg-javascript-reference)

Some samples of Windbg javascript scripts:
- [windbg-scripts](https://github.com/0vercl0k/windbg-scripts) by 0vercl0k
- [WinDbg-Samples](https://github.com/microsoft/WinDbg-Samples) by Microsoft
- [windbg_js_scripts](https://github.com/hugsy/windbg_js_scripts) by hugsy
- [WinDbg_Scripts](https://github.com/yardenshafir/WinDbg_Scripts) by yardenshafir

## Resources

- [WinDbg docs](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/)
- [@windbgtips](https://nitter.net/windbgtips)
- 2020/05/21 [WinDbg - the Fun Way: Part 1](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-1-2e4978791f9b) by Yarden Shafir
- 2020/05/21 [WinDbg — the Fun Way: Part 2](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-2-7a904cba5435) by Yarden Shafir
- 2015 [Fix Your (Offline) Symbols](https://www.osr.com/nt-insider/2015-issue1/fix-offline-symbols/) by David Boyce
- [Symbols for Windows Debugging](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols)
- [Learn WinDbg](http://www.windbg.xyz/)
- 2019 ["Modern Debugging with WinDbg Preview" DEFCON 27 workshop](https://github.com/hugsy/defcon_27_windbg_workshop) by hugsy
  & Overcl0k
- [WinDBG quick start tutorial](https://codemachine.com/articles/windbg_quickstart.html) 
- [WinDBG expression evaluation tutorial: An in-depth tutorial on the MASM and C++ expression evaluators in WinDBG.](https://codemachine.com/articles/windbg_expressions.html)
- [Dbg3011_Advanced_WinDbg](https://gitlab.com/opensecuritytraining/dbg3011_advanced_windbg) by Cedric Halbronn
