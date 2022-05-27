# Linux (by CUCUxii)

## Índice 
- [Que es la Terminal](#Inicio)


---------------------------------------------------------------------------

## ¿Que es la Terminal?

La Terminal o shell es la linea de comandos, la consola. Normalmente estamos acostumbrados a operar un sistema operativo mediante una interfaz gráfica (iconos, desplazamiento a base de clicks) ya que estas se inventaron para hacer la informática mas accesible, ya que a mucha gente ver una terminal de letras verdes sobre fondo negro le eche para atras, piense que es algo muy complicado o algo de *hackers*  
Yo aprendi a utilizar la terminal y me acabó encantando, tiene muchas mas posibilidadades que una interfaz gráfica (en mi Linux he eliminado esta última,
reduciendolo todo a la consola).  
Antes de empezar, quiero aclarar que en informática a las carpetas las llamamos **directorios** y a los archivos **ficheros**.
Los comandos son programas y como tales, te dan una respuesta. Es decir, una salida a una entrada.
      \[comando] -> \[respuesta]       whoami -> usuario
      
- Algunos se les puede pasar un input: Ej
```console
 [usuario@linux]-[~/escritorio]:$ cat ./archivo.txt
```

---------------------------------------------------------------------------

## Lo Básico: Las rutas (carpetas)

 * **Ruta absoluta** →  /home/usuario/escritorio/carpeta_1 ->  Siempre es la misma
 * **Ruta relativa** →  ./carpeta  Depende del contexto, de donde estés actualmente (En este ejemplo, te refieres a una carpeta que estña en el escritorio)
  Por tanto hacemos lo mismo con *cat ./archivo.txt* que con *cat /home/usuario/escritorio/archivo.txt*
    - *Inciso, explicaré como funcionan las rutas relativas y ciertos riesgos de seguridad asociados.*

	. carpeta actual      .. carpeta padre (la que contiene a la actual)   ~ /home/usuario    - directorio anterior
	
---------------------------------------------------------------------------

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
[usuario@linux]-[~/escritorio]:$ cd ../; ls
escritorio   musica   documentos   descargas  
[usuario@linux]-[~]:$ cd ./musica; ls
nirvana   depeche_mode   metallica
```
---------------------------------------------------------------------------

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
[usuario@linux]-[~/escritorio]:$ mkdir ./Carpeta; ls
programas  Archivo.txt  Carpeta  
[usuario@linux]-[~]:$ cd ./Carpeta; touch Ejemplo.txt; ls
Ejemplo.txt
[usuario@linux]-[~/musica]:$ cd ../; rm -rf ./Carpeta; ls
programas  Archivo.txt
```
---------------------------------------------------------------------------

## Mover y copiar
-  **MV** → mover -> vale con carpetas (movera tambien su contenido) o archivos. \[mv ruta/fichero ruta]
     * mv fichero.txt ruta/ → Meter fichero de directorio actual a otra carpeta
     * mv ruta/archivo . → Mover fichero de una carpeta al directorio actual.
     * mv nombre.txt nuevo_nombre.txt → renombra el archivo (si no existe el segundo, si existe lo sobreescribe)
     * mv /* /dev/null → *mueve toda la raiz al "dev null" y manda el sistema operativo a tomar por culo*
```console
[usuario@linux]-[~/escritorio]:$ mv ../musica/nirvana .; ls
programas  Archivo.txt Nirvana
[usuario@linux]-[~/escritorio]:$ rm -rf ./nivana; ls
programas  Archivo.txt
```
-  **CP** -> copiar. Al contrario de mover, el archivo original no desaparece, sino que se duplica.
     * cp ruta/documento nueva_ruta/ → haz una copia del fichero en otra carpeta
     * cp ruta/documento nueva_ruta/ nuevo_nombre → Que la copia tenga otro nombre diferente.
     * cp -r ruta/ nueva_ruta/ → si es una carpeta (con cosas dentro) -r de recursividad

---------------------------------------------------------------------------   	

## Leer, escribir
-  **CAT**  -> mostrar el contenido el documento en la terminal →  cat documentoX.txt → “Hola”
```console
[usuario@linux]-[~/escritorio]:$ cat Archivo.txt
Hola Mundo
```
-  **NANO**  -> Editor de texto → nano fichero.txt. Si el archivo no existía lo crea.
     * Ctrl+O -> Guardar
     * Ctrl+X -> salir 
-  **VIM**  -> como nano pero mas sofisticado
     * I -> modo edición ||  * Esc → salir a modo normal ||
     * : -> comandos ->  :q! Salir sin guardar || :w → guardar || :wq! → salir guardando || :%s/palabra/replace/g
     * Copiar Pegar ->  dd  → cortar linea (d2d 2 ls.) || yy  → copiar || p → pegar || o → nueva linea || u → deshacer  || ctrol r → rehacer
     * v → modo visual  (se selecciona con los controles) -> *	d → cortar || y → copiar || p -> pegar
     * Moverse -> gg → irse al principio || G → irse al final
     * / buscar palabra
	Borrar/sutituir:
			Sustituir → :%s/palabra/replace/g
			Borrar (ej, todas las lienas comentadas) → :g/^\#/d    Borrar lineas vacias → :g/^$/d
			Cambiar “\\n” por saltos de linea →   %s/\\\\n/\r/g
-  **ECHO** -> 	imprime un mensaje en la terminal →   echo”Hola Mundo”		
     * echo -e”\nHola Mundo”  \n es un salto de linea y -e es para que lo interprete en vez de imprimirlo.
-  **READ** -> Suele ser para leer el input de un usuario pero si esta pipeado lee de un archivo.
-  **SPONGE** -> es como un echo pero no da problemas si lee de un archivo y escribe luego en el mismo
```console
[usuario@linux]-[~/escritorio]:$ cat users.txt | while read user; do echo $user@mail.com; done | sponge users 
Elliot@mail.com
Angela@mail.com
Darlene@mail.com
```
---------------------------------------------------------------------------

## Filtros
-  **GREP** ->  Buscar una palabra en un archivo de texto grep “palabra” ./fichero.txt  → palabra y toda la línea que la siga.
     * -n Indicar en que linea de texto esta la palabra
     * -oP → Regex  →  grep ^f fichero.txt         grep -oP ‘\w{1.10}’
     * Mostrar lineas debajo y encima ademas →  grep “palabra” -A (después) -B(antes) -C(antes y después), grep “palabra” -A 4 → es recomendable | tail -n (después) 
     * -v “palabra” → Quita las lineas donde esta cierta palabra: grep -v “palabra”	  
     *	grep -r -i -E “users|pass|key” → Busca estas palabras dentro de todos los archivos (-r recursivo (carpetas en carpetas) -i (da igual sean mayusculas o minusulas  -E palabras )
     *	grep -r -i “palabra” –text → busca la palabra incluso dentro de binarios y comprimidos	
-  **AWK** -> Filtrar elementos de una linea (columnas)   → Ideal pipearlo con el grep
     * awk `NR==4´ → muestra solo la columna 4 
     * awk 'NF{print $NF}'. Imprime el último argumento  → awk 'NF{print $NF}' el primero
     * awk ‘/palabra/’ filtrar lo que está al lado de “palabra”
     * awk ‘{print $2}’ Imprimeme el segundo argumento.      FS=:  →   delimitador (:)
-  **CUT** → Filtrar partes de una string 
     * cut -d “”-f 3 → -d es delimitador “” lo que hay antes de lo que interesa filtrar -f es la columna
     * Mas de una columna → -f 3,5 (solo la 3 y 5)  -f 3-5 de la tres a la cinco -f 3- todas desde la 3
-  **TAIL** -> Mostrar un archivo de texto a partir de cierta linea:
     * tail -n 2        -n es numero de linea -> -n 2: muestra las últimas dos líneas -n +2 muestra a partir de la línea 2.
-  **HEAD** -> el texto de antes de cierta línea (- n 2: muestra las primeras dos líneas, -n +2 muestra antes de la línea
-  
---------------------------------------------------------------------------

## Busqueda
-  **FIND** ->  Buscar un archivo en el sistema   find ruta -name “encontrar.txt”
     * | xargs cat →  Imprime lo que contenga escrito → “Hola Mundo”
     * -size Xc  → filtra por tamaño -size 1024
     * -type f →  buscar ficheros, (d directorios) (-type f -printf “%f\t%p\t%u\%g\t%m\n t es tabulacion %p (ruta) u(usuario…) -> formateado en una tabla.)
     *	-readable  -executable... filtra ficheros, por si son legibles, ejecutables
     * -user  -group  filtra archivos por quien tiene derecho a abrirlos.
     * -perm /4000 → busqueda per archivos SUID
     * find / -type f -newermt 2017-08-16 ! -newermt 2020-08-16  -ls → busca los archivos 
-  **FILE** -> decir que tipo de archivo es → file ./CarpetaX/* te saca todos los archivos de la carpetaX y te dice que son.
-  **SORT** -> ordenar lineas por orden alfábetico   sort bandas.txt  → ACDC Motorhead Opeth…
     * sort | uniq -u te muestra solo la línea de código que no esté repetida.
-  **WC** -> Contador
     * -l -> contar lienas
     * -c -> contar caracteres
-  **DIFF** → Comparar archivos:   diff archivo1 archivo2   lo diferente →   <palabradel1 (archivo1)   >palabradel2 (archivo2)  -> diff <(echo “&variable1”) <(echo “&variable2”) 	

---------------------------------------------------------------------------

## Sustituciones
-  **SED** -> Sustituir algo en una cadena de texto
     * sed “s/X/Y/g” → cambia la palabra “X” por “Y”  →  s es de search, buscar, y g es de global (mas de una vez)
     * sed s/ */ /g →  busca “uno o más espacios” y sustituyemelos por “un solo espacio”
     * Elimina todas lineas que empiecen por “#” (comentadas) → sed /^#/d
     * Elimina todas las lineas vacias → sed /^$/d
-  **TR** -> borrar y sustituir caracteres
     * tr -s / → Borrar caracteres repetidos
     * tr -d “ “  → Eliminar caracteres → “\n” saltos de linea “ “ espacios     
     * tr ‘r’ ‘T’ → Susitutye caracteres (r por T) →  para palabras no va por que no toma la palabra sino todas sus letras
---------------------------------------------------------------------------

## Enlaces
 
Simbolicos
```console
[usuario@linux]-[~]:$ ln -s -f ~/ssh/id_rsa ./ssh.txt  # Lo que escribas en ssh.txt lo escribira en ssh/id_rsa (son como el mismo archivo)
[usuario@linux]-[~]:$ unlink ./ssh.txt # Quitar el link
```
-------------------------------------------------------------------------------------------------------------------
   
## Peticiones WEB curl

- **CURL**

Peticion GET
```console
[usuario@linux]-[~/escritorio]:$ curl -s "http://web.xii"	# Peticion GET normal, la salida es el codigo fuente de la página
[usuario@linux]-[~/escritorio]:$ curl -H 		# Te imprime los headers (caracteristicas de la peticion)
```
Peticion POST
```console
[usuario@linux]-[~/escritorio]:$ curl -sX POST "http://web.xii/quote.php" -d "name=github&description=subir codigo a la web&fecha=2022"
[usuario@linux]-[~/escritorio]:$ curl -sX POST "http://web.xii/upload.php" -F "name=Informe" -F "file=<archivo.txt"  # "<" para subir cosas
```
Autenticaciones
```console
[usuario@linux]-[~/escritorio]:$ curl "http://admin.web.xii/manage" -H "Cookie: PHPSESSID=0a123; admin=1" # Poner dos cookies
[usuario@linux]-[~/escritorio]:$ curl "http://web.xii/login" -U "admin:password123"
```
Archivos y Otros
```console
[usuario@linux]-[~/escritorio]:$ curl "http://admin.web.xii/download/archivo.txt -o ./archivo.txt # Descargar el archivo en tu carpeta actual
[usuario@linux]-[~/escritorio]:$ curl "http://admin.web.xii" -x http://localhost:8080   # Redireccionar trafico a un proxy
[usuario@linux]-[~/escritorio]:$ curl -A "Googlebot/2.1 (+http://www.google.com/bot.html)" http://example.com   # Cambiar el User Agent
```







