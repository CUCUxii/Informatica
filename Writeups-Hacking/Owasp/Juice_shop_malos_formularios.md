
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
