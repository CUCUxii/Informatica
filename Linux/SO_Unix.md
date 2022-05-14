
## Syscalls

Todas las funciones de C (el codigo) son una simplificación de los syscalls. Estos son llamadas al modo kernel (el modo con máximos privilegios sobre el hardware)
Este realiza ciertas acciones sobre lo que se le ha pasado en el modo usaurio (como escribir algo en la pantalla) y devuelve a este el control y el resultado.  
En lenguaje ensamblador se traduce como la insutrrución *SYSCALL* precedido de un MOV de un numero a un registro, el cual actua como 
[indice](https://filippo.io/linux-syscall-table/) de la insutruccón del syscall que se está haciendo.

Ej MOV EAX 1 = write. Se usa en muchas funciones de C como "puts", "printf"

-------------------------------------------------

## Demonios

Un demonio es el resultado de hacer un fork (o clone) a un proceso y matar al padre con un *exit(0)*. Por tanto el programa parece que finaliza pero el hijo
sigue ejecutándose en segundo plano (demonio). Dicho demonio suele tener un UID y GUI (es un usuario sin nombre)

-------------------------------------------------

## Usuarios

Los usuarios en Linux no son nada mas que un número *UserID o UID* al igual que los grupos *Group ID GUID*. Los usuarios no son solo personas que usen el ordenador, sino
demonios (procesos y servicios), aunque pocos tienen una bash (las personas).  
Muchas veces esos usuarios tienen un nombre (visible en el /etc/passwd) pero tambien pueden no tenerlo (en el caso de muchos demonios)

-------------------------------------------------

## Archivos

En Linux existe la política de que *todo son archivos* por lo que por ejemplo si quieres borrar algo puedes moverlo al archivo /dev/null y desaparecerá para siempre.

-------------------------------------------------

## Descriptor de ficheros 

Un descriptor de ficheros es un numero que describe una accion que hace un programa sobre el sistema.
```
ENTRADA ESTANDAR (teclado) "STDIN" -> 0
SALIDA ESTANDAR (pantalla) "STDOUT" -> 1
ERROR ESTANDAR (pantalla) "STDERR" -> 2
```
Estos son los estandar pero cuando trabajamos con tuberías y demonios se suelen crear más que luego se suelen borrar

