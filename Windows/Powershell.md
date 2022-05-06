
##  Archivos

 - **Get-ChildItem** → listar elementos (ls)
	  * -Path → indicar ruta   Get-ChildItem C:\Users\  (se puede omitir el -Path)
   * 	-Name → solo da el nombre    |  -Recurse → recursivo, buscar subdirectorios dentro de las carpetas
   * 	-Attributes → atributos   Directory+!System → busca archivos del sistema que no sean directorios.
   * 	Regex → Get-ChildItem C:\Windows\S* → Sytem32 Systemapps
   * 	-Exclude / -Include *.txt → no muestra archivos txt, solo muestra archivos txt (o cualquier otra extension)

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

