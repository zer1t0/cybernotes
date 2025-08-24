# GDB / Pwndbg

Here are described different aspects of gdb and how to perform certain
actions. Additionally, commands of [pwndbg](https://github.com/pwndbg/pwndbg) are also presented.

## Launch gdb

To debug/examine a binary:
```
gdb <binary-file>
```

To attach to a living process:
```
gdb <binary-file> <pid>
```

We need to indicate the binary file that the process uses to get access to the
symbols.

To examine a core dump:
```
gdb <binary-file> <core-file>
```

For a core dump we need to indicate the binary used by the process that creates
the core dump, in order to access to the symbols.

## Help

We can get the help for an specific command by using `help <command>`:
```
(gdb) help x
Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
  t(binary), f(float), a(address), i(instruction), c(char), s(string)
  and z(hex, zero padded on the left).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
The specified number of objects of the specified size are printed
according to the format.  If a negative number is specified, memory is
examined backward from the address.

Defaults for format and size letters are those previously used.
Default count is 1.  Default address is following last thing printed
with this command or "print".
```

## Misc

### Use Intel syntax

We can change the assembly syntax to intel with:
```
(gdb) set disassembly-flavor intel
```

If we want this as the default disassembly syntax, we can add this command to
our gdbinit file.

### Disable pagination

We executing some commands that produce a lot of lines of output, gdb can give
us just the first lines of output and ask for confirmation for continue with the
following prompt:
```
--Type <RET> for more, q to quit, c to continue without paging--
```

We can disable this prompt with the `set pagination off` command:
```
(gdb) set pagination off
```

From now on, gdb will display the full output without asking. If we want this as
the default behavior, we can add this command to our gdbinit file.

### Enable ASLR

When debugging a child program, gdb disables ASLR by default. We can prevent
this behavior by using the command `set disable-randomization off`:

```
(gdb) set disable-randomization off
```

### Execute many commands in command line

We can start gdb by executing commands with the `-ex` option, and we can specify
many to execute several commands in sequence. Here is an example:
```
gdb -q program -ex "set pagination off" -ex "disassemble main" -ex "quit"
```

This is has the same effect as:
```
$ gdb -q program
Reading symbols from program...
(gdb) set pagination off
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000118d <+0>:	lea    ecx,[esp+0x4]
   0x00001191 <+4>:	and    esp,0xfffffff0
   0x00001194 <+7>:	push   DWORD PTR [ecx-0x4]
....
End of assembler dump.
(gdb) quit
```

## Symbols

### Get symbol address

We can use the `info address <symbol>` to get the symbol address:
```
(gdb) info address main
Symbol "main" is at 0x8048756 in a file compiled without debugging.
```

### List symbols

We can list the symbols with the `info functions` command:
```
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  puts@plt
0x0000000000001040  printf@plt
0x0000000000001050  atoi@plt
0x0000000000001060  __cxa_finalize@plt
0x0000000000001070  _start
0x00000000000010a0  deregister_tm_clones
0x00000000000010d0  register_tm_clones
0x0000000000001110  __do_global_dtors_aux
0x0000000000001150  frame_dummy
0x0000000000001159  add
0x000000000000116d  main
0x00000000000011fc  _fini
```

In many cases listing all the symbols can be overkill, but we can filter and
just look for those that match a regular expression:
```
(gdb) info functions print
All functions matching regular expression "print":

Non-debugging symbols:
0x0000000000001040  printf@plt
```


## Data

### Print string

We can print an string with the `x/s` or `p` commands from a reference to the
string:
```
(gdb) x/s 0x08046820
0x08046820:	"/bin/bash"
```

```
(gdb) p (char*)0x00010876
$14 = 0x10876 "Hello world\n"
```

Be aware that it is common to find a register or memory address that
points to a string address, not the string directly, so you may need indicate
that is a `char **` type like this:

```
(gdb) x/s *(char**)$esp
0x56557008:	"Hello %s\n"
```

```
(gdb) p *(char**)$esp
$1 = 0x56557008 "Hello %s\n"
```

### Print array

In case we have a variable or register that points to a dynamically allocated
array, we can print it with something like `print *<var/reg>@<count>`. Here is
an example:

```
(gdb) print *$rax@4
$13 = {0, 4, 8, 9}
```

Another possibility is to cast the variable to a fixed length array type:
```
print (int[4])*$rax
$16 = {0, 4, 8, 9}
```

### Print stack

We can list the frames of the stack with the [backtrace](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Backtrace.html#Backtrace) command:
```
(gdb) bt
#0  0x000055555555515d in add ()
#1  0x00005555555551d3 in main ()
```

Or we can know the values in the stack by reading the memory pointed by the
stack pointer with the help of the [x](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Memory.html#Memory) command:
```
(gdb) x/8xg $rsp
0x7fffffffe030:	0x00007fffffffe060	0x00005555555551d3
0x7fffffffe040:	0x00007fffffffe178	0x0000000300000000
0x7fffffffe050:	0x0000000000000000	0x0000000100000003
0x7fffffffe060:	0x0000000000000003	0x00007ffff7def24a
```

In this case, the `8xg` modifiers indicate to show eigth giant words (8 bytes
each) in hexadecimal format.

Additionally, for a more complete output, we can use the [stack](https://pwndbg.re/pwndbg/latest/commands/stack/stack/) command of pwndbg:
```
pwndbg> stack
00:0000│ rbp rsp 0x7fffffffe020 —▸ 0x7fffffffe050 ◂— 3
01:0008│+008     0x7fffffffe028 —▸ 0x5555555551d3 (main+102) ◂— mov dword ptr [rip + 0x2e53], eax
02:0010│+010     0x7fffffffe030 —▸ 0x7fffffffe168 —▸ 0x7fffffffe44e ◂— '/home/ada/add'
03:0018│+018     0x7fffffffe038 ◂— 0x300000000
04:0020│+020     0x7fffffffe040 ◂— 0
05:0028│+028     0x7fffffffe048 ◂— 0x100000003
06:0030│+030     0x7fffffffe050 ◂— 3
07:0038│+038     0x7fffffffe058 —▸ 0x7ffff7def24a (__libc_start_call_main+122) ◂— mov edi, eax
```

### Show process command line

We can retrieve the exact command line of the process with `info proc cmdline`:

```
(gdb) info proc cmdline
process 4730
cmdline = '/home/ada/hello Ada'
```

### List memory maps

We can [list the memory maps of the process](https://stackoverflow.com/a/5691536) with the `info proc mappings` command:

```
(gdb) info proc mappings
process 4730
Mapped address spaces:

	Start Addr   End Addr       Size     Offset  Perms   objfile
	0x56555000 0x56556000     0x1000        0x0  r--p   /home/ada/hello
	0x56556000 0x56557000     0x1000     0x1000  r-xp   /home/ada/hello
	0x56557000 0x56558000     0x1000     0x2000  r--p   /home/ada/hello
	0x56558000 0x56559000     0x1000     0x2000  r--p   /home/ada/hello
	0x56559000 0x5655a000     0x1000     0x3000  rw-p   /home/ada/hello
	0xf7d7e000 0xf7da0000    0x22000        0x0  r--p   /usr/lib/i386-linux-gnu/libc.so.6
	0xf7da0000 0xf7f19000   0x179000    0x22000  r-xp   /usr/lib/i386-linux-gnu/libc.so.6
	0xf7f19000 0xf7f99000    0x80000   0x19b000  r--p   /usr/lib/i386-linux-gnu/libc.so.6
	0xf7f99000 0xf7f9b000     0x2000   0x21b000  r--p   /usr/lib/i386-linux-gnu/libc.so.6
	0xf7f9b000 0xf7f9c000     0x1000   0x21d000  rw-p   /usr/lib/i386-linux-gnu/libc.so.6
	0xf7f9c000 0xf7fa6000     0xa000        0x0  rw-p
	0xf7fc1000 0xf7fc3000     0x2000        0x0  rw-p
	0xf7fc3000 0xf7fc7000     0x4000        0x0  r--p   [vvar]
	0xf7fc7000 0xf7fc9000     0x2000        0x0  r-xp   [vdso]
	0xf7fc9000 0xf7fca000     0x1000        0x0  r--p   /usr/lib/i386-linux-gnu/ld-linux.so.2
	0xf7fca000 0xf7fed000    0x23000     0x1000  r-xp   /usr/lib/i386-linux-gnu/ld-linux.so.2
	0xf7fed000 0xf7ffb000     0xe000    0x24000  r--p   /usr/lib/i386-linux-gnu/ld-linux.so.2
	0xf7ffb000 0xf7ffd000     0x2000    0x31000  r--p   /usr/lib/i386-linux-gnu/ld-linux.so.2
	0xf7ffd000 0xf7ffe000     0x1000    0x33000  rw-p   /usr/lib/i386-linux-gnu/ld-linux.so.2
	0xfffdd000 0xffffe000    0x21000        0x0  rw-p   [stack]
```

### Registers

#### Show registers

We can use the `info registers` command to display the registers values:
```
(gdb) info registers
eax            0x8048670	134514288
ecx            0x45b	1115
edx            0x0	0
ebx            0x804a000	134520832
esp            0xbffffa70	0xbffffa70
ebp            0xbffffa88	0xbffffa88
esi            0x45b	1115
edi            0x0	0
eip            0x8048549	0x8048549 <shell+51>
eflags         0x296	[ PF AF SF IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
```

Additionally, if we are in the TUI mode, we can display the registers in its own
window with the `layout regs` command.

In case we just want to get the value of one register we can pass it to `info register`:
```
(gdb) info registers eax
eax            0x8048670	134514288
```



## Execution
### Breakpoints

Breakpoints are interruptions placed by the debugger into the debuggee to
retrieve control to the debugger when a certain location of the program is
reached. It is also possible to just trigger a breakpoint when a certain
condition is met.

#### Set breakpoints

In order to set a breakpoint the command [break](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_node/gdb_28.html) can be used, but also any of
its shorter forms like `b` or `br`.

##### Set breakpoint at function

To set a breakpoint into a function we can just use its name:
```
(gdb) b main
Breakpoint 1 at 0x804875b
```

##### Set breakpoint at address

In order to set a breakpoint in a specific address we must include the symbol
`*` as a prefix:
```
(gdb) b *0x804875b
Breakpoint 1 at 0x804875b
```

##### Set breakpoint at function offset

We can [set a breakpoint at a function offset](https://stackoverflow.com/a/52268979) by doing a trick and treat the
function name as an address in the program:
```
(gdb) b *(main+65)
Breakpoint 2 at 0x8048567
```

This will set a breakpoint into the <u>assembly instruction</u> whose offset is
65 inside the `main` function.

#### List breakpoints

We can use the command `info breakpoints` or `info break` or `i b` to list our
breakpoints:
```
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x804875b <main+5>
```

#### Remove breakpoint

We can remove the breakpoints by ID with the `delete breakpoints` command:
```
(gdb) delete breakpoints 1
```

Or also by using the `clear` command and specifying the area where the
breakpoint is:
```
(gdb) clear *0x804875b
Deleted breakpoint 1
```

Moreover, if we want to delete all breakpoints, we can just use
`delete breakpoints` without any breakpoint as argument:

```
(gdb) delete breakpoints
Delete all breakpoints? (y or n) y
```

### Watchpoints

Watchpoints, also called data breakpoints, are used to interrupt the program
when the value of a certain variable or register is changed.

#### Set a watchpoint

```
(gdb) watch $eax
Watchpoint 13: $eax
```

#### List watchpoints

```
(gdb) info watchpoints
Num     Type           Disp Enb Address    What
13      watchpoint     keep y              $eax
```

### Execution controls
#### Start program

We can start the debugee program execution with the command `run` (that can be
abbreviated as `r` too):
```
(gdb) r
```

If we want pass arguments to the program, we can pass them in the `run` command:
```
(gdb) run -l -a
```

Moreover, in case we rerun the program again, those arguments passed the first
time are automatically passed again.

#### Continue program

We can instruct a program to continue its normal execution until the end or a
breakpoint by using the `continue` (or `cont` or `c`) command:
```
(gdb) c
```

#### Stepping

We can execute only one source code line by using the `next` or the `step`
commands. The difference between them is that if the next line is a function
call, then `step` will enter the function whereas `next` will skip the function
and go to the next line in the current function.

To go to the next source code line, and enter a function if it's called, we can
use `step` or `s`:
```
(gdb) step
```

Or we can use `next` or `n` to go to the next source line without enter into any
called function:
```
(gdb) next
```

Additionally, if we want to step over assembly instructions instead of source
code lines, we can do it with `stepi` and `nexti` commands, that are analogous
to `step` and `next`:

We can use `stepi` or `si` to execute one assembly instruction and enter into
function calls:
```
(gdb) stepi
```

Or we can use `nexti` or `ni` to execute one assembly instruction without
entering into function calls:
```
(gdb) nexti
```

#### Execute until the end of function

We can use the command `finish` or `fin` to continue execution until the current
function finishes (or a breakpoint is triggered):

```
(gdb) finish
```


#### Execute out of loop

To execute until we are out of a loop, we can use the `until` command, that will
execute until an instruction with a higher memory address is reached



## Scripting

### gdbinit files

[gdbinit](https://www.man7.org/linux/man-pages/man5/gdbinit.5.html) files allows us to indicate gdb commands that we would like to
execute when starting a gdb session.

The most common one is `~/.gdbinit` (gdbinit in user directory) that is used to
specify common options for all the gdb sessions like setting assembly syntax to
intel:
```
$ echo "set disassembly-flavor intel" >> ~/.gdbinit
```

We shouldn't put file specific commands in `~/.gdbinit` as this is loaded before
the file we are debugging and it can give problems. For example, if we try to
set a breakpoint in `~/.gdbinit`, this won't work as the file is not loaded yet.

Alternatively, to use project specific gdbinit files, we have a few options like
using the `--command/-x` parameter:
```
$ gdb --command project.gdbinit ./debugge
```

Or we can set the project options into `./.gdbinit`, however as a security
measure, loading gdbinit for current directory is disabled by default so we need
to enable it by specifying `add-auto-load-safe-path .` in our `~/.gdbinit` file.


## TUI

[TUI](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_chapter/gdb_19.html) (Terminal User Interface) is a gdb mode that offers a graphical interface in
the terminal. You can activate TUI mode in gdb by specifying `-tui` in the gdb
command line or pressing `Ctrl-x a` in a gdb session.


## Resources

- 2008: [Art of Debugging](https://nostarch.com/debugging.htm) by Norman Matloff and Peter Jay Salzman
- 09/08/2016: [gdb Debugging Full Example (Tutorial): ncurses](https://www.brendangregg.com/blog/2016-08-09/gdb-example-ncurses.html) by Brendan Gregg

- [pwndbg (tool)](https://github.com/pwndbg/pwndbg) by disconnect3d, zachriggle, gsingh93, patryk4815 and [others](https://github.com/pwndbg/pwndbg/graphs/contributors)
- [peda (tool)](https://github.com/longld/peda) by longld and [others](https://github.com/longld/peda/graphs/contributors)
- [gef (tool)](https://github.com/hugsy/gef) by Grazfather, hugsy and [others](https://github.com/hugsy/gef/graphs/contributors)
