# GDB / Pwndbg / GEF

Here are described different aspects of gdb and how to perform certain
actions. Additionally, commands of [pwndbg](https://github.com/pwndbg/pwndbg) and [gef](https://github.com/hugsy/gef) are also presented.

The commands of gdb will be start by the `(gdb)` prompt. For example:
```
(gdb) info registers
```
The gdb commands can be always used inside gdb, even if we are using gef or
pwndbg.

The gef commands will start by the `gef➤` prompt. These commands will require
the use of [gef](https://github.com/hugsy/gef). For example:
```
gef➤ pattern create
```

The pwndbg commands will start by the `pwndbg>` prompt. These commands will
require the use of [pwndbg](https://github.com/pwndbg/pwndbg). For example:
```
pwndbg> stack
```

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

### Launch gdbserver

We can a launch a gdbserver in remote (or even local) machine with
`gdbserver <host>:<port> <binary-file>`. For example:
```
$ gdbserver localhost:2000 /bin/ls
```

Or to attach to process ``gdbserver <host>:<port> --attach <pid>`:
```
$ gdbserver localhost:2000 --attach 23958
```

And then we connect to gdbserver with the `target` command:
```
$ gdb -q
(gdb) target remote localhost:2000
```

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

### Attach to child process when fork

When we are debugging a program that forks (like for example some kind of
network service) and we want to debug the child process, we can indicate to gdb
to automatically detach from parent and attach to child with:
```
(gdb) set follow-fork-mode child
```

To verify it we can check the value of `follow-fork-mode`:
```
(gdb) show follow-fork-mode
Debugger response to a program call of fork or vfork is "child".
```

Once the program fork, we should see the following message:
```
(gdb) c
[Attaching after process 1 fork to child process 18]
[New inferior 2 (process 18)]
[Detaching after fork from parent process 1]
[Inferior 1 (process 1) detached]
```


### Show current architecture

We can see what is the current target architecture with `show architecture` command:
```
(gdb) show architecture
The target architecture is set to "auto" (currently "i386:x86-64").
```

### Change current architecture

We can use the `set architecture` command to change the architecture that gdb is
using to inspect the target:

```
(gdb) set architecture armv5t
The target architecture is set to "armv5t".
```

### Using De Bruijn cyclic pattern

We can generate a [De Bruijn cyclic pattern](https://en.wikipedia.org/wiki/De_Bruijn_sequence) with the gef command [pattern create](https://hugsy.github.io/gef/commands/pattern/):
```
gef➤  pattern create --period 4 60
[+] Generating a pattern of 60 bytes (n=4)
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaa
```

And then we can search the offset of a substring in the pattern with [pattern
search](https://hugsy.github.io/gef/commands/pattern/#pattern-search). We can provide a number or a string to search:
```
gef➤  pattern search --period 4 0x6161616d
[+] Searching for '0x6161616d'
[+] Found at offset 48 (little-endian search) likely
[+] Found at offset 45 (big-endian search)
```

```
gef➤  pattern search --period 4 "aaba"
[+] Searching for 'aaba'
[+] Found at offset 3 (little-endian search) likely
[+] Found at offset 2 (big-endian search)
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

### Print variables while executing

We can insert calls to `printf` function with `dprintf` (Dynamic Printf). Here
is an example of inserting a dynamic printf to print the value of a variable
inside a loop:

```
(gdb) list main
1	#include <stdio.h>
2
3	int main() {
4	  int i = 0;
5	  int sum = 0;
6
7	  for(i = 0; i < 10; i++) {
8	    sum += i;
9	  }
10
(gdb) dprintf main-loop.c:8,"Sum: %d\n",sum
Dprintf 1 at 0x1158: file main-loop.c, line 8.
(gdb) r
...
Sum: 0
Sum: 0
Sum: 1
Sum: 3
Sum: 6
Sum: 10
Sum: 15
Sum: 21
Sum: 28
Sum: 36
Total: 10
[Inferior 1 (process 5350) exited normally]
```

In order to print the variables, `dprintf` inserts a breakpoint, so we can use
the `condition` command to add a condition to only print under defined
circunstances. For example, the next lines makes that variable `sum` is only
printed when its value is odd:

```
(gdb) condition 1 sum % 2 == 1
(gdb) r
Starting program: /home/ada/projects/c-test/main-loop
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Sum: 1
Sum: 3
Sum: 15
Sum: 21
Total: 10
[Inferior 1 (process 3216) exited normally]
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

In case of using GEF, we can use the `dereference` command to print the stack
with the references:
```
gef➤  dereference 5
0xfffb4230│+0x0000: 0x00000000	 ← $sp
0xfffb4234│+0x0004: 0x00000000
0xfffb4238│+0x0008: 0x00010538  →  <_start+0> mov r11,  #0
0xfffb423c│+0x000c: 0x00000000
0xfffb4240│+0x0010: 0x00000000
```

### Show the data type

If we have debugging symbols available, we can see the type of a variable with the `ptype` command:

```
(gdb) ptype dog
type = struct animal_t {
    char *name;
    int species;
    char *sound;
}
```

And we can even show the offsets of type:
```
(gdb) ptype /o dog
type = struct animal_t {
/*    0      |     8 */    char *name;
/*    8      |     4 */    int species;
/* XXX  4-byte hole  */
/*   16      |     8 */    char *sound;
}
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

### Write memory

We can use the set command to write values in specific memory regions, but we
need to provide a type.

In order to write an string, we need to write it as an array of characters. For example:
```
set {char[10]}0xfffb4d0a = "Foobar"
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

We can also use `print` command to check a register value:
```
(gdb) p $rax
$3 = 0x5555556b12b0
```

Resources:
- [GDB: Registers](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Registers.html#Registers)

#### Modify registers

We can use the [set](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Assignment.html) command to modify the value of a register:
```
(gdb) set $rax = 7
```

Curiously, we can also use the `print` command for the same purpose, with the
only difference that the value will be printed after being set:
```
(gdb) print $rax = 7
$6 = 0x7
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

##### Set breakpoint at code line

With debugging symbols, we can also set breakpoints in lines inside the source
code files:

```
(gdb) b main.c:42
Breakpoint 1 at 0x5555555755a1: file main.c, line 42.
```

##### Set conditional breakpoint

We can set a conditional breakpoint with an `if` clause to specify a condition:
```
(gdb) break main-loop.c:14 if (sum >= 10)
Breakpoint 1 at 0x1191: file main-loop.c, line 14.
```

```
(gdb) break *main+72 if $eax > 5
Breakpoint 3 at 0x555555555191: file main-loop.c, line 14.
```


#### List breakpoints

We can use the command `info breakpoints` or `info break` or `i b` to list our
breakpoints:
```
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x804875b <main+5>
```

#### Add condition to breakpoint

We can use the [condition](https://www.sourceware.org/gdb/current/onlinedocs/gdb.html/Conditions.html) command to add a condition to an already set
breakpoint. We need to specify the breakpoint number along with the condition
:
```
(gdb) condition 1 sum > 4
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

### Signals

This section explains how to manage [signals](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Signals.html) with gdb.

#### Check how a signal is handled

We can check how a signal is handled by using the `handle` command:
```
(gdb) handle SIGPIPE
Signal        Stop	Print	Pass to program	Description
SIGPIPE       Yes	Yes	Yes		Broken pipe
```

Additionally, we can use the `info signals` or `info handle` commands to show
how every signal is handled:
```
(gdb) info signals
Signal        Stop	Print	Pass to program	Description

SIGHUP        Yes	Yes	Yes		Hangup
SIGINT        Yes	Yes	No		Interrupt
SIGQUIT       Yes	Yes	Yes		Quit
...
```

#### Change how a signal is handled

We can indicate to gdb how to handle a signal by using the `handle` command:
```
(gdb) handle SIGPIPE nostop print pass
Signal        Stop	Print	Pass to program	Description
SIGPIPE       No	Yes	Yes		Broken pipe
```

There are three sets of options:
- `stop` | `nostop`: Indicates if gdb must retake control when signal is
  received.
- `print` | `noprint`: Indicates if gdb must indicate when the signal is
  received.
- `pass` | `nopass`: Indicates if gdb must pass the signal to the debugee.

For more information check `help handle`.


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

#### Step in

We can execute only one source code line, entering in a function if that is the
case, with `step` or `s`:
```
(gdb) step
```

If we want to only execute one assembly instruction then we can use `stepi` or
`si`:
```
(gdb) stepi
```

#### Step over

We can execute only one source code line, avoiding entering into functions,
with `next`:
```
(gdb) next
```

If we want to only execute one assembly instruction then we can use `nexti` or
`ni`:
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

## Code

### Show the source code

Warning: For this to work we need the binary with symbols and the source code.

We can use the `list` command to print source code in an specific region:

```
(gdb) list main
1	#include <stdio.h>
2	#include <stdlib.h>
3
4	int main(int argc, char** argv) {
5	  int i = 0;
6	  int sum = 0;
7	  int start = 0;
8
9	  if (argc > 1) {
10	    start = atoi(argv[1]);
```

However, as we can appreciate, just a tiny portion is show. By default, `list`
will show the code around the line we indicate, in this case the code around the
line that starts main function.

We can extend the portion by indicating the starting and ending line separated
by a comma:

```
(gdb) list main,20
4	int main(int argc, char** argv) {
5	  int i = 0;
6	  int sum = 0;
7	  int start = 0;
8
9	  if (argc > 1) {
10	    start = atoi(argv[1]);
11	  }
12
13	  for(i = start; i < 10; i++) {
14	    sum += i;
15	  }
16
17	  printf("Total: %d\n", i);
18	}
```

This is not a perfect approach to show completely a function, I know, not sure
if there is a way to being able to show all the source code of a function in a
simple way, but anyway we can read the code with our favorite text editor.

Additionally, we can use the **TUI** mode with `Ctrl+x a` to see the code. Once in the
TUI mode the code should be displayed, or we can activate it with `layout src` command.

### Show the assembler code of a function

We can use the `disassemble` command of gdb to show the assembly code of a function:

```
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000001149 <+0>:	push   rbp
   0x000000000000114a <+1>:	mov    rbp,rsp
   0x000000000000114d <+4>:	sub    rsp,0x20
   0x0000000000001151 <+8>:	mov    DWORD PTR [rbp-0x14],edi
   0x0000000000001154 <+11>:	mov    QWORD PTR [rbp-0x20],rsi
   ................................
   0x00000000000011a6 <+93>:	lea    rax,[rip+0xe57]        # 0x2004
   0x00000000000011ad <+100>:	mov    rdi,rax
   0x00000000000011b0 <+103>:	mov    eax,0x0
   0x00000000000011b5 <+108>:	call   0x1030 <printf@plt>
   0x00000000000011ba <+113>:	mov    eax,0x0
   0x00000000000011bf <+118>:	leave
   0x00000000000011c0 <+119>:	ret
End of assembler dump.
```

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
