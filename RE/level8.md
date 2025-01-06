# Reversing
```
Dump of assembler code for function main:
   0x080486f0 <+0>:     push   %ebp
   0x080486f1 <+1>:     mov    %esp,%ebp
   0x080486f3 <+3>:     sub    $0x18,%esp
   0x080486f6 <+6>:     lea    0x8048868,%eax
   0x080486fc <+12>:    mov    %eax,(%esp)
   0x080486ff <+15>:    call   0x80483e0 <printf@plt>
   0x08048704 <+20>:    xor    %ecx,%ecx
   0x08048706 <+22>:    movl   $0x0,(%esp)
   0x0804870d <+29>:    mov    %eax,-0x4(%ebp)
   0x08048710 <+32>:    mov    %ecx,-0x8(%ebp)
   0x08048713 <+35>:    call   0x80483f0 <time@plt>
   0x08048718 <+40>:    mov    %eax,(%esp)
   0x0804871b <+43>:    call   0x8048410 <srand@plt>
   0x08048720 <+48>:    call   0x80485f0 <password>
   0x08048725 <+53>:    xor    %eax,%eax
   0x08048727 <+55>:    add    $0x18,%esp
   0x0804872a <+58>:    pop    %ebp
   0x0804872b <+59>:    ret
```
```c
void main() {
  printf("Let's start a THREE FOUR game!\n");
  uint seed = time();
  srand(seed);
  password();
  return;
}
```

```
Dump of assembler code for function password:
   0x080485f0 <+0>:     push   %ebp
   0x080485f1 <+1>:     mov    %esp,%ebp
   0x080485f3 <+3>:     push   %esi
   0x080485f4 <+4>:     sub    $0x34,%esp
   0x080485f7 <+7>:     lea    0x80487d7,%eax
   0x080485fd <+13>:    mov    %eax,(%esp)
   0x08048600 <+16>:    call   0x80483e0 <printf@plt>

printf("THREE FOUR MACHINES\n");

   0x08048605 <+21>:    mov    %eax,-0x18(%ebp)
   0x08048608 <+24>:    call   0x8048440 <rand@plt>

eax = rand();

   0x0804860d <+29>:    lea    0x80487ec,%ecx
   0x08048613 <+35>:    mov    $0x14,%edx
   0x08048618 <+40>:    mov    %edx,-0x1c(%ebp)

int num = 0x14;

   0x0804861b <+43>:    cltd
   0x0804861c <+44>:    mov    -0x1c(%ebp),%esi
   0x0804861f <+47>:    idiv   %esi

eax = rand() / num;
edx = rand() % num;

   0x08048621 <+49>:    add    $0xf,%edx
   0x08048624 <+52>:    mov    %edx,-0x8(%ebp)
   0x08048627 <+55>:    mov    -0x8(%ebp),%edx

int lucky_num = rand() % num + 0xf; // ebp_8

   0x0804862a <+58>:    mov    %ecx,(%esp)
   0x0804862d <+61>:    mov    %edx,0x4(%esp)
   0x08048631 <+65>:    call   0x80483e0 <printf@plt>

printf("I will use %d for my lucky number.\n", lucky_num);

   0x08048636 <+70>:    lea    0x8048810,%ecx
   0x0804863c <+76>:    mov    %ecx,(%esp)
   0x0804863f <+79>:    mov    %eax,-0x20(%ebp)
   0x08048642 <+82>:    call   0x80483e0 <printf@plt>

printf("Please send me your lucky number.\n");

   0x08048647 <+87>:    lea    0x8048833,%ecx
   0x0804864d <+93>:    lea    -0xc(%ebp),%edx
   0x08048650 <+96>:    mov    %ecx,(%esp)
   0x08048653 <+99>:    mov    %edx,0x4(%esp)
   0x08048657 <+103>:   mov    %eax,-0x24(%ebp)
   0x0804865a <+106>:   call   0x8048460 <__isoc99_scanf@plt>

int input; //ebp_c
scanf("%d", input);

   0x0804865f <+111>:   movl   $0x0,-0x10(%ebp)
   0x08048666 <+118>:   mov    %eax,-0x28(%ebp)
   0x08048669 <+121>:   mov    -0x10(%ebp),%eax
   0x0804866c <+124>:   cmp    -0x8(%ebp),%eax
   0x0804866f <+127>:   jge    0x80486a3 <password+179>

int i = 0;
if ( i >= lucky_num) jmp;

@ start 0x804a03c = keys+4 = 0x4, 0x804a038 = keys = 0x3
   0x08048675 <+133>:   mov    0x804a03c,%eax
   0x0804867a <+138>:   mov    %eax,-0x14(%ebp)
   0x0804867d <+141>:   mov    0x804a038,%eax
   0x08048682 <+146>:   add    0x804a03c,%eax
   0x08048688 <+152>:   mov    %eax,0x804a03c
   0x0804868d <+157>:   mov    -0x14(%ebp),%eax
   0x08048690 <+160>:   mov    %eax,0x804a038

tmp = keys+4;
keys+4 += keys;
keys = tmp;

   0x08048695 <+165>:   mov    -0x10(%ebp),%eax
   0x08048698 <+168>:   add    $0x1,%eax
   0x0804869b <+171>:   mov    %eax,-0x10(%ebp)
   0x0804869e <+174>:   jmp    0x8048669 <password+121>

++i;
loop;
________
int keys[2] = [3, 4];
for (int i = 0; i < lucky_num; ++i) {
    int tmp = keys[1];
    keys[1] += keys[0];
    keys[0] = tmp;
}
________

   0x080486a3 <+179>:   mov    0x804a03c,%eax
   0x080486a8 <+184>:   cmp    -0xc(%ebp),%eax
   0x080486ab <+187>:   jne    0x80486cc <password+220>

if (input != keys[1]) jmp;

   0x080486b1 <+193>:   lea    0x8048836,%eax
   0x080486b7 <+199>:   mov    %eax,(%esp)
   0x080486ba <+202>:   call   0x80483e0 <printf@plt>
   0x08486bf <+207>:    mov    %eax,-0x2c(%ebp)
   0x080486c2 <+210>:   call   0x8048580 <get_a_shell>
   0x080486c7 <+215>:   jmp    0x80486dd <password+237>

printf("Great! you got my password!\n");
get_a_shell();
return;

   0x080486cc <+220>:   lea    0x8048853,%eax
   0x080486d2 <+226>:   mov    %eax,(%esp)
   0x080486d5 <+229>:   call   0x80483e0 <printf@plt>

printf("Uh oh, wrong value.\n");

   0x080486da <+234>:   mov    %eax,-0x30(%ebp)
   0x080486dd <+237>:   add    $0x34,%esp
   0x080486e0 <+240>:   pop    %esi
   0x080486e1 <+241>:   pop    %ebp
   0x080486e2 <+242>:   ret
```
```c
void password() {
  printf("THREE FOUR MACHINES\n");

  int num = 0x14;
  int lucky_num = rand() % num + 0xf;
  printf("I will use %d for my lucky number.\n", lucky_num);
  printf("Please send me your lucky number.\n");

  int input;
  scanf("%d", input);

  int keys[2] = [3, 4];
  for (int i = 0; i < lucky_num; ++i) {
    int tmp = keys[1];
    keys[1] += keys[0];
    keys[0] = tmp;
  }

  if (input == keys[1]) {
    printf("Great! you got my password!\n");
    get_a_shell();
    return;
  }

  printf("Uh oh, wrong value.\n");
  return;
}
```

# Solving
Solving this one is pretty simple. We need to start running the program so that we know that the lucky number is, and then recreate the loop
that calculates what our input should be. For this particular run, I got 30 as the lucky number. 

```py
>>> keys = [3, 4]
>>> for _ in range(0, 30):
...     tmp = keys[1]
...     keys[1] += keys[0]
...     keys[0] = tmp
... 
>>> print(keys[1])
7881196
```

This means that our input should be 7881196.

```
$ ./level8
Let's start a THREE FOUR game!
THREE FOUR MACHINES
I will use 30 for my lucky number.
Please send me your lucky number.
7881196
Great! you got my password!
Spawning a privileged shell

$ cat flag
cand{rabbit_numbers}
```
