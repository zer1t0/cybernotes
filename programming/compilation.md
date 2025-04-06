# Compilation

Compilation consists of [four stages](https://teachcomputerscience.com/the-four-stages-of-compilation/): Preprocessing, Compiling, Assemble and
Linking.

The following diagram indicates the input and output of the compilation phases,
including the common files extensions, as well as the names of the phases along
with the programs used in each of them along with the `gcc` command.
Additionally, in order to also provide understanding of how dynamic and static
libraries are used, the Loader is included, which is not part of the compilation
but of the execution.

Here is the compilation/execution diagram:
```
COMPILATION

       Source code (.c, .h)
               |
               v
        .--------------.
        | Preprocessor |
        | (gcc -E/cpp) |
        '--------------'
               |
               v
    Expanded code (.i, .ii)
               |
               v
        .--------------.
        |   Compiler   |
        | (gcc -S/cc1) |
        '--------------'
               |
               v
    Assembly code (.s, .asm)
               |
               v
        .-------------.
        |  Assembler  |
        | (gcc -c/as) |
        '-------------'
               |
               v             .----------.
        Object file (.o) --> | Archiver | -> Static libraries
        with machine code    | (ar rcs) |     (.a, .lib)
               |             '----------'           |
               v                                    |
          .----------.                              |
          |  Linker  | <----------------------------'
          | (gcc/ld) |
          '----------'
               |
               +---------------------.
               v                     v
          Executables        Dynamic libraries
         (a.out, .exe)          (.so, .dll)
-----------------------------------------------------------------
 EXECUTION     |                     |
               v                     |
        .---------------.            |
        |    Loader     | <----------'
        | (ld-linux.so) |
        '---------------'
               |
               v
            Process
```

We can see that the different stages produce different kind of code, but also
notice that there is usually a `gcc` option to perform the operation, this is
because gcc is really just a unified frontend to the different compilation
phases. Instead of gcc, other compilers like clang could be used as well to
perform these steps, we will focus on gcc for clarity.


Let's see how `gcc` allows us to perform each of the stages by using
the following C program as example:

```c
#include <stdio.h>

void main() {
  printf("Hello world\n");
}
```
We save the previous program in a `helloworld.c` file.

## Preprocessing

The first step is the preprocessing, that creates what is known as the true C
source code. In this stage the C code is transformed by removing the comments,
and expanding all the lines that start with `#`, like `#include` or
`#define`. This means including the code of those headers in the source file as
well as resolving the macros and other compilation stuff.

We can preprocess our C file to create an expanded code file with the `-E`
flag (for Expand) in `gcc`:
```
gcc -E helloworld.c > helloworld.i
```

It is also possible to use [cpp](https://www.man7.org/linux/man-pages/man1/cpp.1.html) (the C PreProcessor) to preprocess the C
file:
```
cpp helloworld.c > helloworld.i
```

If we read our new `helloworld.i` file, you will see not only our program code
but also additional lines with the code from the `stdio.h` header file. We can
check it by just reading the .i file:

```shell
$ cat helloworld.i
# 0 "helloworld.c"
# 0 "<built-in>"
# 0 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 0 "<command-line>" 2
# 1 "helloworld.c"
# 1 "/usr/include/stdio.h" 1 3 4
# 28 "/usr/include/stdio.h" 3 4
....stripped....

typedef unsigned char __u_char;
typedef unsigned short int __u_short;
typedef unsigned int __u_int;
typedef unsigned long int __u_long;

....stripped....
# 3 "helloworld.c"
void main() {
  printf("Hello world\n");
}
```

## Compilation

Now we have the C code expanded, the next phase is to actually compile it. This
means to transform our C code into assembly code.

Compiling the expanded code file to create an assembly file can be done with the
`-S` flag in `gcc`:
```
gcc -S helloworld.i
```

> You can also specify the `-masm=intel` option to produce the assembly code in
> Intel syntax.

An alternative is to use `cc1` to compile the file, here is an example, but you
should use `gcc` since it passes the correct options to `cc1`:
```
/usr/libexec/gcc/x86_64-linux-gnu/14/cc1 helloworld.i
```

This will generate a `helloworld.s` file that contains assembly code. We can
check it the produced assembly code by reading the file (it is also a nice trick
to learn some assembly from compiler:)

```
$ cat helloworld.s
	.file	"helloworld.c"
	.text
	.section	.rodata
....stripped....
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	leaq	.LC0(%rip), %rax
	movq	%rax, %rdi
	call	puts@PLT
	nop
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
....stripped....
```

## Assemble

The next step is to assemble our code and produce the machine code. We can use
the `-c` flag in `gcc` that will create an object file that contains the machine
code as well as the indications for different symbols in the file:
```
gcc -c helloworld.s
```

> We can also include the gcc `-g` flag that will include debug information into
> the code file so it is easier to debug the program later.

Or alternatively, you could use [as](https://www.man7.org/linux/man-pages/man1/as.1.html):
```
as helloworld.s -o helloworld.o
```

Once executed we will have our `helloworld.o` code file. This is a binary file
that contains the machine code for the provided assembly file as well as symbols
information, in the ELF format, but it is not ready to be executed since it do
not contains the information about loading the program or the code of the
required static libraries.

> Apart from the ELF format, commonly used in Unix-like OS, there are others
> executable formats like PE, used by Windows, and Mach-O used by MacOS and iOS.

We can inspect this ELF file with the [objdump](https://www.man7.org/linux/man-pages/man1/objdump.1.html) or [readelf](https://www.man7.org/linux/man-pages/man1/readelf.1.html) programs:
```shell
$ file helloworld.o
helloworld.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
$ objdump -x helloworld.o

helloworld.o:     file format elf64-x86-64
helloworld.o
architecture: i386:x86-64, flags 0x00000011:
HAS_RELOC, HAS_SYMS
start address 0x0000000000000000

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000016  0000000000000000  0000000000000000  00000040  2**0
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000000  0000000000000000  0000000000000000  00000056  2**0
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000056  2**0
                  ALLOC
  3 .rodata       0000000c  0000000000000000  0000000000000000  00000056  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .comment      00000026  0000000000000000  0000000000000000  00000062  2**0
                  CONTENTS, READONLY
  5 .note.GNU-stack 00000000  0000000000000000  0000000000000000  00000088  2**0
                  CONTENTS, READONLY
  6 .eh_frame     00000038  0000000000000000  0000000000000000  00000088  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA
SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 helloworld.i
0000000000000000 l    d  .text	0000000000000000 .text
0000000000000000 l    d  .rodata	0000000000000000 .rodata
0000000000000000 g     F .text	0000000000000016 main
0000000000000000         *UND*	0000000000000000 puts


RELOCATION RECORDS FOR [.text]:
OFFSET           TYPE              VALUE
0000000000000007 R_X86_64_PC32     .rodata-0x0000000000000004
000000000000000f R_X86_64_PLT32    puts-0x0000000000000004


RELOCATION RECORDS FOR [.eh_frame]:
OFFSET           TYPE              VALUE
0000000000000020 R_X86_64_PC32     .text
```

## Archiving

Archiving is not properly a compilation phase, but just joining several `.o`
files into one `.a` file. This is specially useful to generate static libraries,
that are no more that pieces of machine code with symbols that any other program
can take to incorporate to its own code.

Let's see an example to clarify this. Imagine we want to create a simple
calculus library with the following c files: `add.c` and `sub.c`.

`add.c`:
```c
int add(int a, int b) {
  return a+b;
}
```

`sub.c`:
```c
int sub(int a, int b) {
  return a-b;
}
```

`calc.h`:
```c
#ifndef CALC_H
#define CALC_H

int add(int a, int b);
int sub(int a, int b);

#endif
```

First, we need to assemble the files into machine code. As we have seen, we
can use the `gcc -o` command:
```
gcc -o add.c sub.c
```

This will create two files, `add.o` and `sub.o`, that can be used by any other
program to incorporate their code. However, it can be annoying to keep track of
adding two files to get the calculus library code. The solution is to archive
the library files into one. We can do this with the [ar](https://www.man7.org/linux/man-pages/man1/ar.1.html) program:
```
ar rcs libcalc.a add.o sub.o 
```

We can inspect the generated `libcalc.a` to be sure the code of the `.o` files
was included:
```shell
$ objdump -t libcalc.a 
In archive libcalc.a:

add.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 add.c
0000000000000000 l    d  .text	0000000000000000 .text
0000000000000000 g     F .text	0000000000000018 add



sub.o:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 sub.c
0000000000000000 l    d  .text	0000000000000000 .text
0000000000000000 g     F .text	0000000000000016 sub
```

As we can see in the `objdump` output, there are two files contained in the
`libcalc.a` archive, `add.o` and `sub.o`. 

> You can also check what files are included in your system libraries by using
> `objdump` into libraries of `/usr/lib/x86_64-linux-gnu/` directory.

Once we have the archived library, we link it from other program to use its
functions.

Imagine we want to create a program to perform calculations. We can use
something like the following to use our calc library:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "calc.h"

int main(int argc, char** argv) {
  int a, b, result;
  char* op;
  if(argc < 4) {
    printf("Usage: %s operation number number\n", argv[0]);
    return -1;
  }

  op = argv[1];
  a = atoi(argv[2]);
  b = atoi(argv[3]);

  if(strcmp(op, "add") == 0) {
    result = add(a, b);
  } else if (strcmp(op, "sub") == 0) {
    result = sub(a, b);
  } else {
    printf("Invalid operation. Must be add or sub\n");
    return -1;
  }
  printf("Result: %d\n", result);
  return 0;
}
```

Then to compile (and link) our program to generate an executable we would need
to reference our calc library. 

On one hand we need to include the `calc.h` header that indicates the
definitions of the functions we want to use, that as we have seen will be
included in our program code in the stage of preprocessing. We already have
included it with the `#include "calc.h"` directive.

On the other hand we need to incorporate our library into the linking phase so
our program can take the code of the `add` and `sub` functions. To do this we
just need pass the `libcalc.a` file to gcc along with our program file (and make
sure that `calc.h` is in the current directory):
```
gcc calculator.c libcalc.a -o calculator
```

This will generate our `calculator` executable that will contain the code for
add and sub functions. We can check it by listing the symbols included in the
.text section of our `calculator` program:
```
$ objdump -t calculator | grep .text
00000000000010f0 l     F .text	0000000000000000              deregister_tm_clones
0000000000001120 l     F .text	0000000000000000              register_tm_clones
0000000000001160 l     F .text	0000000000000000              __do_global_dtors_aux
00000000000011a0 l     F .text	0000000000000000              frame_dummy
00000000000012b4 g     F .text	0000000000000018              add
00000000000010c0 g     F .text	0000000000000026              _start
00000000000011a9 g     F .text	000000000000010b              main
00000000000012cc g     F .text	0000000000000016              sub
```

We can see at the end the `add` and `sub` functions included in our
executable. Let's them try the program:
```
$ ./calculator add 1 1
Result: 2
$ ./calculator sub 1 1
Result: 0
```

Success!! We just have used the static library code.

## Linking

Once we have the object file(s) the final step is to link them with the code of
other libraries and include information to allow the loader to read our
executable and being able to run it, like the dynamic libraries that our program
relies on. (In this case we are not linking against any static library, but you
can see an example of this in the Archive section).

We just need to pass our object file to gcc:
```
gcc helloworld.o -o helloworld
```

This will finally generate our `helloworld` program that we can use:

```shell
$ ./helloworld
Hello world
```

Success!!

Apart from executables, the result of the linking stage can also be a dynamic
library. Let's see an example by reusing the source files of our Archive
example. This time instead of creating a static library we are going to create a
dynamic library.

First we need to link our files with the `-shared` option of `gcc` to create our
dynamic library (.so):
```
gcc -shared add.c sub.c -o libcalc.so
```

Then we can create a program to use it. Let's reuse our `calculator` program
from the Archive section. Same as the static library, we need to specify the
dynamic library when we link our program (and make sure that `calc.h` is in the
current directory):
```
gcc calculator.c libcalc.so -o calculator-dyn
```

We can now check how the code of `add` and `sub` functions is not included into
our calculator:
```shell
$ objdump -t calculator-dyn | grep .text
0000000000001130 l     F .text	0000000000000000              deregister_tm_clones
0000000000001160 l     F .text	0000000000000000              register_tm_clones
00000000000011a0 l     F .text	0000000000000000              __do_global_dtors_aux
00000000000011e0 l     F .text	0000000000000000              frame_dummy
0000000000001100 g     F .text	0000000000000026              _start
00000000000011e9 g     F .text	000000000000010b              main
```

Since the code of the `sub` and `add` functions is in `libcalc.so`, we cannot just
execute our calculator without telling it where `libcalc.so` is:
```
$ ./calculator-dyn 
./calculator-dyn: error while loading shared libraries: libcalc.so: cannot open shared object file: No such file or directory
```

For this purpose, we have two options, include `libcalc.so` in one of our
system library directories like `/usr/lib/x86_64-linux-gnu/` or just indicate
the directory where our library is in the `LD_LIBRARY_PATH` variable:
```shell
$ LD_LIBRARY_PATH=. ./calculator-dyn add 1 1
Result: 2
```

## References

- [The four stages of compilation](https://teachcomputerscience.com/the-four-stages-of-compilation/)
- [All you need to know about C Static libraries](https://dev.to/iamkhalil42/all-you-need-to-know-about-c-static-libraries-1o0b)
