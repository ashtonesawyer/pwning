# Reversing
```
Dump of assembler code for function main:
   0x08048740 <+0>:     push   %ebp
   0x08048741 <+1>:     mov    %esp,%ebp
   0x08048743 <+3>:     sub    $0x8,%esp
   0x08048746 <+6>:     call   0x8048570 <number_generator>
   0x0804874b <+11>:    lea    0x804886a,%eax
   0x08048751 <+17>:    mov    %eax,(%esp)
   0x08048754 <+20>:    call   0x8048390 <printf@plt>
   0x08048759 <+25>:    mov    %eax,-0x4(%ebp)
   0x0804875c <+28>:    call   0x80486c0 <password>
   0x08048761 <+33>:    xor    %eax,%eax
   0x08048763 <+35>:    add    $0x8,%esp
   0x08048766 <+38>:    pop    %ebp
   0x08048767 <+39>:    ret
```

```c
void main() {
    number_generator();
    printf("Let's start a game!\n");
    password();
   return;
}
```

```
Dump of assembler code for function number_generator:
   0x08048570 <+0>:     push   %ebp
   0x08048571 <+1>:     mov    %esp,%ebp
   0x08048573 <+3>:     sub    $0x10,%esp
   0x08048576 <+6>:     movl   $0x2,-0x4(%ebp)
   0x0804857d <+13>:    movl   $0x2,0x804a02c
   0x08048587 <+23>:    movl   $0x3,0x804a030
   0x08048591 <+33>:    movl   $0x4,-0x8(%ebp)
   0x08048598 <+40>:    cmpl   $0x1f4,-0x4(%ebp)
   0x0804859f <+47>:    jge    0x804863d <number_generator+205>
   0x080485a5 <+53>:    movl   $0x0,-0xc(%ebp)
   0x080485ac <+60>:    mov    -0xc(%ebp),%eax
   0x080485af <+63>:    cmp    -0x4(%ebp),%eax
   0x080485b2 <+66>:    jge    0x8048618 <number_generator+168>
   0x080485b8 <+72>:    mov    -0xc(%ebp),%eax
   0x080485bb <+75>:    mov    0x804a02c(,%eax,4),%eax
   0x080485c2 <+82>:    mov    %eax,-0x10(%ebp)
   0x080485c5 <+85>:    mov    -0x10(%ebp),%eax
   0x080485c8 <+88>:    imul   -0x10(%ebp),%eax
   0x080485cc <+92>:    cmp    -0x8(%ebp),%eax
   0x080485cf <+95>:    jle    0x80485f0 <number_generator+128>
   0x080485d5 <+101>:   mov    -0x8(%ebp),%eax
   0x080485d8 <+104>:   mov    -0x4(%ebp),%ecx
   0x080485db <+107>:   mov    %eax,0x804a02c(,%ecx,4)
   0x080485e2 <+114>:   mov    -0x4(%ebp),%eax
   0x080485e5 <+117>:   add    $0x1,%eax
   0x080485e8 <+120>:   mov    %eax,-0x4(%ebp)
   0x080485eb <+123>:   jmp    0x8048618 <number_generator+168>
   0x080485f0 <+128>:   mov    -0x8(%ebp),%eax
   0x080485f3 <+131>:   cltd
   0x080485f4 <+132>:   idivl  -0x10(%ebp)
   0x080485f7 <+135>:   cmp    $0x0,%edx
   0x080485fa <+138>:   jne    0x8048605 <number_generator+149>
   0x08048600 <+144>:   jmp    0x8048618 <number_generator+168>
   0x08048605 <+149>:   jmp    0x804860a <number_generator+154>
   0x0804860a <+154>:   mov    -0xc(%ebp),%eax
   0x0804860d <+157>:   add    $0x1,%eax
   0x08048610 <+160>:   mov    %eax,-0xc(%ebp)
   0x08048613 <+163>:   jmp    0x80485ac <number_generator+60>
   0x08048618 <+168>:   cmpl   $0x1f4,-0x4(%ebp)
   0x0804861f <+175>:   jne    0x804862a <number_generator+186>
   0x08048625 <+181>:   jmp    0x804863d <number_generator+205>
   0x0804862a <+186>:   jmp    0x804862f <number_generator+191>
   0x0804862f <+191>:   mov    -0x8(%ebp),%eax
   0x08048632 <+194>:   add    $0x1,%eax
   0x08048635 <+197>:   mov    %eax,-0x8(%ebp)
   0x08048638 <+200>:   jmp    0x8048598 <number_generator+40>
   0x0804863d <+205>:   add    $0x10,%esp
   0x08048640 <+208>:   pop    %ebp
   0x08048641 <+209>:   ret
```

A quick scan of the `number_generator()`, shows that `rand()` and similar are not called, so I moved forward with the assumption that
whatever it did I would be able to observe in memory, and could skip actually reversing the function. As such, I moved straight to
working on `password()`. 

```
Dump of assembler code for function password:
   0x080486c0 <+0>:     push   %ebp
   0x080486c1 <+1>:     mov    %esp,%ebp
   0x080486c3 <+3>:     sub    $0x28,%esp
   0x080486c6 <+6>:     lea    0x8048817,%eax
   0x080486cc <+12>:    mov    %eax,(%esp)
   0x080486cf <+15>:    call   0x8048390 <printf@plt>

printf("Please send me the password.\n");

   0x080486d4 <+20>:    lea    0x8048835,%ecx
   0x080486da <+26>:    lea    -0x4(%ebp),%edx
   0x080486dd <+29>:    mov    %ecx,(%esp)
   0x080486e0 <+32>:    mov    %edx,0x4(%esp)
   0x080486e4 <+36>:    mov    %eax,-0x8(%ebp)
   0x080486e7 <+39>:    call   0x80483e0 <__isoc99_scanf@plt>

scanf("%d", input); // ebp_4

   0x080486ec <+44>:    mov    -0x4(%ebp),%ecx
   0x080486ef <+47>:    mov    %ecx,(%esp)
   0x080486f2 <+50>:    mov    %eax,-0xc(%ebp)
   0x080486f5 <+53>:    call   0x8048650 <check>
   0x080486fa <+58>:    cmp    $0x0,%eax
   0x080486fd <+61>:    je     0x804871e <password+94>

if (!check(input)) jmp;

   0x08048703 <+67>:    lea    0x8048838,%eax
   0x08048709 <+73>:    mov    %eax,(%esp)
   0x0804870c <+76>:    call   0x8048390 <printf@plt>
   0x08048711 <+81>:    mov    %eax,-0x10(%ebp)
   0x08048714 <+84>:    call   0x8048500 <get_a_shell>
   0x08048719 <+89>:    jmp    0x804872f <password+111>

printf("Great! you got my password!\n");
get_a_shell();
return;

   0x0804871e <+94>:    lea    0x8048855,%eax
   0x08048724 <+100>:   mov    %eax,(%esp)
   0x08048727 <+103>:   call   0x8048390 <printf@plt>

printf("Uh oh, wrong value.\n");

   0x0804872c <+108>:   mov    %eax,-0x14(%ebp)
   0x0804872f <+111>:   add    $0x28,%esp
   0x08048732 <+114>:   pop    %ebp
   0x08048733 <+115>:   ret
```

```c
void password() {
   int input;
   printf("Please send me the password.\n");
   scanf("%d", input);
   
   if (check(input)) {
      printf("Great! you got my password!\n");
      get_a_shell();
   }
   else 
      printf("Uh oh, wrong value.\n");
      
   return;
}
```

```
Dump of assembler code for function check:
   0x08048650 <+0>:     push   %ebp
   0x08048651 <+1>:     mov    %esp,%ebp
   0x08048653 <+3>:     sub    $0xc,%esp
   0x08048656 <+6>:     mov    0x8(%ebp),%eax
   0x08048659 <+9>:     mov    %eax,-0x8(%ebp)
   0x0804865c <+12>:    movl   $0x0,-0xc(%ebp)

int num = input; //ebp_8
int i = 0;       // ebp_c

   0x08048663 <+19>:    cmpl   $0x1f4,-0xc(%ebp)
   0x0804866a <+26>:    jge    0x80486a6 <check+86>

if (i >= 0x1f4) return 1;

   0x08048670 <+32>:    mov    -0x8(%ebp),%eax
   0x08048673 <+35>:    mov    -0xc(%ebp),%ecx
   0x08048676 <+38>:    cltd
   0x08048677 <+39>:    idivl  0x804a02c(,%ecx,4)

int keys[] @ 0x804a02c. Appears to be array of primes -> {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, ...}

   0x0804867e <+46>:    cmp    $0x0,%edx
   0x08048681 <+49>:    jne    0x8048693 <check+67>

if (input % keys[i] != 0) jmp;

   0x08048687 <+55>:    movl   $0x0,-0x4(%ebp)
   0x0804868e <+62>:    jmp    0x80486ad <check+93>

return 0;

   0x08048693 <+67>:    jmp    0x8048698 <check+72>
   0x08048698 <+72>:    mov    -0xc(%ebp),%eax
   0x0804869b <+75>:    add    $0x1,%eax
   0x0804869e <+78>:    mov    %eax,-0xc(%ebp)
   0x080486a1 <+81>:    jmp    0x8048663 <check+19>
   0x080486a6 <+86>:    movl   $0x1,-0x4(%ebp)
   0x080486ad <+93>:    mov    -0x4(%ebp),%eax
   0x080486b0 <+96>:    add    $0xc,%esp
   0x080486b3 <+99>:    pop    %ebp
   0x080486b4 <+100>:   ret
```

```c
int check(int input) {
   int num = input;
   for (int i = 0; i < 0x1f4; ++i) {
      if (input % array_of_primes[i] == 0) return 0;
   }
   return 1;
}
```

# Solving
The goal is for the return of `check()` to be 1. `check()` returns 1 when 
there is a remainder for many prime numbers. The easiest way to do that is to input a number where 
abs(input) is lower than the lowest number in the array of primes. So the easiest input is 1. 

```
$ ./level9
Let's start a game!
Please send me the password.
1
Great! you got my password!
Spawning a privileged shell

$ cat flag 
cand{prime_nUmbers}
```
