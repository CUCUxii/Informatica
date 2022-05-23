

## DNS

Dns o domain resolution service es un servidor intermediario (normalmente gestionado por nuestro provedor de internet) al que le hacemos peticiones mandandole strings
(direcciones web) y este nos contesta con la IP correspondiente, a la que se le hace la petición.

---------------------------------------------------------------------------

## SOCKETS

Los sockets son archivos en los que se escribe la data que se quiere enviar. Estos sockets (ejemplo con tcp) se encargan de enviar los bytes a la otra maquina
sin que el programador de turno tenga preocuparse por cuestiones mas complejos de la conexion como saltos y esas cosas. Todo eso lo hace el sistema operativo.
```
PAQUETE TCP
  ORIGEN:10.10.10.10:5000    cadena "Hola Mundo"    DESTINO:10.10.20.20:5001
```
Por ejemplo en Linux 

---------------------------------------------------------------------------

## Protocolo TCP

Protocolo de Control de Transmision. Es un protocolo de conexion que tiene varias medidas de seguridad para que la conexión sea fiable.

1. Se encarga de comprobar que no se pierdan datos (paquetes) en el camino. 
2. Numera los paquetes para que se reordenen.
3. Tiene un checksum para comprobar que no se corrompan
4. Comprueba que hay memoria suficiente para recibir todos los paqeutes

TCP tiene una medida para comprobar que los paquetes llegan al destino.

