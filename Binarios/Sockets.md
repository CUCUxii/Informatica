
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
```console
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
numero =  struct.pack('<L',int(numero))   # El numero se pasa a entero (porque es una string) y  a little endian "<L" (no se puede hacer con strings)
con.send(numero)   # Enviarlo y luego recibir el mensaje
resp2 = con.recv(1024)
print(resp2)
```
```console
[user@protostar]-[/opt/protostar/bin]:$ python /tmp/exploit.py 
Thank you sir/madam
```


