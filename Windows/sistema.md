
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










 
