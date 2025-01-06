Dump of assembler code for function main:
   0x56556480 <+0>:     push   %ebp                                                                                                                                                                     
   0x56556481 <+1>:     mov    %esp,%ebp
   0x56556483 <+3>:     sub    $0x8,%esp
   0x56556486 <+6>:     lea    0x5655709c,%eax
   0x5655648c <+12>:    mov    %eax,(%esp)
   0x5655648f <+15>:    call   0xf7dd5a90 <__printf>
   0x56556494 <+20>:    call   0x565562f0 <password>
   0x56556499 <+25>:    xor    %eax,%eax
   0x5655649b <+27>:    add    $0x8,%esp
   0x5655649e <+30>:    pop    %ebp
   0x5655649f <+31>:    ret 

void main() {
  printf("Let's start an esoteric game!\n");
  password();
  exit(1);
}

Dump of assembler code for function password:
   0x565562f0 <+0>:     push   %ebp
   0x565562f1 <+1>:     mov    %esp,%ebp
   0x565562f3 <+3>:     sub    $0x218,%esp
   0x565562f9 <+9>:     lea    -0x200(%ebp),%eax
   0x565562ff <+15>:    xor    %ecx,%ecx
   0x56556301 <+17>:    mov    %eax,(%esp)
   0x56556304 <+20>:    movl   $0x0,0x4(%esp)
   0x5655630c <+28>:    movl   $0x200,0x8(%esp)
   0x56556314 <+36>:    call   0x56556080 <memset@plt>

char buffer[200];  //ebp_200
memset(buffer, 0, 0x200);

   0x56556319 <+41>:    lea    -0x200(%ebp),%eax
   0x5655631f <+47>:    lea    0x5655702f,%ecx
   0x56556325 <+53>:    mov    %ecx,(%esp)
   0x56556328 <+56>:    mov    %eax,0x4(%esp)
   0x5655632c <+60>:    call   0xf7dd6c60 <__isoc99_scanf>

scanf("%50s", buffer);

   0x56556331 <+65>:    lea    -0x200(%ebp),%eax
   0x56556337 <+71>:    lea    0x56557034,%ecx
   0x5655633d <+77>:    mov    %ecx,(%esp)
   0x56556340 <+80>:    mov    %eax,0x4(%esp)
   0x56556344 <+84>:    call   0xf7dd5a90 <__printf>

printf("Your buffer: %s\n", buffer);

   0x56556349 <+89>:    lea    -0x200(%ebp),%ecx
   0x5655634f <+95>:    mov    %esp,%eax
   0x56556351 <+97>:    mov    %ecx,(%eax)
   0x56556353 <+99>:    call   0xf7e20680 <__strlen_sse2_bsf>
   0x56556358 <+104>:   mov    %eax,-0x204(%ebp)

size_t len = strlen(buffer); 

   0x5655635e <+110>:   lea    -0x200(%ebp),%ecx
   0x56556364 <+116>:   mov    -0x204(%ebp),%eax
   0x5655636a <+122>:   mov    %ecx,(%esp)
   0x5655636d <+125>:   mov    %eax,0x4(%esp)
   0x56556371 <+129>:   call   0xf7e1b470 <memfrob>

 memfrob(buffer, len);

   0x56556376 <+134>:   lea    -0x200(%ebp),%eax
   0x5655637c <+140>:   mov    %eax,(%esp)
   0x5655637f <+143>:   call   0x56556290 <add_a_space>

add_a_space(buffer);

---------------
Dump of assembler code for function add_a_space:
   0x56556290 <+0>:     push   %ebp
   0x56556291 <+1>:     mov    %esp,%ebp
   0x56556293 <+3>:     push   %eax
   0x56556294 <+4>:     mov    0x8(%ebp),%eax
   0x56556297 <+7>:     movl   $0x0,-0x4(%ebp)
   0x5655629e <+14>:    mov    0x8(%ebp),%eax
   0x565562a1 <+17>:    mov    -0x4(%ebp),%ecx
   0x565562a4 <+20>:    movsbl (%eax,%ecx,1),%eax
   0x565562a8 <+24>:    cmp    $0x0,%eax
   0x565562ab <+27>:    je     0x565562dc <add_a_space+76>

if (buffer + (i*1) != 0) {   
  // do smth
}
else 
  return;
                                    
NOTE: since we're working with a string, buffer[i] != 0 may be better represented with buffer[i] != '\0'                         
   
   0x565562b1 <+33>:    mov    0x8(%ebp),%eax
   0x565562b4 <+36>:    mov    -0x4(%ebp),%ecx
   0x565562b7 <+39>:    movsbl (%eax,%ecx,1),%eax
   0x565562bb <+43>:    add    $0x20,%eax
   0x565562be <+46>:    and    $0xff,%eax
   0x565562c3 <+51>:    mov    %al,%dl
   0x565562c5 <+53>:    mov    0x8(%ebp),%eax
   0x565562c8 <+56>:    mov    -0x4(%ebp),%ecx
   0x565562cb <+59>:    mov    %dl,(%eax,%ecx,1)

buffer[i] += 0x20;
buffer[i] &= 0xff;  // buffer[i] & 1 == buffer[i]
   
   0x565562ce <+62>:    mov    -0x4(%ebp),%eax
   0x565562d1 <+65>:    add    $0x1,%eax
   0x565562d4 <+68>:    mov    %eax,-0x4(%ebp)
   0x565562d7 <+71>:    jmp    0x5655629e <add_a_space+14>

i += 1;
loop

   0x565562dc <+76>:    add    $0x4,%esp
   0x565562df <+79>:    pop    %ebp
   0x565562e0 <+80>:    ret

void add_a_space(void* buff) {
  for (int i = 0; buffer[i] != '\0'; ++i) {
    buff[i] += 0x20;
  }
}
---------

   0x56556384 <+148>:   lea    -0x200(%ebp),%eax
   0x5655638a <+154>:   mov    %eax,(%esp)
   0x5655638d <+157>:   call   0xf7db6c40 <__GI_atoi>
   0x56556392 <+162>:   mov    %eax,-0x208(%ebp)

int atoi_ret = atoi(buffer);

   0x56556398 <+168>:   mov    -0x208(%ebp),%eax
   0x5655639e <+174>:   lea    0x56557045,%ecx
   0x565563a4 <+180>:   mov    %ecx,(%esp)
   0x565563a7 <+183>:   mov    %eax,0x4(%esp)
   0x565563ab <+187>:   call   0xf7dd5a90 <__printf>

printf("Your Integer: %u\n", atoi_ret);

   0x565563b0 <+192>:   lea    -0x200(%ebp),%edx
   0x565563b6 <+198>:   mov    -0x208(%ebp),%eax
   0x565563bc <+204>:   lea    0x56557057,%ecx
   0x565563c2 <+210>:   mov    %edx,(%esp)
   0x565563c5 <+213>:   movl   $0x14,0x4(%esp)
   0x565563cd <+221>:   mov    %ecx,0x8(%esp)
   0x565563d1 <+225>:   mov    %eax,0xc(%esp)
   0x565563d5 <+229>:   call   0xf7dd5ac0 <__GI___snprintf>

snprintf(buffer, 0x14, "%x", atoi_ret);

   0x565563da <+234>:   lea    -0x200(%ebp),%eax
   0x565563e0 <+240>:   lea    0x5655705a,%ecx
   0x565563e6 <+246>:   mov    %ecx,(%esp)
   0x565563e9 <+249>:   mov    %eax,0x4(%esp)
   0x565563ed <+253>:   call   0xf7dd5a90 <__printf>

printf("Your Hex integer: %s\n", buffer);

   0x565563f2 <+258>:   lea    -0x200(%ebp),%ecx
   0x565563f8 <+264>:   mov    %esp,%eax
   0x565563fa <+266>:   mov    %ecx,(%eax)
   0x565563fc <+268>:   call   0xf7e20680 <__strlen_sse2_bsf>
   0x56556401 <+273>:   mov    %eax,-0x204(%ebp)

len = strlen(buffer);

   0x56556407 <+279>:   lea    -0x200(%ebp),%ecx
   0x5655640d <+285>:   mov    -0x204(%ebp),%eax
   0x56556413 <+291>:   mov    %ecx,(%esp)
   0x56556416 <+294>:   mov    %eax,0x4(%esp)
   0x5655641a <+298>:   call   0xf7e1b470 <memfrob>

memfrob(buffer, len);

   0x5655641f <+303>:   lea    -0x200(%ebp),%eax
   0x56556425 <+309>:   mov    %eax,(%esp)
   0x56556428 <+312>:   call   0x56556290 <add_a_space>

add_a_space(buffer);

   0x5655642d <+317>:   lea    -0x200(%ebp),%eax
   0x56556433 <+323>:   mov    (%eax),%eax
   0x56556435 <+325>:   sub    $0x68333a3e,%eax
   0x5655643a <+330>:   setne  %al
   0x5655643d <+333>:   movzbl %al,%eax
   0x56556440 <+336>:   cmp    $0x0,%eax
   0x56556443 <+339>:   jne    0x56556461 <password+369>

if (*((int *)buffer) != 0x68333e3a) jmp;

   0x56556449 <+345>:   lea    0x56557078,%eax
   0x5655644f <+351>:   mov    %eax,(%esp)
   0x56556452 <+354>:   call   0xf7dd5a90 <__printf>
   0x56556457 <+359>:   call   0x56556230 <get_a_shell>
   0x5655645c <+364>:   jmp    0x5655646f <password+383>

printf("Great! you got my password!\n");
get_a_shell();
return;

   0x56556461 <+369>:   lea    0x56557095,%eax
   0x56556467 <+375>:   mov    %eax,(%esp)
   0x5655646a <+378>:   call   0xf7dd5a90 <__printf>

printf("Oh no\n");

   0x5655646f <+383>:   add    $0x218,%esp
   0x56556475 <+389>:   pop    %ebp
   0x56556476 <+390>:   ret

void password() {
  char buffer[0x200]; // ebp_200
  memset(buffer, 0, 0x200);

  scanf("%50s", buffer);
  printf("Your buffer: %s\n", buffer);

  size_t len = strlen(buffer);
  memfrob(buffer, len);
  add_a_space(buffer);
  atoi_ret = atoi(buffer);
  printf("Your Integer: %u\n", atoi_ret)

  snprintf(buffer, 0x14, "%x", atoi_ret);
  printf("Your Hex Integer: %s\n", buffer);

  len = strlen(buffer);
  memfrob(buffer, len)
  add_a_space(buffer);

  if (*((int*) buffer) == 0x68333a3e) {
      printf("Great! you got my password!\n");
      get_a_shell();
  }
  else {
    printf("Oh no\n")
  }
}
