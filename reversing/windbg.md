# WinDbg

## Installation

You can [install WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)
from the [Microsoft Store](https://apps.microsoft.com/detail/9pgjgd53tn86). 


## Commands

The [official documentation for WinDbg
commands](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/debugger-commands).

### lm: List Modules

The `lm` command allows to list the loaded libraries.

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

## References

- [WinDbg - the Fun Way: Part 1](https://medium.com/@yardenshafir2/windbg-the-fun-way-part-1-2e4978791f9b)
