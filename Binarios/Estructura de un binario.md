### \[Índice]

 - [CODIGO seccion .text](#codigo-seccion-.text)
 - [Global offset table](#global-offset-table)
 - [Procedure Linkage Table](#procedure-linkage-table)
 - [La pila](#la-pila)


----------------------------------------------------------------

# CODIGO seccion .text

El código son las instrucciones en ensamblador del programa, estas vienen de su código original en C, solo que pasadas a lenguaje máquina.
No se pueden modificar, es decir solo "READ_ONLY" o solo lectura. Tampoco son lineales, gracias a los bucles y comparaciones. 
user@protostar:/opt/protostar/bin$ objdump -t ./format4 | grep "text"
```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -t ./format4 | grep "text"
08048400 g     F .text	00000000              _start
080484b4 g     F .text	0000001e              hello
080484d2 g     F .text	00000042              vuln
08048514 g     F .text	0000000f              main 
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format4
(gdb) set disassembly-flavor intel
(gdb) disas main
0x08048514 <main+0>:	push   ebp
0x08048515 <main+1>:	mov    ebp,esp
0x08048517 <main+3>:	and    esp,0xfffffff0
0x0804851a <main+6>:	call   0x80484d2 <vuln>
0x0804851f <main+11>:	mov    esp,ebp
0x08048521 <main+13>:	pop    ebp
0x08048522 <main+14>:	ret    
```

----------------------------------------------------------------

# Global offset table 

**\[GOT]**

Escribimos un binario que utiliza funciones de libc (como por ejemplo "printf()") pero no queremos que nuestro binario sea demasiado pesado. 
Entonces lo compilamos normalmente con  gcc ->    ```console gcc código.c -o ./binario```

Este binario ocupaŕá poquito espacio sencillamente porque no tiene el código de todas las funciones de libc en sí, sino que tiene un link o puntero a las funciones albergadas en el sistema en que se ejecuta.

Así que ejecutas esas librerias desde sistema y no del binario (son cosas independientes). 
Pero tanto nuestro bianrio al usarlas, variará según la versíon que estas tengan y si no estan en el sistema, no funcionará.
O puede que el ASLR no exista en nuestro binario pero si en libc... etc.

Estos links a las funciones de libc (sus direcciones) están escritos en la "Global Offset Table". Los escribe en concreto la libreria "ld.so" o *"enlazador dinámico"* una función que busca esas direcciones por todo el sistema para despues meterlas en la GOT (lo hace la primera vez que llamamos al la funcion y despues las ejecuta).

Vamos a verlo en acción.

En el ensamblador, cuando sale una llamada a una función, sale como *"call   0x804839c \<fgets@plt>"*
Es decir, salta a la dirección  "0x80483cc" que está en la tabla .plt (siguientes 3 instrucciones). Esta la vamos a ver en la siguiente sección muy detalladamente.


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

(gdb) disas 0x80483cc -> printf@plt
0x80483cc <printf@plt>:	jmp    DWORD PTR ds:0x804971c
0x80483d2 <printf@plt+6>:	push   0x20
0x80483d7 <printf@plt+11>:	jmp    0x804837c

(gdb) disas 0x80483ec -> exit@plt
0x80483ec <exit@plt>:	jmp    DWORD PTR ds:0x8049724
0x80483f2 <exit@plt+6>:	push   0x30
0x80483f7 <exit@plt+11>:	jmp    0x804837c
```
----------------------------------------------------------------

# Procedure Linkage Table

**\[PLT]**

EL binario, que todavía no sabe donde está la direccion real de la funcion de libc, salta a un sitio que si conoce, la tabla .plt (Function trampoline)
Esta siempre tiene tres insutrcciones por funcion.

1. La primera insutruccion alberga una direccion de memoria, de la siguiente función a ejecutar (puntero de funcion)
 Si quisieramos que ejecutara otra funcion, tendríamos que cambiar el valor que guarda dicha dirección
- Si es la primera vez que se ejecuta, la direccion de la insturcciíon que esta guardada te vuelve a mandar a la PLT (\<exit@plt+6>).
- Si es la sgunda, ya no saltara a \<exit@plt+6> sino que te mandará a la entrada de la GOT en la que ya estará escrita su direccion correspondiente.

2. La segunda insutrccion (plt+6) escribe un valor en la pila. Es el argumento que se le pasara a ld.so, como un indice de funciones, (ej: en printf es 0x20 y en exit 0x30) Este se basará en este índice a la hora de buscar la función a escribir en GOT.
3. La tercera instrucción siempre es la misma, salta al inicio de la seccion "plt", este ejecuta el ld.so con el indice de antes. (Buscar la direcion, escribirla en la GOT y saltar a la primera insutrccion de exit@plt (o prinf@plt) que como ya tira de una GOT con la direccion buena, funciona)

Esta es la teoría, puede resultar algo confusa asi que vamos a verla en acción.
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
(gdb) x/i 0x80483cc -> Primera entrada de la plt con el puntero
0x080483cc <printf@plt+0>:	jmp    DWORD PTR ds:0x804971c -> Puntero (memoria que guarda la direccion de la GOT)
(gdb) x/x 0x804971c
0x804971c <_GLOBAL_OFFSET_TABLE_+28>:	0xb7eddf90 -> Dirección de la función "printf" en la GOT.
(gdb) x/x 0xb7eddf90
0xb7eddf90 <__printf>:	0x53e58955 -> Código de la funcion prtinf, es decir la GOT ya  tiene bien
```
Pero exit al haber hecho el breakpoint antes de que se ejecute (no está su dicha direccion escrita en la GOT todavía)

```console
(gdb) disas 0x80483ec  -> Primera entrada de la plt de exit
Dump of assembler code for function exit@plt:
0x080483ec <exit@plt+0>:	jmp    DWORD PTR ds:0x8049724 -> Puntero, como con printf 
0x080483f2 <exit@plt+6>:	push   0x30
0x080483f7 <exit@plt+11>:	jmp    0x804837c
End of assembler dump.
(gdb) x/x 0x8049724 -> ¿Adónde nos mandará el puntero esta vez?
0x8049724 <_GLOBAL_OFFSET_TABLE_+36>:	0x080483f2 -> Curiosamente como no existe todavía la direccion en la GOT nos manda otra vez a plt.
(gdb) x/2i 0x80483f2
0x80483f2 <exit@plt+6>:	push   0x30
0x80483f7 <exit@plt+11>:	jmp    0x804837c -> Salta a esta dirección...
```
Este sitio donde salta es el principio de plt
```console
[user@protostar]-[/opt/protostar/bin]:$ objdump -d ./format4
...
Disassembly of section .plt:
0804837c <__gmon_start__@plt-0x10>:   -> Resulta ser el inicio de plt
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

Como ya hemos dicho en la primera entrada de plt tenemos guardado un puntero, este apunta a la entrada en la GOT o de vuelta a plt para llamar al ld.so, si sobreescribimos ese puntero con la dirección de una función que nos interese, podemos manipular el programa a nuestro antojo.
Aunque cambie la direccion de libc por el ASLR, en la tabla GOT es fija, asi que si se saca de ahí, tenemos la vida resulta. (pero eso es para 
otro articulo) 

*Fuente: Live Overflow, trastear con lo aprendido de eĺ con ./format4*

----------------------------------------------------------------

# La pila

La pila es una memoria, es dónde un programa almacena varaibles, argumentos y más cosas que necestia para funcionar. 

Esta se crea con insutrcciones específicas llamadas "function prolog" y se llama "stack frame", cada función tiene la propia
Pero antes de eso quiero explicar el asunto de los registros para que se entienda.
 - EBP -> El registro ebp es el "frame base pointer", es un registro que se usa principalmente como referencia a la hora de llamar varaibles, por eejemplo \[ebp - 0x4]. 
 - ESP -> es el inicio de la pila. 
 - EIP -> puntero de insutrcción, almacena la dirección de la siguiente instrucción a ejecutar, 
 
```console
(gdb) b *0x08048425 Al poner el breakpoint todavia no se ha ejecutado esta insutrucción...
(gdb) x/i $eip
0x8048425 <main+9>:
 ```
Si se hace una llamada a una funcion con call o jmp ```call   0x80483f4 <vuln>``` se pone ese número (0x80483f4) en el eip y asi saltara a "vuln". 


Vamos a ver como se crea el stack frame de [./format1](https://exploit.education/protostar/format-one/)

```console
[user@protostar]-[/opt/protostar/bin]:$ gdb ./format1
(gdb) set disassembly-flavor intel
(gdb) disas main
0x0804841c <main+0>:	push   ebp   -> Guarda ebp en la pila -> $esp = 0xbffff6a8:	0xbffff728 (dirección de EBP)
0x0804841d <main+1>:	mov    ebp,esp -> Pon esp donde está ebp ->  $ebp = 0xbffff6a8:	0xbffff728 Igual
0x0804841f <main+3>:	and    esp,0xfffffff0 -> Alinea la pila, esto no importa mucho.
0x08048422 <main+6>:	sub    esp,0x10 -> Resta a esp 0x10 para crear el marco de pila -> 0xbffff690 (0xbffff6a8 - 0x10) 
```
La pila de "main" va de $esp = 0xbffff690  a  $ebp = 0xbffff6a8  (16)
> *\[esp =  0xbffff690 <- stack frame de vuln()]  \[ebp 0xbffff6a8: 	0xbffff728] \[retorno -> acabar el programa]*
>> El retorno (despues del ebp), es a donde saltará una vez acabada la funcion (direccion que pondra en eip)

Entramos en la función "vuln"

```console
0x08048430 <main+20>:	call   0x80483f4 <vuln> -> PUSH eip (guarda en la pila el eip de main +25), POP eip,0x80483f4 
0x08048435 <main+25>:	leave 
(gdb) disas vuln
0x080483f4 <vuln+0>:	push   ebp -> Lo mismo de antes; guarda el antiguo ebp en la pila, pero antes de la de main -> $esp =  0xbffff688: 0xbffff6a8
0x080483f5 <vuln+1>:	mov    ebp,esp -> Mete esp en ebp. Tanto esp como ebp ahora son = 0xbffff688  (8 bytes antes de main)
0x080483f7 <vuln+3>:	sub    esp,0x18 -> Restale 0x18 a esp para crear el marco de pila->  0xbffff670 (0xbffff688 - 0x18) 
```

La pila de vuln empieza en  0xbffff66c (0xbffff690 - 0x18) O sea es 18 direcciones antes que el de main.
> *\[esp = 0xbffff670:  stack frame de vuln()]  \[ebp = 0xbffff688:0xbffff6a8]  \[retorno -> 0xbffff68c: eip de main+25 = 0x08048435]*

```console
(gbd) b *vuln
(gbd) run
(gdb) x/40x $esp
0xbffff670:	0xb7ff1040	0x0804960c	0xbffff6a8	0x08048469 -> Pila de vuln
0xbffff680:	0xb7fd8304	0xb7fd7ff4	0xbffff6a8	0x08048435
0xbffff690:	0xbffff8ae	0xb7ff1040	0x0804845b	0xb7fd7ff4 -> Pila de main
0xbffff6a0:	0x08048450	0x00000000	0xbffff728	0xb7eadc76
(gdb) x/i 0x08048435
0x8048435 <main+25>:	leave
```

Cuando ejecute **leave**: eliminara el stack frame de vuln para varaibles, restaurara el ebp (pop ebp ->  0xbffff6a8) y volvera a la siguiente insutruccion de main (pop eip -> 0x08048435).
Ahora main vuelve con su stack original (0xbffff690 a 0xbffff6a8 (16)) Pero como la siguiente insutrccion de main es otro leave, adios al stack de main tambien.










