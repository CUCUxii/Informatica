
Libreria -> #include<unistd.h>
Las funciones syscall tienen parametros, pero no siempre hay que escribir todos. 

Los descriptores de archivo se asignan a archivos abiertos, tuberias y demonios para hacer sobre ellos acciones como close, read... pasandoles el descriptor 
correspondiente ej -> Si habiamos abierto un archivo al que le habíamos asignado el desc. "3" -> open(...) = 3. Para escribir en él write(3,...) y cerrarlo close(3)


 - **write** -> write(descriptor_de_fichero, "Texto", tamaño_bytes_texto)  ->  write(1, "Hola Mundo", 10);
 - **read** -> read(desc_de_fichero, "Texto", tamaño_bytes_texto)  ->  read(0, "Hola Mundo", 10);
 
 
 ----------------------------------------------------------------
 
 ## ARCHIVOS
 
  - **open** -> open("ruta/archivo", flags, modo) = desc_de_fichero -> [MAN](https://man7.org/linux/man-pages/man2/open.2.html)
 ```
 # FLAGS
   O_RDWR -> leer/escribir   O_WRONLY -> escritura   O_RDONLY -> lectura 
   O_CREAT -> crearlo si no existe, O_APPEND -> añadir cosas 
 # PRIVILEGIOS
   S_ + R/W/X + USR/GRP/OTH -> leer escribir ejecutar   usuario grupo otros. Tiene un codigo numerico
 ```
 - **close** -> close(descpt.). Se puede usar para tuberias, archivos, sockets y demonios.
 - **dup** -> dup(descpt., descpt.) -> 
dup(4, 1) -> que el stdout vaya al descriptor 4 (por ejemplo asociado a un archivo) es decir escribir en ese archivo y no en pantalla
Ejemplo ->  ```bash cat archivo.txt > /dev/null``` -> ```C open("/dev/null", O_RDWR) = 4; dup2(4,0); dup2(4,1); dup2(4,2); close(4)```

----------------------------------------------------------------
 
 ## SOCKETS Y CONEXIONES 
 
  - **socket** -> crear un socket   socket(tipo, modo) = descpt.   -> socket(AF_INET, SOCK_STREAM) = 5    socket ipv4 en el descpt. 5
ipv4 suele ser AF_INET
  - **setsockopt** -> defimir caracteristicas del socket
  - **bind** -> darle al socket sus puerto/ip -> bind(descpt., {sa_family=tipo, sin_port=htons(puerto), sin_addr("ip")}, bytes_que_acepta)
  - **listen** -> ponerlo en escucha ->  listen(descpt)

 ```C
socket(AF_INET, SOCK_STREAM) = 5;
bind(5, {sa_family=AF_INET, sin_port=htons(443), sin_addr("127.0.0.1")}, 16)
listen(5)
 ```
 
----------------------------------------------------------------
 
 ## FORKS -> crear procesos hijo.

Un proceso se duplica, saliendo un padre y un hijo. El padre es el proceso original mientras que el hijo es su derivado. Si un padre se cierra, el hijo queda
huerfano, conviertiendose en padre si se vuelve a duplicar.   
Un buen ejemplo de esto es el proceso padre "shell" y el hijo cualquier otro proceso que corra a partir de la shell.

  - **clone()**
  - **fork()**

Las funciones fork y clone suelen tener como argumento el PID del hijo resultante (o su descriptor de fichero () = desc. aunque se usa mas el PID)
 
  ```C
open("/opt/protostar/run/net0.pid", O_RDWR|O_CREAT|O_TRUNC, 0700) = 3
clone(3) = 404
codigo que hace el hijo, ejemplo va en incognito por lo que redirige todo al /dev/null
 ```
 
