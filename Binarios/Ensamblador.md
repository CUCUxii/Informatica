
## Instrucciones (más basico)

 - **MOV** -> MOV(destino, origen) -> mueve los datos de un sitio a otro (en general registros)
```
x32 -> mov    [esp+0x08], 0x3   -> mete un "3" en la pila (offset 08 ya que es esp+08    esp=0000 0000 0003 0000)
x64 -> mov    rdi,rsp     -> mueve lo del registro rsp(inicio de la pila) al rdi(para convertirlo en el argumento de en una proxima funcion)
```
 - **LEA** -> LEA(destino, origen) -> mueve punteros (las direcciones no los datos) de un sitio a otro.
 - **PUSH** -> mete el ultimo valor en la pila
 - **POP** -> POP(destino) mueve el ultimo valor de la pila en tal registro.
 - **JMP** -> Saltar.



## Operaciones comnues 

 - **Crear marco de pila** 
```
0x0000555555555168 <+0>:     push   rbp       Guarda rbp en la pila (el rbp de la antigua funcion)
0x0000555555555169 <+1>:     mov    rbp,rsp   Mueve rsp a rbp (Por lo que estan pegados)
0x000055555555516c <+4>:     sub    rsp,0x60  Sustrae x60 (96) de rsp, por lo que el stack frame ocupa ese espacio
(gdb) p/x $rsp
$2 = 0x7fffffffdf10
(gdb) p/x $rbp
$3 = 0x7fffffffdf70
pwndbg> p/x $rbp - $rsp
$5 = 0x60    
```
Mas información sobre los marcos de pila [aqui](https://github.com/CUCUxii/CUCUxii.github.io/blob/main/Binarios/Estructura%20de%20un%20binario.md#la-pila)

 - **Comparaciones** -> Programa que te pide una contraseña "Wintermute" y si la pones mal te dice "Contraseña incorrecta"
```
0x000055555555519a <+69>: lea  rax,[rip+0xe67]  # 0x555555556008       # Mete en rax el puntero rip+0xe67 = 0x555555556008
0x00005555555551a1 <+76>: cmp  rdx,rax       # Compara lo que hay en rdx (input del usario) con lo del puntero
0x00005555555551a4 <+79>: jne  0x5555555551b4 <vuln+95>   # Si la contraseña no es correcta te imprime en vuln95 que no es correcta.

pwndbg> x/s 0x555555556008
0x555555556008: "Wintermute"    # Wintermute es la palabra clave que el usuario debe introducir (variable target) para que el programa siga correctamente
```
En radare2 se puede ver mas claramente. Aqui se simplifican ademas las direeciones de memoria 
```
0x0000119a      488d05670e00.  lea rax, str.Wintermute; 0x2008 ; "Wintermute"
0x000011a1      4839c2         cmp rdx, rax
0x000011a4      750e           jne 0x11b4
```
 - **Pasar argumentos a una funcion**
Normalmente se suele usar punteros hacia las strings y trasladar su contenido con lea, a rdx u otro registro para efectuar una funcion.
Las funciones toman sus argumentos de los registros siguiendo las calling conventions.

```
0x00005555555551a6 <+81>:    lea    rdi,[rip+0xe6b]        # 0x555555556018
0x00005555555551ad <+88>:    call   0x555555555030 <puts@plt>
pwndbg> x/s 0x555555556018
0x555555556018: "you have modified the target :)"   # O sea esta string se la hemos pasado a puts
> puts("you have modified the target :)")
```

------------------------------------------------------------------------------------------

Mas ejemplos de como los dos programas nos muestran el ensamblador
```
#RADARE2
0x000015ad      488d3d190b00.  lea rdi, str.___activated_license:__s_n ; 0x20cd ; "[+] activated license: %s\n" ; const char *format
#GDB
0x00005555555555ad <+624>:   lea    rdi,[rip+0xb19]        # 0x5555555560cd 
pwndbg> x/s 0x5555555560cd
0x5555555560cd: "[+] activated license: %s\n"
```
