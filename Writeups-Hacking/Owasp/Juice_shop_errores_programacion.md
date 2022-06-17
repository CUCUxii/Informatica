

## 1. Link a imagen defectuoso.

Una página suele mostrar recursos como fotos y demas, esto está definido en el código fuente con por ejemplo ```<img src=/images/gato.png>``` el 
asunto está en el nombre de la foto "gato.png", este tiene que cumplir ciertos requisitos como no incluir caracteres especiales porque va a entrar 
en conflicto, me explicaré con un ejeplo -> un "/" refiere a un subdirectorio o "#" a una sección. Así que el sistema los interpretaá como tal y no 
como simples caracteres del título. Por lo que esos caracteres se codificarán (ejemplo un espacio es %20) y de ahí todos esos caracteres raros que 
salen en las urls. 

En esta web tenemos cierto problema en la seccion "Photo Wall", donde salen fotos y enlaces a twitter. Hay una foto que no carga y su nombre tiene un 
emoticono... Si le das a inspeccionar elemento en la parte de la foto sale un ```src="assets/public/images/uploads/😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg``` O sea un emoticono y algunos "#". Como el sisteam no sabe interpretarlo la foto no carga. Hay que ccorregir el titulo, ponerlo y recargar.

```console
[cucuxii]:$  php --interactive
php > echo urlencode("😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg");
%F0%9F%98%BC-%23zatschi-%23whoneedsfourlegs-1572600969477.jpg    
```
Esto es lo que hay que poner en img src detras de "uploads/"




