
# [Under the wire - century](https://underthewire.tech/century)

Estos retos están enfocados al manejo de Powershell, el lenguaje de comandos/programación de Windows (junto a cmd), equivalente a Bash en linux

-----------------------

## Nivel 1 

Este nivel es simplemente loguearse con la contraseña century1 usuario century1. El sistema es un poco lento.
```console
[cucuxii]:$ sshpass -p "century1" ssh century1@century.underthewire.tech
```
Luego te pide que veas la version de Powershell que se está empleando
```powershell
PS C:\users\century1\desktop> $PSVersionTable
BuildVersion                   10.0.14393.5127  
```
Credenciales siguiente nivel -> century2:10.0.14393.5127  

-----------------------

## Nivel 2

Aqui te pide que busques el Alias del comando que hace de wget y el nombre del archivo del escritorio
```powershell
PS C:\users\century2\desktop> Get-Alias "wget"
Alias           wget -> Invoke-WebRequest                                  
PS C:\users\century2\desktop> (Get-ChildItem .).Name # -> 443
```
El comando Get-Alias es para ver los alias, o sea nombres alternativos personalizados para los comandos, y **Get-ChildItem** lista los archivos, se pone
entre parentesis y .Name para que solo muestre el nombre que es lo que nos interesa. La salida de comandos de Powershell son objetos con propiedades, 
asi que accedemos a una en concreto o nos salen todas:
```powershell
PS C:\users\century2\desktop> Get-ChildItem .
Mode                LastWriteTime         Length Name                                                                                                                                                         
-a----        8/30/2018   3:29 AM            693 443  
```
Credenciales siguiente nivel -> century3:invoke-webrequest443

-----------------------

## Nivel 3

Aqui nos piden que contemos cuantos archivos hay en en el Escritorio, El comando **measure-object** sirve para hacer conteos de cosas. En la salida tiene
la propiedad "Count" para que ponga el numero de elementos

```powershell
PS C:\users\century3\desktop> (Get-ChildItem -File -Path "." | Measure-Object).Count # -> 123
```
Credenciales siguiente nivel -> century4:123

-----------------------

## Nivel 4

Aqui entra la clasica jugarreta del nombre con espacios, en concreto de una carpeta. Hay que ver como se llama el archivo que reside en su interior.
Cuando le pasas algo con espacios a un comando lo suele interpretar como varias ordenes en vez de una, por lo que da problemas, asi que hay que ponerlo
entre comillas para que lo vea como algo unico.

```powershell
PS C:\users\century4\desktop> Get-ChildItem
d-----        6/23/2022  10:30 PM                Can You Open Me                                                                    
PS C:\users\century4\desktop> (Get-ChildItem  ".\Can You Open Me").Name  # -> 49125                         
```
Credenciales siguiente nivel -> century5:49125

-----------------------

## Nivel 5

Nos pide el nombre del dominio más el nombre del archivo que hay en el escritorio. Cundo tenemos que buscar información sobre el porpio sistema,
entra en juego el comando **Get-WmiObject**. Tambien vale echo $env:USERDOMAIN para que te muestre la varaible de entorno del nombre de dominio.

```powershell
PS C:\users\century5\desktop> (Get-WmiObject Win32_ComputerSystem).Domain # -> underthewire.tech
PS C:\users\century5\desktop> echo $env:USERDOMAIN # -> underthewire
PS C:\users\century5\desktop> (Get-ChildItem).Name # -> 3347                                        
```
Credenciales siguiente nivel -> century6:underthewire3347

-----------------------

## Nivel 6

Este nivel es casi identico al 3, o sea contar pero esta vez carpetas... el mismo comando funciona perfecto.
```powershell
PS C:\users\century6\desktop> (Get-ChildItem -Path "." | Measure-Object).count # -> 197
```

Credenciales siguiente nivel -> century7:197

-----------------------

## Nivel 7

Aqui nos pide que busquemos por toda la carpeta del usaurio century6 (o sea el conjunto de contacts, desktop, documents, downloads, favorites, music y videos).
un archivo que se llame "readme" . La manera idonea de resolver esto es con la busqueda recursiva, una vez encontrado, se pasa a **Get.Content** para que 
lo escriba mediante una tuberia "|"

```powershell
PS C:\users\century7\desktop> Get-CHildItem -Recurse -Filter "*readme*" -Path . | Get-Content # ->  7points
```
Credenciales siguiente nivel -> century8:7points

-----------------------

## Nivel 8

Aqui nos pide que contemos el numero de cadenas unicas que hay en el archivo "Unique.txt" del escritorio
```powershell
PS C:\users\century8\desktop> (Get-Content .\unique.txt | Sort-Object -Unique | Measure-Object).Count # -> 696  
```                               
El Get-Content le pasa todo el texto al comando **Sort-Object** que con la opción Unique te muestra solo cadenas que salgan una vez, esto te muestra todas
esas palabras, pero nos interesa su conteo, asi que se le pasa a **Measure-Object**

Credenciales siguiente nivel -> century9:696

-----------------------

## Nivel 9

En este nivel te dan un archivo de texto con palabras random y te piden que cojas la numero 161. Como se separan por espacios, hay que hacer que se 
separen por saltos de linea para aplicar el conteo. Eso se hace con "split '' "
```powershell
PS C:\users\century9\desktop> (Get-Content .\Word_File.txt) -split ' ' | Select -First 161 
``` 
Credenciales siguiente nivel -> century10:pierid

-----------------------

## Nivel 10

Nos pide la octava y decima palabra de la descripcion del servicio de Actualizacion de Windows
```powershell
PS C:\users\century10\desktop> (Get-Service -DisplayName "*Update*").DisplayName # -> Windows Update
PS C:\users\century10\desktop> (Get-WmiObject win32_Service -Filter "DisplayName='Windows Update'").Description
Enables the detection, download, and installation of updates for Windows and other programs. If this service is disabled, users of t
his computer will not be able to use Windows Update or its automatic updating feature, and programs will not be able to use the Wind
ows Update Agent (WUA) API.
PS C:\users\century10\desktop> (Get-ChildItem) # -> 110
``` 
Credenciales siguiente nivel -> century11:windowsupdates110

-----------------------

## Nivel 11

Nos pide el nombre del archivo oculto que hay en todo century11

```powershell
PS C:\users\century11\desktop> (Get-ChildItem ..\ -Force -Hidden -Recurse -ErrorAction SilentlyContinue).Name # -> secret_sauce
``` 
A parte de este salian mas, pero este era el que mas llamaba la atencion y el ultimo.

Credenciales siguiente nivel -> century12:secret_sauce

-----------------------

## Nivel 12

Nos piden la descripción del ordenador que es Controlador de Dominio (ordenador que administra un grupo empresarial) y el nombre del archivo del
escritorio.
```powershell
PS C:\users\century12\desktop> (Get-ChildItem).Name #  -> 9 _things
PS C:\users\century12\desktop> (Get-ADDomainController).Name  # -> UTW
PS C:\users\century12\desktop> Get-ADComputer UTW -Properties Description # -> i_authenticate_things
``` 
-----------------------

## Nivel 13

Nos dan un archivo de texto que contiene muchas palabras, hay que contarlas. Para eso el **Meausre_Object** pero con otro parámetro (si no cuenta lineas)
```powershell
PS C:\users\century13\desktop> (Get-ChildItem | Get-Content | Measure-Object -Word).Words  # -> 755           
```
-----------------------

## Nivel 14

Este es el ultimo reto. Nos dan un archivo con muchas palabras y hay que contar todos los "polo"s que salgan. El split '' nos lo parte en lineas y 
sls (Select-String) nos filtra polos. El regex es para filtrar palabras que contengan *polo* como *carpology* diciendo que quiero filtrar todas las 
lineas que empiezen y acaben por *polo*

```powershell
PS C:\users\century14\desktop> ((Get-Content .\countpolos) -split ' ' | sls "^polo$").Count  # -> 153
```


