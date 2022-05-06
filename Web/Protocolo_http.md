# Protocolo http

Es el protocolo de trasnferencia de datos por hipertexto, es decir, por poner un texto, te trasnfiere cosas (http:// o https://)
Es un protocolo para cliente, que hace una petición a un servidor que le responderá con contenido.
Las apis son algo similar, hacemos una petición al conjunto de software y este nos reponderá.


Que es un servidor? Es un ordenador al que puedes acceder desde internet (obviamente, a una parte restringida de el).
Guarda un codigo HTML (enriquecido con un CSS y un Javascript) que se te muestra por pantalla, decorado y bonito, también imagenes que se te interpretan, 
o un programa (juego, red social, peticion curl…)

-------------------------------------------------------------------------

## Peticion al servidor

En una peticion web se envian dos cosas, las cabeceras y los datos. Los datos son lo que manda en la conexión y las cabeceras son los detalles 
sobre esa conexión.

 - MÉTODOS -> Son tipo de operación vas a hacer en el servidor, es decir, si mandar datos o leer unos que existen, etc
 - CABECERAS ->  son las caracteristicas de la peticion (cookie, fecha, tipo de contenido…). Se ponen automaticamente, aunque se pueden modificar.

Con esos datos, el servidor puede hacer varias cosas; 
 - Entrada de un comando → estos datos pueden pasarse de entrada a un programa y mostrarte su respuesta. (ejemplo, te da la informaciónd de una foto por exiftool)
 - Entrada de base de datos → envias datos que se guardan como entrada	
 - Serializarse o deserializarse → para convertirse por ejemplo en una cookie (ej . datos en formulario de registro)
 - Guardarse como recurso en el servidor → cuando subes una imagen o texto esta se guarda dentro del servidor, en la carpeta correspondiente.
				
El servidor te manda una respuesta, en base a esos datos  
Ejemplo → te registras en un formulario, esos datos se convierten en una cookie y esa cookie viaja como Header en el resto de peticiones,
ahorrandote autenticarse todo el rato.  
Compras un articulo, que esta en una SQL, la pagina de compra te dice su precio, te muestra su foto y datos… todos esos datos sacados del registro correpondiente. Compras el articulo y le envia al banco una peticion...

-------------------------------------------------------------------------

## Cabeceras

 - User agent -> programa mediante el cual realizas la peticion (un navegador, python, el curl…)
```console 
[cucuxii]─[Timming]$ curl -A "Googlebot/2.1 (+http://www.google.com/bot.html)" http://example.com
```
 - Cookie -> json serializado que tiene credenciales o entradas de bases de datos (utiles para webs de compras)
 - Content-Type -> tipo de contenido que estas enviando (util para que el servidor sepa como lo tendra que gestionar)
    * application/json → para jsons → curl -H “Content-Type: application/json”

-------------------------------------------------------------------------

## Métodos

Definen que tipo de accion se puden hacer sobre un recurso.

- Acciones seguras: si se puede realizar sin que modifique el servidor > ej solo lecturas
- Accioens idempotentes: la misma accion varias veces seguidas y que el servidor se quede igual > ej crear un recurso y que si lo creas otra vez no se cree una copia)
- Acciones cacheables: el cliente se pueda guardar el resultado: descargas

- **GET** ->  Es lectura de un recurso. → seguro, cacheable e idempotente, siempre son peticiones que van por url, viendose los datos todo el rato. (texto, fotos…) 
- **POST** ->  para crear nuevos recursos o identidades en la base de datos. Genera un cambio en el servidor, no es idempotente ni seguro. Son peticiones por formulario, o paneles en la web, no se ven los datos en la url, por lo que son mas discretos (contraseñas) 
- **HEAD** -> como el GET, pero no te da respuesta. 
- **PUT** -> muy similar a Post solo que reemplaza todos los datos de una entidad, si no exisita, la crea. “La pone”
- **DELETE** ->  elimina el recurso en el servidor que indicas en la url
- **CONNECT** ->  genera una conexión con el recurso del servidor
- **OPTIONS** ->  para pedirle al servidor que se puede hacer con él
- **TRACE** ->  realiza el envio de un mensaje para ver por donde pasa
- **PATCH** ->  para hacer modificaciones parciales (no como el put que es para completas)

En una llamada http tenemos el verbo (GET, POST…) y la entidad  https://pokeapi.co/api/v2/item/master-ball (la entidad es master-ball, el resto de su ruta)

Directory listing: cuando por GET pones una carpeta que contiene entidades y te las enumera:   (como hacer un ls)
	Ej   https://pokeapi.co/api/v2/pokemon (te salen los pokemon disponibles)
	
-------------------------------------------------------------------------

## Códigos de error

[Estandar](https://es.wikipedia.org/wiki/Anexo:C%C3%B3digos_de_estado_HTTP) de codigo de error: numeros
- Informativos: 100 -199 
- Satisfactorios: 200-299 todo bien     
- Redirecciones: 300-399   Si te manda a otra 301
- Errores de clientes: 400-499 mala petición
- Errores del servidor : 500-599 el servidor va mal

Ej 200 ok y 201 created (PUT) 202 accepted (acepta la petcion pero no la ha procesador todavia). Ejemplos
 - 301: se ha movido la url
 - 400: bad request (has puesto mal la url, no puede interpretar pues la solicitud
 - 401: con unas credenciales no puedes acceder al recurso
 - 404: no existe el recurso
 - 504: significa TIMEOUT. Es un error que aparece cuando el tiempo es demasiado largo
	








