
Obtener información del usuario actual

```powershell
PS C:\Users\legacyy\Documents> whoami /user
User Name         SID
================= ============================================
timelapse\Elliot S-1-5-21-671920749-559770252-3318990721-1603

PS C:\Users\legacyy\Documents> whoami /priv
Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

Mirar los usuarios que hay:

```powershell
PS C:\Users\legacyy\Documents> cd C:\Users\
PS C:\Users> ls
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       10/23/2021  11:27 AM                Administrator  
d-----       10/25/2021   8:22 AM                Elliot 
d-r---       10/23/2021  11:27 AM                Public    
d-----       10/25/2021  12:23 PM                svc_deploy
d-----        2/23/2022   5:45 PM                TRX
```
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

