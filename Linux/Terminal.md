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

---------------------------------------------------------------------------
## Lo Básico: Las rutas (carpetas)

 * **Ruta absoluta** →  /home/usuario/escritorio/carpeta_1 ->  Siempre es la misma
 * **Ruta relativa** →  ./carpeta  Depende del contexto, de donde estés actualmente (En este ejemplo, te refieres a una carpeta que estña en el escritorio)
  Por tanto hacemos lo mismo con *cat ./archivo.txt* que con *cat /home/usuario/escritorio/archivo.txt*
    - *Inciso, explicaré como funcionan las rutas relativas y ciertos riesgos de seguridad asociados.*

	. carpeta actual      .. carpeta padre (la que contiene a la actual)   ~ /home/usuario    - directorio anterior
  
## Movimiento: desplazamiento entre directorios (carpetas)
-  **PWD** -> comando para que el sistema te diga en que directorio estás 
-  **LS** -> Listar las carpetas que hay dentro del directorios
     * ls -l -> que te salga en forma de lista, información mas detallada
     * -a -> Que te salgan tambien los archivos/carpetas ocultas

	```console
	[usuario@linux]-[~/escritorio]:$ pwd
	/home/usuario/escritorio
	[usuario@linux]-[~/escritorio]:$ ls
	programas  Archivo.txt
	```
-  **CD** -> para cambiar de directorio
	```console
	[usuario@linux]-[~/escritorio]:$ pwd
	/home/usuario/escritorio
	[usuario@linux]-[~/escritorio]:$ cd ../
	escritorio   musica   documentos   descargas  
	[usuario@linux]-[~]:$ cd ./musica
	[usuario@linux]-[~/musica]:$ ls
	nirvana   depeche_mode   metallica
    ```
## Crear/borrar/mover carpetas y archivos
- **TOUCH** -> Crear documentos vacios  \[touch ruta/archivo]
- **RM** -> borrar archivos \[rm ruta/archivo]
     * rm -r ./Carpeta → se borra la carpeta y su contenido (recursivo)
     * rm -f * → forzar, para archivos que te pregunta todo el rato como los .git
- **MKDIR** -> Crear una nueva carpeta 
- **RMDIR** -> Para borrar carpetas vacias, pero mejor usar \[rm -rf]

	```console
	[usuario@linux]-[~/escritorio]:$ ls
	programas  Archivo.txt
	[usuario@linux]-[~/escritorio]:$ mkdir ./Carpeta
	programas  Archivo.txt  Carpeta  
	[usuario@linux]-[~]:$ cd ./Carpeta
	[usuario@linux]-[~/musica]:$ touch Ejemplo.txt
	[usuario@linux]-[~/musica]:$ ls 
	Ejemplo.txt
	[usuario@linux]-[~/musica]:$ cd ../
	[usuario@linux]-[~/musica]:$ rm -rf ./Carpeta
	[usuario@linux]-[~/musica]:$ ls
	programas  Archivo.txt
    ```
## Mover y copiar
-  **MV** → mover -> vale con carpetas (movera tambien su contenido) o archivos. \[mv ruta/fichero ruta]
     * mv fichero.txt ruta/ → Meter fichero de directorio actual a otra carpeta
     * mv ruta/archivo . → Mover fichero de una carpeta al directorio actual.
     * mv nombre.txt nuevo_nombre.txt → renombra el archivo (si no existe el segundo, si existe lo sobreescribe)
     * mv /* /dev/null → *mueve toda la raiz al "dev null" y manda el sistema operativo a tomar por culo*
	```console
	[usuario@linux]-[~/escritorio]:$ mv ../musica/nirvana .
	[usuario@linux]-[~/escritorio]:$ ls
	programas  Archivo.txt Nirvana
	[usuario@linux]-[~/escritorio]:$ rm -rf ./nivana
    ```
   		

   
   
   
   
