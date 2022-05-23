# Protocolo http

Es el protocolo de transferencia de datos por hipertexto, es decir, por poner un texto, te trasnfiere cosas (http:// o https://)
Es un protocolo para cliente, que hace una petición a un servidor que le responderá con contenido.
Las apis son algo similar, hacemos una petición al conjunto de software y este nos reponderá.

Que es un servidor? Es un ordenador al que puedes acceder desde internet (obviamente, a una parte restringida de el).
Guarda un codigo HTML (enriquecido con un CSS y un Javascript) que se te muestra por pantalla, decorado y bonito, también imagenes que se te interpretan, 
o un programa (juego, red social, peticion curl…)

-------------------------------------------------------------------------

## Peticion al servidor

En una peticion web se envian lo que llamamos cabeceras, una string con los datos de la peticion. Se suelen escribir automáticamente pero con python requests o curl
se pueden modificar.

 - Peticion -> ```GET /store/index.html  HTTP/1.1```
 - Host ->  ip  ```Host: 10.10.20.20```
 - User agent -> programa mediante el cual realizas la peticion (un navegador, python, el curl…)   ```User-Agent: Mozilla/5.0```
 - Cookie -> json serializado que tiene credenciales o entradas de bases de datos (utiles para webs de compras)
 - Content-Type -> tipo de contenido que estas enviando (util para que el servidor sepa como lo tendra que gestionar) ```Content-Type: text/html; charset=UTF-8```
 - Contenido -> Lo que mandas, Ejemplo de peticion por post a un formulario de contactos  ```nombre=Antonio&correo=mariscos@recio.com```

## Respuesta del servidor

Tambien tiene headers, pero a diferencia de una peticion, una respuesta tiene contenido (por ejemplo una imagen, una respuesta de una base de datos, un html)
Todo eso en texto plano (bytes). Por ejemplo el html nos viene como codigo html que nuestro navegador formatea automaticamente siguiendo un css por links.
Es decir, los Los links de un html los interpreta.  Pero si la peticion se hace con el programa "curl" se imprime todo como RAW o texto plano.

 - Respuesta ->  Protocolo y Codigo de estado ```HTTP/1.1 200 OK```  
 - Content-Lenght -> la longitud de la respuesta ```Content-Lenght: 3288```

-------------------------------------------------------------------------

### 1. Métodos

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

### 2. Códigos de estado

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
	
### 3.Tipos de contenido

Los content type. Sirven para que el navegador identifique que tipo de archuvo se esta mandando y lo muestre correctemente ```Content-Type: text/html; charset=UTF-8```
* application/json → para jsons → ```curl -H “Content-Type: application/json”```
* text/html -> para htmls  → ```curl -H “Content-Type: application/json”```
* text/css -> para los css
* image/png (o jpg) -> para fotos







