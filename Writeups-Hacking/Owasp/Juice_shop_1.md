
La "tienda de zumos vulnerable" es una imagen docker del OWASP que simula una web, solo que llena de vulnerabilidades para prácticas

----------------------------------------------------------------------------------------------
## 1 - Acceder a la cuenta del Admin

Es uno de los primeros retos, consiste en loguearse en la cuenta del admin con una inyeccion sql básica.

Explicación -> la inyeccion consiste en poner en el panel de login, **' or 1=1-- -** que dice lo siguiente. 
*O el usuario es 'texto vacío' (NO) O 1=1 (SI)* Como es un OR, se da por vaĺido con que una de esas afirmaciones sea verdad (1=1).

Así que entramos a la cuenta del Admin porque esta inyeccion opera con el id de usaurio 1, que suele corresponder al admin
> Un id  es un índice en una tabla sql que corresponde a cada entrada, normalmente el administrador es el primero que se añade asi que tiene un 1.

----------------------------------------------------------------------------------------------
