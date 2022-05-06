# Protocolo http

Es el protocolo de trasnferencia de datos por hipertexto, es decir, por poner un texto, te trasnfiere cosas (http:// o https://)
Es un protocolo para cliente, que hace una petición a un servidor que le responderá con contenido.
Las apis son algo similar, hacemos una petición al conjunto de software y este nos reponderá.


Que es un servidor? Es un ordenador al que puedes acceder desde internet (obviamente, a una parte restringida de el).
Guarda un codigo HTML (enriquecido con un CSS y un Javascript) que se te muestra por pantalla, decorado y bonito, también imagenes que se te interpretan, 
o un programa (juego, red social, peticion curl…)

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
ahorrandote autenticarse todo el rato. Compras un articulo, que esta en una SQL, la pagina de compra te dice su precio, te muestra su foto y datos…
todos esos datos sacados del registro correpondiente. Compras el articulo y le envia al banco una peticion...
