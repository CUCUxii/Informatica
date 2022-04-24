


## \[GOT] "Global offset table" y \[PLT] "Procedure Linkage Table"

Escribimos un binario que utiliza funciones de libc (como por ejemplo "printf()") pero no queremos que nuestro binario sea demasiado pesado. 
Entonces lo compilamos normalmente con gcc código.c -o ./bianrio
Este binario ocupaŕá poquito espacio.

Sencillamente porque no tiene el código de las fucniones de libc en sí. sino que ha las tomado del sistema y ha hecho un link(puntero) hacia ellas.

Así que ejecutas esas librerias desde sistema y no del binario (son cosas independientes). 
Pero tanto nuestro bianrio al usarlas, variará según la versíon que estas tengan y si no estan en el sistema, no funcionará.
O puede que el ASLR no exista en nuestro binario pero si en libc... etc.

Estos links a las funciones de libc están escritos en la "Global Offset Table". Vamos a verlo en acción.

En el ensamblador, cuando sale una llamada a una función, sale como *"call   0x804839c \<fgets@plt>"*
Es decir, salta a la dirección  "0x80483cc" que está en la tabla .plt (siguientes 3 instrucciones). EL binario, que todavía no sabe donde
está la direccion real de la funcion de libc, salta a un sitio que si conoce, la tabla .plt.

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
(gdb) x/3i 0x80483cc
ESto es la tabla plt, la sección de nuestra funcion printf
0x80483cc <printf@plt>:	jmp    DWORD PTR ds:0x804971c
0x80483d2 <printf@plt+6>:	push   0x20
0x80483d7 <printf@plt+11>:	jmp    0x804837c
```
¿Que hay en la .plt?
1. Instrucción para que saltemos otra vez, a 0x804971c. Exactamente a la tabla GOT, la parte de prtinf donde está su direccion real (prinf@got)
2. Tambien una insutrccion push con un numero no muy grande 
3. Y otro salto a otra direeción, que si vemos el .plt de otra funcion, ejemplo exit@plt es la misma.

```console
(gdb) x/3i 0x80483ec
0x80483ec <exit@plt>:	jmp    DWORD PTR ds:0x8049724
0x80483f2 <exit@plt+6>:	push   0x30
0x80483f7 <exit@plt+11>:	jmp    0x804837c
```
Esta ultima funcion (ld.so) es la que se encarga de encontrar en el sistema la dirección real de la función de libc y darsela a la tabla GOT, 
el push de antes es un parametro para dicha funcion, y actua como un indice, es decir, si es 0x20 es printf() y si es 0x30 es exit(). Despues
de darsela al GOT, se ejecuta y despues, vuelve a nuestra ducnnion principal (en este caso "vuln").
La segunda vez que se ejecute un prtinf() como ya está su direccion en la GOT, no hace falta llamar otra vez a ld.so

Podemos cambiar la direccion de la funcion de GOT por otra que nos interese (format4)

Aunque cambie la direccion de libc por el ASLR, en la tabla GOT es fija, asi que si se saca de ahí, tenemos la vida resulta. (pero eso es para 
otro articulo) 

Fuente: Live Overflow, trastear con lo aprendido de eĺ con ./format4




