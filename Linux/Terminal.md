# Linux (by CUCUxii)

## Índice 
- [Que es la Terminal](#Inicio)





---------------------------------------------------------------------------
## ¿Que es la Terminal?

La Terminal o shell es la linea de comandos, la consola. Normalmente estamos acostumbrados a operar un sistema operativo mediante una interfaz gráfica (iconos,
desplazamiento a base de clicks) ya que estas se inventaron para hacer la informática mas accesible, ya que a mucha gente ver una terminal de letras verdes
sobre fondo negro le eche para atras, piense que es algo muy complicado o algo de *hackers*

Yo aprendi a utilizar la terminal y me acabó encantando, tiene muchas mas posibilidadades que una interfaz gráfica (en mi Linux he eliminado esta última,
reduciendolo todo a la consola).

Antes de empezar, quiero aclarar que en informática a las carpetas las llamamos **directorios** y a los archivos **ficheros**.

Los comandos son programas y como tales, te dan una respuesta. Es decir, una salida a una entrada.
      \[comando] -> \[respuesta]       whoami -> usuario

Algunos se les puede pasar un input: Ej leer:  que archivo quieres leer (que se le pasa mediante las rutas que a continuación explicaré)
 ```console
    cat ./archivo.txt
 ```
---------------------------------------------------------------------------
## Lo Básico: Las rutas (carpetas)

 * **Ruta absoluta** →  /home/usuario/escritorio/carpeta_1 ->  Siempre es la misma
 * **Ruta relativa** →  ./carpeta  Depende del contexto, de donde estés actualmente (En este ejemplo, te refieres a una carpeta que estña en el escritorio)
  Por tanto hacemos lo mismo con *cat ./archivo.txt* que con *cat /home/usuario/escritorio/archivo.txt*
    - *Inciso, explicaré como funcionan las rutas relativas y ciertos riesgos de seguridad asociados.*

	. carpeta actual      .. carpeta padre (la que contiene a la actual)   ~ /home/usuario    - directorio anterior
  
   ### Movimiento: desplazamiento entre directorios (carpetas)
   - **PWD** -> comando para que el sistema te diga en que directorio estás 
   - **LS** -> Listar las carpetas que hay dentro del directorios
          * ls -l -> que te salga en forma de lista, información mas detallada
          * -a -> Que te salgan tambien los archivos/carpetas ocultas
	```console
	[usuario@linux]-[~/escritorio]:$ pwd
	/home/usuario/escritorio
	[usuario@linux]-[~/escritorio]:$ ls
	programas  Archivo.txt
        ```
   - **CD** -> para cambiar de directorio
       ```console
       [usuario@linux]-[~/escritorio]:$ pwd
       /home/usuario/escritorio
       [usuario@linux]-[~/escritorio]:$ cd ../
       escritorio   musica   documentos   descargas  
       [usuario@linux]-[~]:$ cd ./musica
       [usuario@linux]-[~/musica]:$ ls
       nirvana   depeche_mode   metallica
       ```
   
   
