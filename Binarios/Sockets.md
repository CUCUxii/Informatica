
fuente [Live Overflow](https://www.youtube.com/c/LiveOverflow/videos)


## Net 0 

En estos ejecicios se introducen las conexiones, es decir aqui la explotación radica en que tenemos que enviar datos a la direccion/puertos 
correspondientes.

El primer ejercicio, Net0, al ejecutarlo parece que no hace nada. Pero si hace, en concreto el proceso se clona creando un demonio o proceso en segundo
plano, mientras que el original se cierra. Al principio define que va a escuchar en el puerto 2999 y va a tener el usuario 999 (los demonios tienen 
siempre un usuario que no deja de ser un numero más)

```console
[user@protostar]-[/opt/protostar/bin]:$ ./ne0
[user@protostar]-[/opt/protostar/bin]:$ netstat -tulvpn | grep "2999"
tcp        0      0 0.0.0.0:2999            0.0.0.0:*               LISTEN    -  
[user@protostar]-[/opt/protostar/bin]:$ nc 127.0.0.1 2999
Please send '1995522783' as a little endian 32bit int
1995522783
I'm sorry, you sent 892942641 instead
```
El binario netcat nos permite enviar paquetes (y comandos) a otras direcciones. Pero como se haría esto manualmente con python?


```python
import socket, struct
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)      # Creamos un socket llamado "con" de tipo ipv4
con.connect(("127.0.0.1", 2999))              # Le asignamos la loopback y el puerto 2999 (el del binario)
print(con.recv(1024))       # Escribimos lo que recibimos de esa conexion.
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
Please send '1095394902' as a little endian 32bit int
```
Quiere que le enviamos ese numero en formato little endian y 32 bits, (o sea dado la vuelta). Pero si ejecuto el binario mas veces salen otros numeros
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
Please send '851767090' as a little endian 32bit int
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
Please send '1461411396' as a little endian 32bit int
```
Asi que lo que hay que hacer el filtrar el numero este directamente del mensaje, ponerlo en formato correcto y volver a enviarlo.
```python
import socket, struct, re
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
con.connect(("127.0.0.1", 2999))
resp1 = con.recv(1024)
numero = re.findall(r"Please send '(.*?)'",resp1)[0]   # Sacar de todo el mensaje el numero en concreto (metiendolo en la variable "numero")
print(numero)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
395667022
```
Aunque cambie este valor, se mantiene mientras se ejecuta el programa, y lo hemos guardado en la variable "numero"
Ahora solo queda mandarlo.
```python
import socket, struct, re
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
con.connect(("127.0.0.1", 2999))
resp1 = con.recv(1024)
numero = re.findall(r"Please send '(.*?)'",resp1)[0]
numero =  struct.pack('I',int(numero))   # El numero se pasa a bytes (Si se pasa como str da error)
con.send(numero)   # Enviarlo y luego recibir el mensaje
resp2 = con.recv(1024)
print(resp2)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
Thank you sir/madam
```

---------------------------------------------------------------

## Net1

Este escucha en el 2998
```python
import socket, struct, re
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
con.connect(("127.0.0.1", 2998))
resp1 = con.recv(1024)
print(resp1)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
?
▒
```
Parece que nos mandan bytes, que el ordenador intenta imprirnos en caracteres ascii (su equivalencia) y al no encontrarlos nos pone estos caracteres tan kafkianos (con saltos de linea "\n"). Asi que habra que pasarlo a otro formato. En concreto a hexadcimal (ya que los caracteres ascii operan con hexadecimal)

> struct.pack es para pasar a bytes. Lo hemos usado en los stack para pasar una memoria en hexadecimal (str) a bytes bien formateados. Puede ser endian 
> "I" o little endian "<I".  struct.pack("I", int o str(hexadecimales, como direcciones de memoria))
> 
> El unpack es para pasar bytes o hexadecimal a ascii, en concreto a un entero (I)  struct.unpack("I",  int o str(hex))

```python
import socket, struct, re
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
con.connect(("127.0.0.1", 2998))
resp1 = con.recv(1024)
print(resp1)
resp1 = struct.unpack("I",resp1)[0]   # Pasar a un entero la data que recibimos (y que sea legible)
con.send(str(resp1))        # Mandarlo de vuelta en formato string (si no da error)
resp2 = con.recv(1024)
print(resp2)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
you correctly sent the data
```
---------------------------------------------------------------

## Net2

Este tiene un bucle donde crea un numero random de 4 bytes, lo imprime (nos lo manda), le suma otro, imprime el resulado (y lo manda)... asi 4 veces.
Asi que voy a utilizar un bucle para recibir 4 datos de 4 bytes y pasarlos a enteros, luego sumarlas.
```python
import socket, struct, re
con = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
con.connect(("127.0.0.1", 2997))
lista = []
for i in range(4):
    resp = struct.unpack("I",con.recv(4))[0]
    lista.append(resp)
print(sum(lista))
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
4812543778
```
Esto ahora hay que mandarselo
```python
numero = sum(lista)
con.send(str(numero))      
print(con.recv(1024))
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
sorry, try again. invalid
```
Poz no le guzto :(. Quiere que se lo enviemos en formato bytes, o sea como lo que nos envia. Entonces lo "structpackeamos" y punto
```console
numero = sum(lista)
con.send(struct.pack("I",numero))
print(con.recv(1024))
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
you added them correctly
```
Ahora zi :)






