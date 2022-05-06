
##  Archivos

 - **Get-ChildItem** → listar elementos (ls)
   *   **-Path** → indicar ruta   Get-ChildItem C:\Users\  (se puede omitir el -Path)
   *   **-Name** → solo da el nombre    
   *   **-Recurse** → recursivo, buscar subdirectorios dentro de las carpetas
   *   **-Attributes** → atributos   Directory+!System → busca archivos del sistema que no sean directorios.
   *   **-Filter** -> buscar algo en concreto
   *   **-Regex** → Get-ChildItem C:\Windows\S* → Sytem32 Systemapps
   *   **-Exclude / -Include \*.txt**→ no muestra archivos txt, solo muestra archivos txt (o cualquier otra extension)

Buscar un archivo en concreto en el sistema (desde cierta ruta) y que te diga su localización.
```powershell
PS C:\Users\cucuxii> Get-ChildItem -Recurse -Filter "Vocabulario.txt" -Name
C:\Users\cucuxii\Desktop\JAPONÉS\vocabulario.txt
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------

##  Objetos

Cuando ejecutas un comando en Powershell (ej, Get-ChildItem) te sale un output en formato objeto  (como una tabla, cada elemento tiene sus propiedades (Columnas))  
*Ejemplo, con Get-ChildItem los objetos Archivo1.txt y Archivo2.txt tienen las propiedades "Mode", "Name", "LastWriteTime" y "Lenght"*   

 - **Get-Member** -> Muestra las propiedades que puede tener la salida. 
   * Get-Member -Name C* → Cancel, Close, Create...
 - **Select-Object** -> Para mostrar la salida como quieras
   * Get-ChilItem | Select-Object Name,BaseName -> Mostrar solo esas propiedades.
   * Get-ChilItem | Select-Object -First 3 → Muestra tres primeras lineas de la salida.
 - **Short-Object** -> Ordenar por...
   * Sort-Object -Ascending -Porperty Name  “Nombre” por orden alfabetico 
 - **Where-Object** -> Filtrar por valor →  Where-Object Propiedad comparador numero
   * -eq (=) | -lt (<) | -gt (>) |-ne (!=)  Ej Where-Object Name -eq  ‘svchost’ 
 - **Where** -> where {$_.Propiedad filtro}. El "$_" se refiere a que se aplica para cada elemento de la salida
   *  Get-ChildItem | where {$_.Name -like "Do*"} ->  Documents, Downloads       
   *  Get-Process | where {$_.ProcessName -notmatch "svchost|chrome|lenovo"}   -> Varios parámetros, en este caso que el nombre no coincida con svchost, chrome...
   *  Get-Process | where {$_.Handles -gt 1000} 
   *  Get-Process | where {$_.Handles -gt 1000 -and $_.ProcessName -match “svchost|chrome”} -> Que se cumplan varios parámetros

```powershell
PS C:\Users\cucuxii\Documents\carpeta > Get-ChildItem
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---        23/10/2020     21:10                Archivo1.txt
-ar---        05/05/2022     18:21                Archivo1.txt
PS C:\Users\cucuxii\Documents\carpeta> Get-ChildItem  | Get-Member
   TypeName: System.IO.DirectoryInfo
Name                      MemberType     Definition
----                      ----------     ----------
Mode                      CodeProperty   System.String Mode{get=Mode;}
Name                      Property       string Name {get;}
PS C:\Users\cucuxii\Documents\carpeta> Get-ChildItem  | Select-Object Name, BaseName
Name            Mode
----            ----
Archivo1.txt    -a----
Archivo2.txt    -a----
PS C:\Users\cucuxii> Get-Process | Where-Object Name -eq  'svchost' | Select-Object -First 2
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    301      16     3896      14728               508   0 svchost
   1776      23    12700      23976              1320   0 svchost
PS C:\Users\palki> Get-Process | where {$_.ProcessName -notlike "svc"} | Select-Object -First 2
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    162       9     2188       7420              6444   0 AdminService
    205      12     2496       9872       0,20   6456   1 AdobeIPCBroker
```

--------------------------------------------------------------------

##  Redes

 - **Get-NetTCPConnection** -> Para sacar las conexiones por TCP
```powershell
PS C:\Users\palki>  Get-NetTCPConnection -State Listen,Established
LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting OwningProcess
------------                        --------- -------------                       ---------- -----       -------------- -------------
192.168.0.236                       65292     95.100.100.200                      443        Established Internet       12172
192.168.0.236                       53355     54.190.100.200                      443        Established Internet       3288
192.168.0.236                       53353     198.50.200.200                      25565      Established Internet       3288
0.0.0.0                             49670     0.0.0.0                             0          Listen                     308
0.0.0.0                             49668     0.0.0.0                             0          Listen                     6004
```

Había muchas mas pero para resumir. De la salida de este comando, lo que interesa es el RemotePort ya que es el puerto de salida,
por el que hace las conexiones, y que nos permite ver mas claramente de que tipo son. 
*Por ejemplo las 443 son las que usa https y el 25565 en este caso es del minecraft que lo dejé abierto a la hora de hacer el comando*

