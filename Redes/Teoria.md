

## Interfaces de red

La IP es un identificador de un equipo en una red, pero un ordenador puede estar conectado a redes diferentes, dichas conexiones son independientes, por lo que usan
interfaces distintas. Un ordenador puede estar conectado a la vez a una red local (LAN), a si mismo (localhost) y a una VPN por ejemplo. Si tiene una tarjeta de red
externa (la tipica Tp-link) es otra interfaz más

 - Loopback 
Siempre usa la IP 127.0.0.1, es una conexion que hace el sistema hacia si mismo (no accesible desde fuera) por tanto puede albergar por ejemplo direcciones web en produccion.  
 - Red Local "LAN"
Esa red la constituyen todos los equipos conectados a un mismo punto de acceso (un router), suele empezar por "192.168.1" y tu direccion dentro te la asigna el router.
(IP privada). 
 - VPN 

#### VPN
Una VPN es una conexion a una LAN (red local) pero remota, es decir, te conectas no desde tu ubicacion sino desde la que esté esa LAN, como si cogieras un cable
gigantesco desde tu ordenaodr Argentina y lo conectaras a un router en EE.UU, por tanto sales a internet desde allí (EE.UU), sirve por tanto para burlar
las restricciones locales, tambien ofrece mucha seguridad ya que una conexion por VPN siempre se cifra. Una VPN al ser como un router remoto, tambien te permite 
acceder a una LAN cerrada al exterior.

--------------------------------------------------------------------

## Direcciones IP

Una IP es un identificador de un equipo en una red. En ipv4 Consta de 4 numeros separados por "." del 0 al 255 (por tanto 1 byte cada uno = 8 bits)
A cada numero por tanto se le denomina octeto. (4 bytes = 32 bits)

 - Direcciones IP "especiales" -> son direcciones tipicas que sirven para determinadas cosas
    * 127.0.0.1 -> La loopback, 

--------------------------------------------------------------------

## Puertos y servicios mas comunes 

Un puerto no es mas que el identificador de una conexión. Un ordenador puede tener 65535 (2^16) conexiones diferentes. Cualquier servicio en linea se peude asignar a 
cualquier puerto, pero hay convenciones, es decir, puertos que se suelen poner por defecto.

 - Protocolo HTTP/HTTPS (web) -> 80, 443, 8080
 - Protocolo SSH (control remoto) -> 22
 - DNS (servidor de resolucion de dominios) -> 53
 - Kerberos (Directorio activo) -> 88
 - Base de datos SQL -> 3306
 - Minecraft -> 25565
 - FTP (compartir archivos) -> 21
 - SMB (compartir archivos) ->  137
 - NFS (compartir archivos) -> 2049
