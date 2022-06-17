

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
-----------------------------------------------------------------------------------------------



