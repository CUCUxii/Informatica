
### Tipos de usuarios en Windows
 - **Administrator** -> El administrador, equivalente al root en linux, superusuario, con máximos privilegios.
 - **Public** -> El usuario que adopta cualquier persona que se conecte (ej, para bajar cosas por smb)

Los datos sobre ese usaurio.
```powershell
PS C:\Users> net user Elliot
User name                    Elliot
Full Name                    Elliot
(...) Muchos mas datos como si el usaurio va a expirar o necesita contraseña, el pais...
Local Group Memberships      *Remote Management Use
Global Group memberships     *Domain Users         *Development
```

Los utlimos datos se refieren a los grupos.

### Tipos de grupos

Con el comando ```net user nombre``` se puede buscar a un usuario y ver sus grupos.

 - Remote management Users -> se pueden concetar por medio del servicio Winrm proporcionando su contraseña.
 - Administrators -> Administradores

----------------------------------------------------------------------------------------

# Registro

El registro de windows es una base de datos con archivos de configuracion (el equivalente en Linux de /etc y archivos *.conf*)
El registro se guarda en System32\config. El registro se organiza por rutas, empezando por la clave principal

Clave principal:
- HKLM → HKEY_LOCAL_MACHINE → configuraciones del sistema local.
- HKU → HKEY_USERS → configuraciones especificas de cada usaurio.
- HKCU → HKEY_CURRENT_USER → configuraciones del usuario actual.

Segunda clave: → HKLM\segunda_clave
- \SOFTWARE → ajustes de software y de Windows
 
Desde consola se puede editar con el programa preinstalado "regedit.exe" 

/v -> Que clave quieres modificar
/t -> que tipo de dato vas a introducir. REG_DWORD es para numeros (ejemplo 1=activado 0=desactivado)
/d -> dato
/f -> force, añade la clave de registro sin preguntar








 
