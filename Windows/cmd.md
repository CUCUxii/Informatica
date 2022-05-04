# Guía de cmd (linea de comandos)
 - [Movimiento](#movimiento)
 - [Filtros](#filtro)
 - [Información del Sistema](#informacion-del-sistema)
---------------------------------------------------------------------------

## Movimiento

-  **CD** -> cambiar de directorio (igual que en Linux)  *cd ruta*
-  **DIR** -> listar todos los contenidos de una carpeta *dir ruta*
```cmd
C:\Users\cucuxii> dir C:\Users\cucuxii\Documents
29/04/2022  10:55    <DIR>          .
29/04/2022  10:55    <DIR>          ..
10/11/2020  20:24    <DIR>          Carpeta
C:\Users\cucuxii> cd C:\Users\cucuxii\Documents\carpeta    # o dir .\carpeta (ruta relativa)
C:\Users\cucuxii\Documents\carpeta> dir 
10/11/2020  20:24    <DIR>          Archivo1.txt
10/11/2020  20:24    <DIR>          Archivo2.txt
C:\Users\cucuxii\Documents\carpeta> cd ..\
C:\Users\cucuxii\Documents> 
```
Ordenar por orden afalbético **dir /a** 
```cmd
C:\Users\cucuxii> dir /a C:\Users\cucuxii\Documents\carpeta
10/11/2020  20:24                   Archivo1.txt
10/11/2020  20:24                   Bandas_rock.txt
10/11/2020  20:24    <DIR>          Carpeta
```
Mostrar solo archivos (nada de carpetas)  **dir /a-d**
```cmd
C:\Users\cucuxii> dir /a-d C:\Users\cucuxii\Documents\carpeta
10/11/2020  20:24                   Archivo1.txt
10/11/2020  20:24                   Bandas_rock.txt
```
Mostrar solo nombres y no tanta información. **dir /b**
```cmd
C:\Users\cucuxii> dir /b C:\Users\cucuxii\Documents\carpeta
Archivo1.txt
Bandas_rock.txt
Carpeta
```
Recursividad (carpetas dentro de carpetas) **dir /s**
```cmd
C:\Users\cucuxii> dir /a C:\Users\cucuxii\Documents\carpeta
10/11/2020  20:24                   Archivo1.txt
10/11/2020  20:24                   Bandas_rock.txt
10/11/2020  20:24    <DIR>          Carpeta
Directorio de C:\Users\cucuxii\Documents\carpeta\Carpeta
10/11/2020  20:24                   Factura1.txt
10/11/2020  20:24                   Factura2.txt
```
Mostrar por columnas **dir /D**
```cmd
C:\Users\cucuxii> dir /a C:\Users\cucuxii\Documents\carpeta
\[Archivo1.txt]         \[Archivo2.txt]         \[Carpeta]
```

Buscar archivos por su extension, (y con recursividad) ejemplo txt -> **>dir /b/s | findstr "txt"**
```cmd
C:\Users\cucuxii\Documents>  >dir /b/s | findstr "txt"
C:\Users\cucuxii\Documents\carpeta\Carpeta\Factura1.txt
C:\Users\cucuxii\Documents\carpeta\Carpeta\Factura2.txt
```

---------------------------------------------------------------------------

## Filtro 

-  **Findstr** -> comando para filtrar la salida.

findstr patron
```cmd
C:\Users\cucuxii> dir C:\Windows\System32
Muuuchos archivos y carpetas...
C:\Users\cucuxii> dir C:\Windows\System32 | findstr "cmd"
15/01/2021  11:12           289.792 cmd.exe
07/12/2019  11:09            20.480 cmdkey.exe
```
 Busca cosas por cada patrón (ejemplo salen tanto las busquedas de 'cmd' como de 'config') -> **findstr "patron1 patron2"**
```cmd
C:\Users\cucuxii> dir C:\Windows\System32 |findstr "cmd config"
15/01/2021  11:12           289.792 cmd.exe
07/12/2019  11:09            20.480 cmdkey.exe 
22/04/2022  09:38    <DIR>          config
11/02/2022  11:41           215.896 coreglobconfig.dll
```
 busquedas case insentive, ignora si son mayusculas o minusculas (es decir sale tanto patron como PATRON como Patron...) **findstr /I "patron"**
```cmd
C:\Users\cucuxii> dir C:\Windows\System32 | findstr "device"
16/04/2021  19:44           231.248 containerdevicemanagement.dll
15/01/2021  11:11           240.688 deviceaccess.dll
C:\Users\cucuxii> dir C:\Windows\System32 | findstr /I "device"
16/04/2021  19:44           231.248 containerdevicemanagement.dll
15/01/2021  11:11           240.688 deviceaccess.dll
07/12/2019  11:08            21.184 DefaultDeviceManager.dll
```
justo que no salga un patrón findstr -> **findstr /V "patron"**
```cmd
C:\Users\cucuxii\Documents\carpeta> dir 
10/11/2020  20:24                   Archivo1.txt
10/11/2020  20:24                   Archivo2.txt
10/11/2020  20:24    <DIR>          Carpeta
C:\Users\cucuxii\Documents\carpeta> dir | findstr /V "Archivo"
10/11/2020  20:24    <DIR>          Carpeta
```

---------------------------------------------------------------------------

## Información del sistema

-  **tasklist** -> Procesos en ejecución

```cmd
C:\Users\cucuxii>tasklist |findstr /vi "svchost Chrome"   # Quitamos estos dos porque se repiten demasiado
System Idle Process              0 Services                   0         8 KB
System                           4 Services                   0        20 KB
Registry                       124 Services                   0    26.628 KB
smss.exe                      1020 Services                   0       232 KB
```
Buscar todos los procesos que carga un usuario en concreto **tasklist /fi "username eq Elliot"**
```cmd
C:\Users\cucuxii>tasklist /fi "username eq cucuxii"
sihost.exe                    3408 Console                    1    97.140 KB
svchost.exe                   3452 Console                    1    27.468 KB
svchost.exe                   3532 Console                    1    20.780 KB
```
Buscar todos los procesos que no funcionan **tasklist /fi "status eq not responding"**
```cmd
C:\Users\cucuxii>tasklist /fi "status eq not responding"
jusched.exe                  13188 Console                    1    12.368 KB
```
Buscar los procesos que consumen muchos recursos -> **tasklist /fi "memusage gt 300000"**
```cmd
C:\Users\cucuxii>tasklist /fi "memusage gt 300000"
SearchApp.exe                 8724 Console                    1   359.116 KB
soffice.bin                   7432 Console                    1   329.460 KB
```

--------------------------------------------------------------------------------

## Redes
-  **netstat** 
```cmd
C:\Users\cucuxii> netstat -ano
TCP    192.168.0.236:139      0.0.0.0:0              LISTENING       4
TCP    192.168.0.236:49484    185.116.156.173:25565  TIME_WAIT       0
TCP    192.168.0.236:49486    198.50.209.201:25565   ESTABLISHED     15196
TCP    192.168.0.236:49521    185.199.108.154:443    ESTABLISHED     13988
TCP    192.168.0.236:49522    185.199.108.133:443    ESTABLISHED     13988
```
Primero nos dice el protocolo, luego la conexión nuestra y despues, a dónde se dirige, esta es la parte interesante ya que se ven claramente los puertos/servicios de
la conexión (443 = https.   25565 = mineacraft) Despues sale el estado de la conexion (ejemplo ESTABILISHED es que se estan mandando datos) y despues el PID o identificador de proceso

Resulta que he encontrado muchas conexiones realizadas por el PID "13988" ¿Pero que es exactamente ese proceso?
```cmd
C:\Users\cucuxii> tasklist /fi "pid eq 13988"
chrome.exe                   13988 Console                    1    43.372 KB
```
Pues era el Chrome.
