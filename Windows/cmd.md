# Guía de cmd (linea de comandos)


---------------------------------------------------------------------------
## Lo Básico: movimiento

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


---------------------------------------------------------------------------

## Filtro -> 
-  **Findstr** -> comando para filtrar la salida.

findstr patron
```cmd
C:\Users\cucuxii> dir C:\Windows\System32
Muuuchos archivos y carpetas...
C:\Users\cucuxii> dir C:\Windows\System32 | findstr "cmd"
15/01/2021  11:12           289.792 cmd.exe
07/12/2019  11:09            20.480 cmdkey.exe
```
findstr "patron1 patron2"-> Busca cosas por cada patrón (ejemplo salen tanto las busquedas de 'cmd' como de 'config')
```cmd
C:\Users\cucuxii> dir C:\Windows\System32 |findstr "cmd config"
15/01/2021  11:12           289.792 cmd.exe
07/12/2019  11:09            20.480 cmdkey.exe 
22/04/2022  09:38    <DIR>          config
11/02/2022  11:41           215.896 coreglobconfig.dll
```
findstr /I "patron" -> busquedas case insentive, ignora si son mayusculas o minusculas (es decir sale tanto patron como PATRON como Patron...)
```cmd
C:\Users\cucuxii> dir C:\Windows\System32 | findstr "device"
16/04/2021  19:44           231.248 containerdevicemanagement.dll
15/01/2021  11:11           240.688 deviceaccess.dll
C:\Users\cucuxii> dir C:\Windows\System32 | findstr /I "device"
16/04/2021  19:44           231.248 containerdevicemanagement.dll
15/01/2021  11:11           240.688 deviceaccess.dll
07/12/2019  11:08            21.184 DefaultDeviceManager.dll
```
findstr /V "patron" -> justo que no salga ese patrón
```cmd
C:\Users\cucuxii\Documents\carpeta> dir 
10/11/2020  20:24                   Archivo1.txt
10/11/2020  20:24                   Archivo2.txt
10/11/2020  20:24    <DIR>          Carpeta
C:\Users\cucuxii\Documents\carpeta> dir | findstr /V "Archivo"
10/11/2020  20:24    <DIR>          Carpeta
```
