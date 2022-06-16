
La "tienda de zumos vulnerable" es una imagen docker del OWASP que simula una web, solo que llena de vulnerabilidades para prácticas

----------------------------------------------------------------------------------------------
## 1 - Acceder a la cuenta del Admin

Es uno de los primeros retos, consiste en loguearse en la cuenta del admin con una inyeccion sql básica.

Explicación -> la inyeccion consiste en poner en el panel de login, **' or 1=1-- -** que dice lo siguiente. 
*O el usuario es 'texto vacío' (NO) O 1=1 (SI)* Como es un OR, se da por vaĺido con que una de esas afirmaciones sea verdad (1=1).

Así que entramos a la cuenta del Admin porque esta inyeccion opera con el id de usaurio 1, que suele corresponder al admin
> Un id  es un índice en una tabla sql que corresponde a cada entrada, normalmente el administrador es el primero que se añade asi que tiene un 1.

----------------------------------------------------------------------------------------------

## 2. XSS

Un ataque XSS es inyectar código a un panel de javascript, resultando en que la web lo asimila como propio e interprete como suyo.
Este se suele probar por tanto con paneles, en este caso el más visible es el de "Search". probé la clásica
```<script> alert("XSS") </script>``` que crea una ventana emergente, no funciónó pero en la "scoreboard" ponía el que si funciaonaría
```<iframe src="javascript:alert('XSS')">``` que es muy similar al "script src"

Tambien, probé a intentar un ataque SSRF aprovechandome de este panel, un SSRF es hacer que el servidor mande una consulta a sí mismo, revelando
datos confidenciales. Como tira contra el localhost, cree un servidor con python ```sudo python -m http.server 80``` quedó la petición asi 
```<iframe src="http://127.0.0.1:8080">``` Haciendome un directory listing o filtrado de carpetas de mi porpio servidor.

Otras cosas que probé fue cargar de mi direccion un index.html con una reverse shell, pero el sistema me lo mostraba sin interpretarlo. Tampoco
funcionço un LFI al /etc/password local.

----------------------------------------------------------------------------------------------

## 3. Filtrado de Rutas

Le hice un *curl* a la web para ver el código fuente, la cosa esque tira de iframe (de ahi la lógica del iframe en el XSS de antes), aun así, este
nos revela ciertas rutas. Para no tener que leer el codigo fuente, se grepea script src que es donde suelen estar los links casi siempre.
```curl http://127.0.0.1:3000 | grep -oP 'script src="(.*?)"``` Obtuve unas cuantas. Entre ellas "main.js" que es la que más me llamo la atención.

```console
[cucuxii]:$ curl http://127.0.0.1:3000/main.js -s | js-beautify > main.txt
```
El js.beautify es para que salga en un formato legible, aun así era inmenso.
La solución otra vez está en aplicar filtros. En scoreboard hablan de descubrir una redirección a una wallet. Así que pobre filtrar por "redirect"
```console
[cucuxii]:$  cat main.js | grep "redirect" 
                                    url: "./redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm",
                                    url: "./redirect?to=https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW",
                                    url: "./redirect?to=https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6",
[cucuxii]:$  curl http://127.0.0.1:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
Found. Redirecting to https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm
```
Como curiosidad esta web tiene un iframe, es decir un cuadro que carga la web real, como una web dentro de otra. Cuando hacemos un curl nos da 
el mismo código siempre, el del marco y no el del contenido, esto es una buena práctica ya que hace que dicho código sea mas complicado de acceder,
pero todavía desde el navegador con boton derecho inspeccionar se ve el bueno.

Otra buena práctica es buscar por el robots.txt, que es una ruta que le indica al motor de busqueda que no indexe ciertas rutas críticas.
```console
[cucuxii]:$  curl http://127.0.0.1:3000/robots.txt
User-agent: *
Disallow: /ftp
```
Probé a intentar acceder al ftp con un wrapper desde el navegador tal que ```ftp://127.0.0.0.1:3000``` pero no resultó.
Otra ruta nueva :3 
```console
[cucuxii]:$  curl http://127.0.0.1:3000/ftp -s | html2text                                                                                                
[                    ]
****** ~ / ftp ******
    * quarantine5/22/2022_12:17:25_PM
    * acquisitions.md9095/22/2022_12:17:18_PM
    * announcement_encrypted.md3692375/22/2022_12:17:18_PM
....
[cucuxii]:$ curl http://127.0.0.1:3000/ftp/acquisitions.md
# Todo el archivo nada interesante...
```
Todo esto son archivos...
Aprovechando el XSS de antes tambien podemos llegar ahí ```<iframe src="http://127.0.0.1:3000/ftp">```

----------------------------------------------------------------------------------------------

## 3. Malas practicas de programación

- **Mala codificación** 
Una página suele mostrar recursos como fotos y demas, esto está definido en el código fuente con por ejemplo ```<img src=/images/gato.png>``` el 
asunto está en el nombre de la foto "gato.png", este tiene que cumplir ciertos requisitos como no incluir caracteres especiales porque va a entrar 
en conflicto, me explicaré con un ejeplo -> un "/" refiere a un subdirectorio o "#" a una sección. Así que el sistema los interpretaá como tal y no 
como simples caracteres del título. Por lo que esos caracteres se codificarán (ejemplo un espacio es %20) y de ahí todos esos caracteres raros que 
salen en las urls. 

En esta web tenemos cierto problema en la seccion "Photo Wall", donde salen fotos y enlaces a twitter. Hay una foto que no carga y su nombre tiene un 
emoticono... Si le das a inspeccionar elemento en la parte de la foto sale un ```src="assets/public/images/uploads/😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg``` O sea un emoticono y algunos "#". Como el sisteam no sabe interpretarlo la foto no carga. Hay que ccorregir el titulo, ponerlo y recargar.

```console
[cucuxii]:$  php --interactive
php > echo urlencode("😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg");
%F0%9F%98%BC-%23zatschi-%23whoneedsfourlegs-1572600969477.jpg    
```
Esto es lo que hay que poner en img src detras de "uploads/"




