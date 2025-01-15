# WinDbg

## Installation

You can [install WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)
from the [Microsoft Store](https://apps.microsoft.com/detail/9pgjgd53tn86). 

## Commands

The [official documentation for WinDbg
commands](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/debugger-commands).

### lm: List Modules

The [lm](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/lm--list-loaded-modules-) command allows to list the loaded libraries. It also indicates if the
library symbols were already loaded.

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

## u: Unassemble

The
[u](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/u--unassemble-)
command disassembles the instructions in a given memory position.

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

## uf: Unassemble Function

The
[uf](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/uf--unassemble-function-)
command the assembly of a function, but it follows the jumps in order to display
the function assembly so it can be messy at first glance.

This example of the `NtAllocateVirtualMemory` function is pretty straightforward:
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

However, when dissasembly `NtQuerySystemTime` with `uf`, it shows parts of another
function `RtlQuerySystemTime` because `NtQuerySystemTime` just jumps into it:
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

Sometimes you may prefer using [u](#u-unassemble) to display raw instructions
without following the jumps.


## References

- [WinDbg docs](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/)
- [WinDbg - the Fun Way: Part 1](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-1-2e4978791f9b)
