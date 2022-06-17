
## Feedback de 0 estrellas.

Cuando nos registramos en la web nos permiten darle el feedback (opinión de usuario) hacia la web. Hay para poner una descripción y darle un rating basado en estrellas.
Te deja poner de 1 a 5 estrellas en el navegador. Pero si interceptamos la peticion por buprsuite (un proxy inverso que nos permite editar esta antes de que llegue 
al servidor) vemos que es una peticion POST hacia /api/Feedbacks/ con los parámetros "userId", "captchaId", "captcha", "comment" y "rating". Este último es el que nos
interesa. Si por ejemplo hemos puesto dos estrellas, saldra un "2" que podremos cambiar a 0 y darle a Fordward para que llegue la petición. Dandonos con eso el logro
"Zero Stars"

----------------------------------------------------------------------------------
