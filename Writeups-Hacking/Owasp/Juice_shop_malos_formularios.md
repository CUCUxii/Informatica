
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


















