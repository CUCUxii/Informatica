


## \[GOT] "Global offset table" y \[PLT] "Procedure Linkage Table"

Escribimos un binario que utiliza funciones de libc (como por ejemplo "printf()") pero no queremos que nuestro binario sea demasiado pesado. 
Entonces lo compilamos normalmente con gcc código.c -o ./bianrio
Este binario ocupaŕá poquito espacio.

Sencillamente porque no tiene el código de las fucniones de libc en sí. sino que ha las tomado del sistema y ha hecho un link(puntero) hacia ellas.

Así que ejecutas esas librerias desde sistema y no del binario (son cosas independientes). 
Pero tanto nuestro bianrio al usarlas, variará según la versíon que estas tengan y si no estan en el sistema, no funcionará.
O puede que el ASLR no exista en nuestro binario pero si en libc... etc.

Estos links a las funciones de libc (sus direcciones) están escritos en la "Global Offset Table". Los escribe en concreto la libreria "ld.so" o *"enlazador dinámico"* una función que busca esas direcciones por todo el sistema para despues meterlas en la GOT (lo hace la primera vez que llamamos al la funcion y despues la ejecuta)

Vamos a verlo en acción.

En el ensamblador, cuando sale una llamada a una función, sale como *"call   0x804839c \<fgets@plt>"*
Es decir, salta a la dirección  "0x80483cc" que está en la tabla .plt (siguientes 3 instrucciones).


```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format4 
(gdb) set disassembly-flavor intel
(gdb) disas main
Instrucciones de main entre ellas, saltar a vuln()
0x0804851a <main+6>:	call   0x80484d2 <vuln>
(gdb) disas vuln
Insutrcciones para crear el stack frame de vuln, alguna varaible y tomar input de usuario para meterlo en un buffer de 520 bytes.
0x080484fa <vuln+40>:	lea    eax,[ebp-0x208]
0x08048500 <vuln+46>:	mov    DWORD PTR [esp],eax
0x08048503 <vuln+49>:	call   0x80483cc <printf@plt>
Esta funcion es en la que nos vamos a centrar, imprimir dicho buffer
Mas instrucciones que ahora no vienen a cuento...
0x0804850f <vuln+61>:	call   0x80483ec <exit@plt>
(gdb) disas 0x80483cc
Esto es la tabla plt, en concreto la parte correspondiente a la funcion printf
0x80483cc <printf@plt>:	jmp    DWORD PTR ds:0x804971c
0x80483d2 <printf@plt+6>:	push   0x20
0x80483d7 <printf@plt+11>:	jmp    0x804837c
(gdb) disas 0x80483ec
La plt de exit, (para ver en que se diferencia de prtinf)
0x80483ec <exit@plt>:	jmp    DWORD PTR ds:0x8049724
0x80483f2 <exit@plt+6>:	push   0x30
0x80483f7 <exit@plt+11>:	jmp    0x804837c
```
----------------------------------------------------------------

# Estructura de la tabla .plt

EL binario, que todavía no sabe donde está la direccion real de la funcion de libc, salta a un sitio que si conoce, la tabla .plt (Function trampoline)
Esta siempre tiene tres insutrcciones por funcion.

1. La primera insutruccion salta a la tabla got, a la parte correspondiente a la funcion a ejecutar, en este caso (exit()). Si quisieramos que ejecutara otra funcion, es el valor que tendriamos que cambiar. 

- Si es la primera vez que se ejecuta, va a la GOT pero lo vuelve a mandar a la PLT, a la siguiente insutrccion \<exit@plt+6>.
- Si es la sgunda, ya no saltara a \<exit@plt+6> sino que ejecutara la funcion de GOT (en la que ya estará escrita su direccion correspondiente)

```console
(gdb) x/i 0x8049724
0x8049724 <_GLOBAL_OFFSET_TABLE_+36>
...
```
2. La segunda insutrccion (plt+6) escribe un valor en la pila. Es el argumento que se le pasara a ld.so, como un indice de funciones, ej, en printf es 0x20 y en exit 0x30. Este buscar tal funcion y la escribira en la GOT.
3. La tercera instrucción siempre es la misma, salta al inicio de la seccion "plt", este ejecuta el ld.so con el indice de antes. (Buscar la direcion, escribirla en la GOT y saltar a la primera insutrccion de exit@plt (o prinf@plt) que como ya tira de una GOT con la direccion buena, funciona)

Esta es la teroia, pero vamos a verla en acción.
Vamos a correr el programa:
```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format4
Break antes de que salga del programa
(gdb) b *0x0804850f  -> llamada a exit()
(gdb) r
Starting program: /opt/protostar/bin/format4 
AAA
AAA
Breakpoint alcanzado!
```

Ahora veremos el prinf (que como el programa ya esta en exit lo ha pasado de sobra, por lo que tendra su direccion en la GOT)
```console
(gdb) x/i 0x80483cc
0x080483cc <printf@plt+0>:	jmp    DWORD PTR ds:0x804971c
(gdb) x/x 0x804971c
0x804971c <_GLOBAL_OFFSET_TABLE_+28>:	0xb7eddf90
(gdb) x/x 0xb7eddf90
0xb7eddf90 <__printf>:	0x53e58955 -> La funcion prtinf, es decir la GOT ya  tiene bien
```
Pero exit al haber hecho el breakpoint antes de que se ejecute no su dicha direccion en la GOT todavía

```console
(gdb) disas 0x80483ec
Dump of assembler code for function exit@plt:
0x080483ec <exit@plt+0>:	jmp    DWORD PTR ds:0x8049724
0x080483f2 <exit@plt+6>:	push   0x30
0x080483f7 <exit@plt+11>:	jmp    0x804837c
End of assembler dump.
(gdb) x/x 0x8049724
0x8049724 <_GLOBAL_OFFSET_TABLE_+36>:	0x080483f2
(gdb) x/2i 0x80483f2
0x80483f2 <exit@plt+6>:	push   0x30
0x80483f7 <exit@plt+11>:	jmp    0x804837c
```
Este sitio donde salta es el principio de plt
```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -d ./format4
...
Disassembly of section .plt:
0804837c <__gmon_start__@plt-0x10>:
804837c:	ff 35 04 97 04 08    	pushl  0x8049704
8048382:	ff 25 08 97 04 08    	jmp    *0x8049708 -> Esta es la instruccion que llama al ld.so
8048388:	00 00                	add    %al,(%eax)
```
Asi que si volvemos con gdb:
```
(gdb) x/x 0x8049708 
0x8049708 <_GLOBAL_OFFSET_TABLE_+8>:	0xb7ff6200
(gdb) x/i 0xb7ff6200
0xb7ff6200 <_dl_runtime_resolve> -> El ld.so
```

Podemos cambiar la direccion de la funcion de GOT por otra que nos interese (format4)
Aunque cambie la direccion de libc por el ASLR, en la tabla GOT es fija, asi que si se saca de ahí, tenemos la vida resulta. (pero eso es para 
otro articulo) 

*Fuente: Live Overflow, trastear con lo aprendido de eĺ con ./format4*










