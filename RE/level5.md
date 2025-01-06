# Reverse
```
Dump of assembler code for function main:
   0x080494e0 <+0>:     push   %ebp
   0x080494e1 <+1>:     mov    %esp,%ebp
   0x080494e3 <+3>:     push   %ebx
   0x080494e4 <+4>:     sub    $0x14,%esp
   0x080494e7 <+7>:     call   0x80494ec <main+12>
   0x080494ec <+12>:    pop    %ebx
   0x080494ed <+13>:    add    $0x2b14,%ebx
   0x080494f3 <+19>:    mov    %ebx,-0x8(%ebp)
   0x080494f6 <+22>:    lea    -0x1f7b(%ebx),%eax
   0x080494fc <+28>:    mov    %eax,(%esp)
   0x080494ff <+31>:    call   0x8049060 <printf@plt>
   0x08049504 <+36>:    mov    -0x8(%ebp),%ebx
   0x08049507 <+39>:    call   0x8049280 <password>
   0x0804950c <+44>:    xor    %eax,%eax
   0x0804950e <+46>:    add    $0x14,%esp
   0x08049511 <+49>:    pop    %ebx
   0x08049512 <+50>:    pop    %ebp
   0x08049513 <+51>:    ret

void main() {
  printf("Let's start a more complex game!\n");
  password();
  return;
}
```

```
Dump of assembler code for function password:
   0x08049280 <+0>:     push   %ebp
   0x08049281 <+1>:     mov    %esp,%ebp
   0x08049283 <+3>:     push   %ebx
   0x08049284 <+4>:     sub    $0x234,%esp
   0x0804928a <+10>:    call   0x804928f <password+15>
   0x0804928f <+15>:    pop    %ebx
   0x08049290 <+16>:    add    $0x2d71,%ebx
   0x08049296 <+22>:    mov    %ebx,-0x224(%ebp)
   0x0804929c <+28>:    lea    -0x204(%ebp),%eax
   0x080492a2 <+34>:    xor    %ecx,%ecx
   0x080492a4 <+36>:    mov    %eax,(%esp)
   0x080492a7 <+39>:    movl   $0x0,0x4(%esp)
   0x080492af <+47>:    movl   $0x200,0x8(%esp)
   0x080492b7 <+55>:    call   0x8049090 <memset@plt>

memset(buffer, 0, 0x200);  // buffer @ ebp_204

   0x080492bc <+60>:    movl   $0x0,-0x210(%ebp)
   0x080492c6 <+70>:    cmpl   $0x200,-0x210(%ebp)
   0x080492d0 <+80>:    jae    0x804935b <password+219>

int i = 0; // ebp_210
if (i >= 0x200) jmp;

   0x080492d6 <+86>:    mov    -0x224(%ebp),%ebx
   0x080492dc <+92>:    lea    -0x208(%ebp),%eax
   0x080492e2 <+98>:    mov    %eax,-0x20c(%ebp)
   0x080492e8 <+104>:   mov    -0x20c(%ebp),%eax
   0x080492ee <+110>:   xor    %ecx,%ecx
   0x080492f0 <+112>:   movl   $0x0,(%esp)
   0x080492f7 <+119>:   mov    %eax,0x4(%esp)
   0x080492fb <+123>:   movl   $0x1,0x8(%esp)
   0x08049303 <+131>:   call   0x8049050 <read@plt>
   0x08049308 <+136>:   mov    -0x20c(%ebp),%eax

read(0, &ebp-0x208, 1);

   0x0804930e <+142>:   movsbl (%eax),%eax
   0x08049311 <+145>:   cmp    $0xa,%eax
   0x08049314 <+148>:   jne    0x804932d <password+173>

if (input != '\n') jmp;

   0x0804931a <+154>:   mov    -0x210(%ebp),%eax
   0x08049320 <+160>:   movb   $0x0,-0x204(%ebp,%eax,1)

buffer[i] = 0;  // buffer[i] = '\0';

   0x08049328 <+168>:   jmp    0x804935b <password+219>

   0x0804932d <+173>:   mov    -0x20c(%ebp),%eax
   0x08049333 <+179>:   mov    (%eax),%cl
   0x08049335 <+181>:   mov    -0x210(%ebp),%eax
   0x0804933b <+187>:   mov    %cl,-0x204(%ebp,%eax,1)

buffer[i] = input;

   0x08049342 <+194>:   jmp    0x8049347 <password+199>
   0x08049347 <+199>:   mov    -0x210(%ebp),%eax
   0x0804934d <+205>:   add    $0x1,%eax
   0x08049350 <+208>:   mov    %eax,-0x210(%ebp)
   0x08049356 <+214>:   jmp    0x80492c6 <password+70>

i++;
loop
_________
for (int i = 0; i < 0x200; ++i) {
    read(stdin, &c, 1);
    if (c == '\n') {
        buffer[i] = '\0';
        break;
    }
    else
        buffer[i] = c;
}
_________

   0x0804935b <+219>:   movl   $0x0,-0x218(%ebp)

int i = 0;

   0x08049365 <+229>:   cmpl   $0x200,-0x218(%ebp)
   0x0804936f <+239>:   jae    0x80493b6 <password+310>
   0x08049375 <+245>:   mov    -0x218(%ebp),%eax
   0x0804937b <+251>:   movsbl -0x204(%ebp,%eax,1),%eax
   0x08049383 <+259>:   cmp    $0x0,%eax
   0x08049386 <+262>:   jne    0x804939d <password+285>

if (buffer[i] != '\0') jmp;

   0x0804938c <+268>:   mov    -0x218(%ebp),%eax
   0x08049392 <+274>:   mov    %eax,-0x214(%ebp)
   0x08049398 <+280>:   jmp    0x80493b6 <password+310>
   0x0804939d <+285>:   jmp    0x80493a2 <password+290>
   0x080493a2 <+290>:   mov    -0x218(%ebp),%eax
   0x080493a8 <+296>:   add    $0x1,%eax
   0x080493ab <+299>:   mov    %eax,-0x218(%ebp)
   0x080493b1 <+305>:   jmp    0x8049365 <password+229>

++i;
loop

   0x080493b6 <+310>:   cmpl   $0x7,-0x214(%ebp)
   0x080493bd <+317>:   je     0x80493e9 <password+361>

if (i == 7) jmp;

   0x080493c3 <+323>:   mov    -0x224(%ebp),%ebx
   0x080493c9 <+329>:   lea    -0x1fd1(%ebx),%eax
   0x080493cf <+335>:   mov    %eax,(%esp)
   0x080493d2 <+338>:   call   0x8049060 <printf@plt>
   0x080493d7 <+343>:   mov    -0x224(%ebp),%ebx
   0x080493dd <+349>:   movl   $0xffffffff,(%esp)
   0x080493e4 <+356>:   call   0x8049080 <exit@plt>

printf("Wrong string length!\n");
exit(-1);
___________
for (int i = 0; i < 0x200; ++i) {
    if (buffer[i] == '\0') {
        if (i != 7) {
            print("Wrong string length!\n");
            exit(-1);
        }
        break;
    }
}
___________

   0x080493e9 <+361>:   mov    -0x224(%ebp),%ebx
   0x080493ef <+367>:   lea    -0x204(%ebp),%eax
   0x080493f5 <+373>:   mov    %eax,(%esp)
   0x080493f8 <+376>:   call   0x80490d0 <atoi@plt>
   0x080493fd <+381>:   mov    -0x224(%ebp),%ebx
   0x08049403 <+387>:   mov    %eax,-0x21c(%ebp)

int atoi_ret = atoi(buffer);

   0x08049409 <+393>:   lea    -0x204(%ebp),%edx
   0x0804940f <+399>:   mov    -0x21c(%ebp),%eax
   0x08049415 <+405>:   lea    -0x1fbb(%ebx),%ecx
   0x0804941b <+411>:   mov    %edx,(%esp)
   0x0804941e <+414>:   movl   $0x14,0x4(%esp)
   0x08049426 <+422>:   mov    %ecx,0x8(%esp)
   0x0804942a <+426>:   mov    %eax,0xc(%esp)
   0x0804942e <+430>:   call   0x80490a0 <snprintf@plt>

snprintf(buffer, 0x14, "%u", atoi_ret);

   0x08049433 <+435>:   movl   $0x0,-0x220(%ebp)

int i = 0;

   0x0804943d <+445>:   cmpl   $0xa,-0x220(%ebp)
   0x08049444 <+452>:   jge    0x80494b6 <password+566>
   0x0804944a <+458>:   mov    -0x224(%ebp),%ecx
   0x08049450 <+464>:   mov    -0x220(%ebp),%eax
   0x08049456 <+470>:   movsbl -0x204(%ebp,%eax,1),%eax
   0x0804945e <+478>:   xor    $0x1,%eax
   0x08049461 <+481>:   mov    -0x220(%ebp),%edx
   0x08049467 <+487>:   movsbl 0x3c(%ecx,%edx,1),%ecx
   0x0804946f <+495>:   cmp    %ecx,%eax
   0x08049471 <+497>:   je     0x804949d <password+541>

ecx+0x3c = password_string = "5385876380";
if (buffer[i]^1 == password_string[i]) jmp;

   0x08049477 <+503>:   mov    -0x224(%ebp),%ebx
   0x0804947d <+509>:   lea    -0x1fb8(%ebx),%eax
   0x08049483 <+515>:   mov    %eax,(%esp)
   0x08049486 <+518>:   call   0x8049060 <printf@plt>
   0x0804948b <+523>:   mov    -0x224(%ebp),%ebx
   0x08049491 <+529>:   movl   $0xffffffff,(%esp)
   0x08049498 <+536>:   call   0x8049080 <exit@plt>

printf("Wrong password, access denied.\n");
exit(-1);

   0x0804949d <+541>:   jmp    0x80494a2 <password+546>
   0x080494a2 <+546>:   mov    -0x220(%ebp),%eax
   0x080494a8 <+552>:   add    $0x1,%eax
   0x080494ab <+555>:   mov    %eax,-0x220(%ebp)
   0x080494b1 <+561>:   jmp    0x804943d <password+445>

++i;
loop
____________
for (int i = 0; i < 0xa; ++i) {
    if (buffer[i] ^ 1 != password_string[i]) {
        printf("Wrong password, access denied.\n");
        exit(-1);
    }
}
____________


   0x080494b6 <+566>:   mov    -0x224(%ebp),%ebx
   0x080494bc <+572>:   lea    -0x1f98(%ebx),%eax
   0x080494c2 <+578>:   mov    %eax,(%esp)
   0x080494c5 <+581>:   call   0x8049060 <printf@plt>
   0x080494ca <+586>:   mov    -0x224(%ebp),%ebx
   0x080494d0 <+592>:   call   0x8049200 <get_a_shell>

printf("Great! you got my password!\n");
get_a_shell();

   0x080494d5 <+597>:   add    $0x234,%esp
   0x080494db <+603>:   pop    %ebx
   0x080494dc <+604>:   pop    %ebp
   0x080494dd <+605>:   ret

void password() {
  char buffer[200];
  memset(buffer, 0, 0x200);

  for (int i = 0; i < 0x200; ++i) {
    char c;
    read(stdin, &c, 1);
    if (c == '\n') {
        buffer[i] = '\0';
        break;
    }
    else
        buffer[i] = c;
  }

  for (int i = 0; i < 0x200; ++i) {
    if (buffer[i] == '\0') {
        if (i != 7) {
            print("Wrong string length!\n");
            exit(-1);
        }
        break;
    }
  }

  atoi_ret = atoi(buffer);
  snprintf(buffer, 0x14, "%u", atoi_ret);
 
  char *password_string = "5385876380";
  for (int i = 0; i < 0xa; ++i) {
    if (buffer[i] ^ 1 != password_string[i]) {
        printf("Wrong password, access denied.\n");
        exit(-1);
    }
  }

  printf("Great! you got my password!\n");
  get_a_shell();
  return;
}
```

# Solving
To figure out the password, we work backward from where we know it needs to end: `password_string`. 
Right before comparing `buffer[i]` to `password_string[i]`, `buffer[i]` is XORed with 1. 

```py
>>> tmp = "5385876380" 
>>> val = ''    
>>> for n in tmp:             
...     val += chr(ord(n)^1)
...
'4294967291'
```

The main problem is that we can't input the entire number as anything longer than 7 characters is flagged as
incorrect and then the program exits. The key to correcting this is in atoi() and snprintf(). 

snprintf() takes the output from atoi() and copies it as an unsigned int into `buffer`, but atoi() returns ints, 
not uints. So we can use this to input our bigger number. 

The max value for an unsigned int is 4,294,967,295. Subtracting our goal number from the max and then negating the value gives us -5, which when
represented as an uint will give our goal. To turn it into a 7 character input, simply frontload zeros. 

```
$ ./level5
Let's start a more complex game!
-000005
Great! you got my password!
Spawning a privileged shell

$ cat flag
cand{two_s_complement}
```
