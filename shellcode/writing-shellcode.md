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

