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
-  **Findstr**
     * findstr 
```cmd
C:\Users\cucuxii> dir C:\Windows\System32
Muuuchos archivos y carpetas...
C:\Users\cucuxii> dir C:\Windows\System32 | findstr "cmd.exe"
15/01/2021  11:12           289.792 cmd.exe
18/09/2021  19:22           468.992 dsregcmd.exe
```




