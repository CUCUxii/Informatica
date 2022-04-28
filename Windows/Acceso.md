

# Puerto 445 TCP -> Descarga de archivos

En la máquina [Timelapse] (https://app.hackthebox.com/machines/Timelapse) cuya IP es 10.10.11.152 tenemos el puerto smb abierto:

Este puerto permite compartir archivos de manera remota, es decir metes la IP, un usaurio y contraseña y te bajas cosas de ese ordenador que estén compartidas por smb.
Puede que no tengamos tal usuario ni contraseña, por lo que solo podremos acceder a los que esté compartido de manera pública.

Primero con **crackmpaexec**, haremos un reconocimiento inicial del servicio

```console
[cucuxii]-[~/Timelapse]:$ crackmapexec smb 10.10.11.152
SMB         10.10.11.152    445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:False)
```
Hay dos herramientas para bajarnos los archivos: tanto **smbmap** que lo permite visualizar todo de manera más clara como **smbclient** que es el oficial.

```console
[cucuxii]-[~/Timelapse]:$ smbmap -H 10.10.11.152 -u "test" -> Usuario inventado, para ver que hay si no tienes ningun usuario ni contraseña.
[+] Guest session   	IP: 10.10.11.152:445	Name: timelapse.htb     -> O sea sin usuario ni contraseña "invitado"                                
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	Shares                                            	READ ONLY	
	SYSVOL                                            	NO ACCESS	Logon server share 
[cucuxii]-[~/Timelapse]:$ smbmap -H 10.10.11.152 -u "test" -r "Shares"
[+] Guest session   	IP: 10.10.11.152:445	Name: timelapse.htb                                     
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Shares                                            	READ ONLY	
	dr--r--r--                0 Mon Oct 25 21:40:06 2021	Dev
	dr--r--r--                0 Mon Oct 25 17:55:14 2021	HelpDesk
```
Nos hemos metido en la carpeta Shares, ya que el resto que tiene el "$" después son cosas del sistema que no interesan mucho. Ahora tiramos de smbclient para que
nos de una consola y bajarnos los archivos.

```console
[cucuxii]-[~/Timelapse]:$ smbclient "\\\10.10.11.152\Shares" -N -> "-N" es null session, o sea como invitado (no tenemos contraseñas) 
smb: \> dir
  Dev                                 D        0  Mon Oct 25 21:40:06 2021
  HelpDesk                            D        0  Mon Oct 25 17:48:42 2021
smb: \> cd Dev
smb: \Dev\> dir
  winrm_backup.zip                    A     2611  Mon Oct 25 17:46:42 2021
smb: \Dev\> get winrm_backup.zip 
getting file \Dev\winrm_backup.zip of size 2611 as winrm_backup.zip (2,9 KiloBytes/sec) (average 482,5 KiloBytes/sec)
```
