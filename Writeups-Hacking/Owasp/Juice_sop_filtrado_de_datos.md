

## MÉTRICAS EXPUESTAS

Descripción: "encuentra el endpoint que muestra el uso de datos de un sistema de monitoreo famoso (prometheus)". 
Con una busqueda rápida en google vi lo que era "prometheus", un repositorio de un sistema de monitoreo de código abierto (vigilar el estado de la página, el tráfico,
las fallas...) luego busqué "prometheus monitoring endpoints" y con Ctrol+F y poniendo como filtro "/" para buscar rutas encontré metrcis, Por tanto
al poner en la URL ```"http://localhost:3000/metrics"``` di con ello

-----------------------------------------------------------------------------------------------

## ACCESO A DOCUMENTOS CONFIDENCIALES

Una buena práctica es buscar por el robots.txt, que es una ruta que le indica al motor de busqueda que no indexe ciertas rutas críticas.
```console
[cucuxii]:$  curl http://127.0.0.1:3000/robots.txt
User-agent: *
Disallow: /ftp
```
Probé a intentar acceder al ftp con un wrapper desde el navegador tal que ```ftp://127.0.0.0.1:3000``` pero no resultó, pero te dice que existe la ruta /ftp
```console
[cucuxii]:$  curl http://127.0.0.1:3000/ftp -s | html2text                                                                                                
****** ~ / ftp ******
    * quarantine5/22/2022_12:17:25_PM
    * acquisitions.md9095/22/2022_12:17:18_PM
    * announcement_encrypted.md3692375/22/2022_12:17:18_PM
....
[cucuxii]:$ curl http://127.0.0.1:3000/ftp/acquisitions.md
# Todo el archivo nada interesante...
```
Todo esto son archivos...
Aprovechando el XSS del panel del búsquéda tambien podemos llegar ahí ```<iframe src="http://127.0.0.1:3000/ftp">```

-----------------------------------------------------------------------------------------------

## BYPASSEAR LA EXTENSIÓN PARA VER DATOS CONFIDENCIALES

En esa ruta "/ftp" hay unos cuantos archivos, pero no todos se pueden ver, por ejemplo "package.json.bak" al intentar mirarlo da error diciendo que 
no tiene una extensión permitida. Eso es porque el sistema le concatena otra extension (no de cual) para que el navegador no te deje visualizarlo.

Pero eso se puede saltar. Hay una cosa llamada "null byte", que es %00 o %2500 (urlencodeando el % a %25) y si se pone al final la ruta que nos ponen 
siempre se deja de leer y ya no da problemas. Si ademas de este "%2500" le metemos ".md" nos dira de descargar el archivo o abrirlo (no esque el
archivo sea un md porque el null byte lo quita sino que el navegador lo entiende como tal)
Asi que podemos bajar tres archivos
```
http://localhost:3000/ftp/eastere.gg%2500.md
http://localhost:3000/ftp/package.json.bak%2500.md
```
Vale, sabemos que hace un null byte, pero ¿porque? En lenguaje de mas bajo nivel, cuando le pasadmos una string a una funcion, por ejemplo 
en C ```printf("Hola Mundo\n")``` a nivel de binario (a parte del salto de linea) le mete un byte "\00" que le indica a la funcion que la string
que le hemos pasado acaba ahí y que no siga leyendo.

-----------------------------------------------------------------------------------------------

## HUEVO DE PASCUA

Un huevo de pascua o easter egg es un pequeño juego que tienen los programadores (normalmente de videojuegos) de esconder una referencia a algo que no 
tiene nada que ver como chiste interno. Como si estamos en un juego de guerra y abrimos una puerta escondida y hay un muñeco de super mario porque si.

En esta web el huevo de pascua es uno de esos archivos que pudimos ver con el truco del null byte, pero no esque sea el huevo en si, sino un mensaje
con una cadena de base64 (se puede distinguir facilmente porque tiene un "==" al final y letras maysuculas/minusuclas/numeros)

Esto en bash se decodifica así
```console
[cucuxii]:$ echo "L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==" | base64 -d; echo
/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt
```
Tenemos una serie de rutas (se ve por los "/") pero cuyos nombres no tienen sentido. Eso es que se le ha aplicado un ROT13, un algoritmo de criptograia
muy muy básico en el que cada letra se corre 13 posiciones en el alfabeto (pero no afecta a simbolos especiales como los "/"). Esto se puede descrifrar 
en alguna [web](https://rot13.com/). 
Queda:
```
/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
```
Haces una peticion a ```http://localhost:3000/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg``` y te sale un minijuego con un 
planeta, eso es el easter egg.

**EXTRA** -> ¿Que es base64?. Cando hablamos de base hablamos del sistema numerico, base 2 es 0 y 1 (dos caracteres), base 10 son nuestros numeros del 
1 al 9 y base16 (hexadecimal) son numeros hasat el 8 y letras de la "a" a la "f" porque no hay mas numeros, siendo la "a" el numero 10 y la "f" el 16.
Cuando mas alta sea una base menos caracteres habra que usar para representar un número (y por tanto una letra)

> La "A" es 41 en hexadecimal y 65 en decimal, mientras que en binario es "01000001" 


-----------------------------------------------------------------------------------------------

## ACCESO A RUTAS SECRETAS

Le hice un *curl* a la web para ver el código fuente, la cosa esque tira de iframe, aun así, este nos revela ciertas rutas. Para no tener que leer 
el codigo fuente, se grepea script src que es donde suelen estar los links casi siempre.
```console
[cucuxii]:$ http://127.0.0.1:3000 | grep -oP 'script src="(.*?)"
``` 
> Como curiosidad esta web tiene un iframe, es decir un cuadro que carga la web real, como una web dentro de otra. Cuando hacemos un curl nos da 
> el mismo código siempre, el del marco y no el del contenido, esto es una buena práctica ya que hace que dicho código sea mas complicado de acceder,
> pero todavía desde el navegador con boton derecho inspeccionar se ve el bueno.

Obtuve unas cuantas rutas. Entre ellas "main.js" que es la que más me llamo la atención. El js.beautify es para que salga en un formato legible, aun así era inmenso.
```console
[cucuxii]:$ curl http://127.0.0.1:3000/main.js -s | js-beautify > main.txt
```
La solución otra vez está en aplicar filtros. En scoreboard hablan de descubrir una redirección a una wallet. Así que podre filtrar por "redirect"
```console
[cucuxii]:$  cat main.js | grep "redirect" 
                                    url: "./redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm",
                                    url: "./redirect?to=https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW",
                                    url: "./redirect?to=https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6",
[cucuxii]:$  curl http://127.0.0.1:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
Found. Redirecting to https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
```
El score-board tambien se encuentra facilmente con el grep, y la seccion de administración oculta también
```console
[cucuxii]:$  cat main.js | grep "score"
path: "score-board"
[cucuxii]:$  cat main.js | grep "admin"
path: "administration"
```
Conseguir la ruta de /administracion que no era visible en la wwb nos da el logro "Admin Section"
Con la ruta de /administration y gracias a la inyeccion sql que nos otorgó ser admin, si borramos del customer feedback la que dio 5 estrellas, obtendremos el logro
**Five-Star-Feedback**

-----------------------------------------------------------------------------------------------

## Foto con Metadatos de localización

Este reto toca la pata del OSINT, o "busqueda de información de fuentes abiertas" que consisten en buscar información personal mediante busquedas especialiazadas
en redes sociales y motores de búsqueda entre otros (por tanto no se considera ilegal). Este en concreto, aprovecha los metadatos de una imagen para sacar la 
localización de una persona. Los metadatos son datos sobre una imagen (resolución, tamaño, dispositivo y lcoalización...) Estos no se ven a primera vista, pero
ahi están, y si tienen inoformación muy personal puede ser desastroso. Un consejo para evitar esto es desactivar la geolocalización del teléfono cuando se saca
una foto.

Hay una foto de un tal "jhonny" que siguiendo la lógica de la web, su correo es "john@juice-sh.op"  (no johnnt segun las pistas)

```console
[cucuxii]:$ curl -s "http://localhost:3000/assets/public/images/uploads/favorite-hiking-place.png" -o foto.png
[cucuxii]:$ exiftool foto.png | grep "GPS" | tail -n1
GPS Position                    : 36 deg 57' 31.38" N, 84 deg 20' 53.58" W
[cucuxii]:$ echo "36º 57' 31.38\" N 84º 20' 53.58\" W"
36º 57' 31.38" N 84º 20' 53.58" W
```
Buscamos estas coordenadas en google maps y nos sale un bosque de Kentucky "Daniel Boone National Forest"
Asi que en el panel de login se le cambia a john la contraseña gracias a la pregunta de seguirdad, "¿Cual es tu sitio favorito
para ir de paseo?" y la respuesta es el bosque este que hemos encontrado. Como error de la web, cuando un usaurio existe, se muestra
su pregunta de seguridad, asi que puede ser un mecanismo para hallar usaurios existentes.

Con una tal emma pasa lo mismo, pregunta de seguridad gracias a foto. Solo que con ella el procedimiento era diferente. En concreto 
visualizando la foto y haciendo zoom, hasta ver la palabra "ITSec" en una ventana.










