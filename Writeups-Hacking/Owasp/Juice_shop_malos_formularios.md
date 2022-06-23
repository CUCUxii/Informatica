
## Feedback de 0 estrellas.

Cuando nos registramos en la web nos permiten darle el feedback (opinión de usuario) hacia la web. Hay para poner una descripción y darle un rating basado en estrellas.
Te deja poner de 1 a 5 estrellas en el navegador. Pero si interceptamos la peticion por buprsuite (un proxy inverso que nos permite editar esta antes de que llegue 
al servidor) vemos que es una peticion POST hacia /api/Feedbacks/ con los parámetros "userId", "captchaId", "captcha", "comment" y "rating". Este último es el que nos
interesa. Si por ejemplo hemos puesto dos estrellas, saldra un "2" que podremos cambiar a 0 y darle a Fordward para que llegue la petición. Dandonos con eso el logro
"Zero Stars"

Ponerle zero estrellas a algo no es crítico, lo que si es crítico es modificar de la misma manera un parámetro que diga si eres admin o no para hacer algo con
altos privilegios.

----------------------------------------------------------------------------------

## Error en el formulario al repetir contraseña

En el panel de login está el clasico "contraseña" y "repetir contraseña", que es para verificar que has puesto una contraseña de la que te acuerdes, obligandote a 
repetirla, pero está tan mal programado que si repites la contraseña y eliminas un caracter en uno de los paneles no pasa nada y te deja continaur.
Nos dan el logro de DRY o "Dont Repeat Yourself" o sea no repetir cosas.

----------------------------------------------------------------------------------

## Mirar la cesta de la compra de otro usuario

Nos registramos como la emma (retos de OSINT), y añadimos un par de productos a la cesta de la compra
Cuando un usaurio hace un pedido en una web, normalmente esto se guarda en alguna parte de la web como datos asociados al usaurio (modificables
en cualquier momento) pudiendo estar tanto en la cookie o la peticion.

Esto en un navegador se ve en la seccion "Almacenamiento", descrifrando la cookie con jwt.io he encontrado cosas interesantes, pero nada 
respecto a la cesta de la compra.
En "almacenamiento de sesion" hay dos campos "item.total" y "key", por lo que parece que estos datos si se refieren a los pedidos.
El bid de emma es 6, el de jhon un 7, asi que dicho bid parece un id de usaurio. Si ponemos un 2 y refrescamos, veremos el pedido de otro
usaurio y nos dará el logro.

----------------------------------------------------------------------------------

## Hacer un review en nombre de otro usaurio

Cuando clicas sobre un producto, puedes dejar una review, si editamos la peticion... vemos un campo "author", donde sale el nombre de quien hace la 
review, si lo cambiamos por cualquier otra cosa nos da el logro.
```
{"message":"test","author":"Anonymous"} -> {"message":"test","author":"bender"} 
```
----------------------------------------------------------------------------------

## Registrar a un usaurio directamente como administrador

En la seccion de registro de un nuevo usaurio. Creamos un usaurio random (da igual lo que pongamos), en el navegador en la seccion "Red" vemos todas las peticiones,
si buscamos por las del método POST, hay una hacia /api/Users, clicamos y en la pestaña solicitud y "sin procesar" podemos ver el formulario de la petición para
crear un nuevo usaurio, lo copiamos y con curl crearemos otro usaurio con privilegios de adminsitrador, esto se suele hacer al poner el campo oculto "role"
como "admin" o como "1". No me funcionó y lo acabé haciendo con el navegador. 

En concreto en la seccion de red de antes, click derecho y "editar peticion y volver a enviar", añadimos lo de "role":"admin" en el "cuerpo de la petición" y 
le damos a enviar, con esto ya tendremos el logro. Tambien se puede hacer de manera muy similar con la herramienta "burpsuite" que edita las peticiones antes de
que lleguen.

----------------------------------------------------------------------------------

## Hacer un pedido a nombre de otro usaurio

Cuando hacemos un pedido (le damos a un producto a "Add to basket") en el apartado de "Red" se produce una petición POST al endpoint 
"/api/BasketItems" y un parámetro "Basket ID" que se refiere a nuestra cesta. 

SI lo cambiamos nos da un forbidden en la seccion de errores, es decir ese parametro está santizado y no se puede cambiar, asi que tuve que estar mucho
más tiempo para ver como bypasear todo:

Gracias a la vulnerabilidad **"Parameter pollution"** podemos añadir otro "Basket Id" con otro valor despues
```
 {"ProductId":24,"BasketId":"3","quantity":1} -> {"ProductId":24,"BasketId":"3","quantity":1, "BasketId":"2"} 
```
El primero esta sanitizado, pero no lo hemos cambiado asi que sin problema, el segundo no, asi que se efectua y ademas cancela al primero
asi que hacemos la peticion por otro usaurio y a nosotros no se nos añade nada,

----------------------------------------------------------------------------------

## Pedir el producto especial Navideño de 2014

En el endpoint vulnerable a una sqli (ver inyecciones), vimos todos los productos, si hacemos un ctrl+F y buscamos por "Christmas" nos sale que el 
producto aparentemente retirado de navidad de 2014 tiene el id de 10. Así que modificando la petición a "/api/BasketItems" con su "productId":
```
 {"ProductId":10,"BasketId":"3","quantity":1} 
```
Y siguiendo todos los pasos para hacer la compra obtendremos el logro.

----------------------------------------------------------------------------------

## Hacer una redirección saltandose la whitelist

Con el "main.js", archivo que sacamos de ver el codigo fuente del iframe de la web. Si filtramos por ./redirect (lo hicimos para sacar una wallet oculta
en otro reto)  ```console [cucuxii]:$ cat main.js | grep "redirect"``` Nos da varias url como por ejemplo esta
```"./redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm"```. SI la ponemos nos redirige a esta cartera de la blockchain,
esto es parámetro de redirección (esta se hace a nombre del servidor y no del cliente). Es decir con este endpoint podemos redirigir la web a donde
queramos....

O no. SI ponemos una redireccion a otra página (ejemplo wargames que es para practicar pentesting)
```http://localhost:3000/redirect?to=https://overthewire.org/wargames``` Nos da un error de que no tenemos permiso para redirigir a un sitio externo.

Las urls que nos habian salido tras lo de redirect (como la de la wallet) son una whitelist, es decir una lista de sitios a los que puedes redirigir sin 
error. Todo lo que esté fuera de ahi estará prohibido. Pero hay alguna manera de saltarse eso como el **parameter pollution**, una vulnerabilidad
que consiste en meter varios paráemtros iguales para que lea solo uno y cancele el restringido.
```http://localhost:3000/redirect?to=https://overthewire.org/wargames?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm```

Aquí el sistema lee que hemos puesto la página de la wallet, por lo que no se queja, pero al haber dos parametros "to" solo lee uno (en este caso el 
primero) que es el nuestro, por lo que nos redirige a dodne nos de la gana.

----------------------------------------------------------------------------------

## Subir un archivo no permitido

En la seccion "complaint" te dejan subir un archivo, el asunto esque solo puede ser un pdf o zip. Si queremos subir por ejemplo un php no nos dejará.
Subimos un pdf cualquiera, preferiblemente pequeño "Test.php". 
Interceptemos con Burpsuite la petición (o editemos y repitamos una capturada por el navegador en Red).

Si la web está muy vagamente programada (este es el caso), podremos editarlo y nos lo acepta, ademas lo intepreta si cambiamos:

```filename="Test.php" Content-Type: aplication/php```

----------------------------------------------------------------------------------

## Usar una interfaz desactualizada.

Si miramos en codigo fuente "main.js" y buscamos por "allowed" para que nos salga la lista de archivos permitidos
```console
[cucuxii]:$ cat main.js | grep -B3 -n "allowed"                                                                                                           
2881:  url: "./file-upload"
2882:  #nada interesante
2883:  allowedMimeType: ["application/pdf", "application/xml", "text/xml", "application/zip", "application/x-zip-compressed", "multipart/x-zip"]
```

Sale tambien "xml", pero solo nos deja seleccionar pdf y zip, asi que hay que subir uno de estos, lo que sea... 
modificar la peticion con el burpsuite y cambiar lo correspondiente para que nos den el logro;
En la parte del cuerpo, poner el xml, en este ejemplo "star_wars.xml".

```
Content-Disposition: form-data; name="file"; filename="star_wars.xml"
Content-Type: application/xml

<!DOCTYPE foo>
<episode_1>
        <year>1999</year>
        <name>The phantom menace</name>
</episode_1>
<episode_2>
    <year>2002</year>
    <name>Attack of the Clones</name>
</episode_2>
<episode_3>
    <year>2005</year>
    <name>Revenge of the Sith</name>
</episode_3>
```

----------------------------------------------------------------------------------

## Obten las credenciales de otra persona sin inyecciones sql

Cuando pagas una serie de productos (completas todo el proceso, en este caso con la cuenta de "Bender"), hay una peticion tal que asi
```http://localhost:3000/rest/track-order/130f-3e9c4157d07fe339```
```json
{"status":"success","data":[{"promotionalAmount":"0","paymentId":"6","addressId":"6","orderId":"130f-3e9c4157d07fe339","delivered":false,"email":"b*nd*r@j**c*-sh.*p","totalPrice":5005.98,"products":[{"quantity":1,"id":4,"name":"Raspberry Juice (1000ml)","price":4.99,"total":4.99,"bonus":0},{"quantity":1,"id":42,"name":"Best Juice Shop Salesman Artwork","price":5000,"total":5000,"bonus":500}],"bonus":500,"deliveryPrice":0.99,"eta":"1","_id":"m9nCHN65dCFNLcXez"}]}
```
Aquí lo que interesa es el campo del email -> "email":"b*nd*r@j**c*-sh.*p" 
Los "*" se han puesto para susituir caracteres y que no haya una filtracion de información personal, eliminando un trozo de esta.

La vulnerabilidad reside en registrar otro usaurio cuyo email tambien coincida con este patron ```"b*nd*r@j**c*-sh.*p"```
como "bandor@juice-sh.op". Una vez eso, en la seccion de "Privacidad y seguridad" dale a "Request Data Export" para que te de todo sus datos.





