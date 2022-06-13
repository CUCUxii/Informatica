
En este post nos vamos a meter con los retos de "phoenix" de exploit education, a diferencia de protostar, hablamos de 64 bits, el estandar actual.

## Stack-Zero

El stack Zero como reto inicial, no porpone ninguna dificultad, ya que simplemente se trata de desbordar el buffer metiendole demasaida "basura"
al programa.

Para profundizar más en el lenguaje a bajo nivel, bamos a desensamblar este bianrio. Vamo a analizarlo por partes.

```
(gdb) disas main
Dump of assembler code for function main:
   0x00000000004005dd <+0>:     push   rbp
   0x00000000004005de <+1>:     mov    rbp,rsp
   0x00000000004005e1 <+4>:     sub    rsp,0x60   # Preparar una pila de 0x60 de tamaño, desde rsp
   0x00000000004005e5 <+8>:     mov    DWORD PTR [rbp-0x54],edi
   0x00000000004005e8 <+11>:    mov    QWORD PTR [rbp-0x60],rsi
  ----------------------------------------------------------------------
   0x00000000004005ec <+15>:    mov    edi,0x400680 
   0x00000000004005f1 <+20>:    call   0x400440 <puts@plt> 
#### (gdb) x/s 0x400680   # Metemos el edi el argumnto de puts, segun calling conventions.
0x400680:       "Welcome to phoenix/stack-zero, brought to you by https://exploit.education"   
  ---------------------------------------------------------------------- 
   0x00000000004005f6 <+25>:    mov    DWORD PTR [rbp-0x10],0x0
#### (gdb) x/x $rbp-0x10    # O sea aqui tenemos la famosa varaible "changme" que hay que rellenar
0x7fffffffe520: 0x00
  ---------------------------------------------------------------------- 
   0x00000000004005fd <+32>:    lea    rax,[rbp-0x50]
   0x0000000000400601 <+36>:    mov    rdi,rax
   0x0000000000400604 <+39>:    call   0x400430 <gets@plt>
#### (gdb) x/s $rbp-0x50    # Con las calling conventions metemos en rdi el argumento de gets, o sea nuestro input.
0x7fffffffe4e0: 'A' <repeats 11 times>   
  ---------------------------------------------------------------------- 
   0x0000000000400609 <+44>:    mov    eax,DWORD PTR [rbp-0x10]
   0x000000000040060c <+47>:    test   eax,eax     
   0x000000000040060e <+49>:    je     0x40061c <main+63>
   0x0000000000400610 <+51>:    mov    edi,0x4006d0      # Por aqui vamos si hemos hecho bien.
   0x0000000000400615 <+56>:    call   0x400440 <puts@plt>
   0x000000000040061a <+61>:    jmp    0x400626 <main+73>
#### (gdb) x/s 0x4006d0  
0x4006d0:       "Well done, the 'changeme' variable has been changed!"
  ---------------------------------------------------------------------- 
   0x000000000040061c <+63>:    mov    edi,0x400708    
   0x0000000000400621 <+68>:    call   0x400440 <puts@plt>
#### (gdb) x/s 0x400708                                                              
0x400708:       "Uh oh, 'changeme' has not yet been changed. Would you like to try again?" 
  ----------------------------------------------------------------------  
   0x0000000000400626 <+73>:    mov    edi,0x0
   0x000000000040062b <+78>:    call   0x400450 <exit@plt>
   
(gdb) b *0x000000000040062b
(gdb) run
AAAAAAAAAAA
(gdb) x/40x $rsp
0x7fffffffe4d0: 0xffffe588      0x00007fff      0x00000000      0x00000001
0x7fffffffe4e0: 0x41414141      0x41414141      0x00414141      0x00000000
0x7fffffffe4f0: 0x00000000      0x00000000      0x00000000      0x00000000
0x7fffffffe500: 0x00000000      0x00000000      0xffffe588      0x00007fff
0x7fffffffe510: 0x00000001      0x00000000      0xffffe598      0x00007fff
0x7fffffffe520: 0x00000000      0x00000000      0x00000000      0x00000000
```
