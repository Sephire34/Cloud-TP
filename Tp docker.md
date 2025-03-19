## Partie 1

**Première étape j ai installé docker en suivant la doc officiel :**
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```      


🌞 **Utiliser la commande `docker run`**

**J ai ensuite crée un conteneur** 

``` bash
Sudo docker run --name web -d -p 9999:80 nginx
```

comme t as demandé j ai lancé avec un partage de port avec le port 9999 redirigé vers le port 80

**J ai donc ensuite obtenue l id du conteneur:**
```
b2c2dc5138425c9d15f4539a0f289e4aa1c0074df490210783cc87aca57be131
```


🌞 **Rendre le service dispo sur internet**

**J'ai rajouté un paramètre dans les réglages réseau pour ajouter le port 9999**

**J'ai donc pu curl et accéder à l'interface web:**
``` bash
curl http:52.224.242.182//:9999
```

``` html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


🌞 **Custom un peu le lancement du conteneur**

Maintenant on vas custom tout ca hein..

déjà je vais crée le répertoire pour stocker la conf et les fichiers du site

``` bash
mkdir -p ~/nginx_custom/html ~/nginx_custom/conf
```

dans le répertoire je met mon petit fichier custom

```bash
echo "<h1>Bienvenue ici tu es chez moi. !</h1>" > ~/nginx_custom/html/index.html
```

bien mtn qu on a fais ca je vais crée un fichier custom conf pour écouter sur le port 7777
``` 
nano ~/nginx_custom/conf/custom.conf
```

et j ai mis la conf que tu nous as donné dedans 
```
server {
    listen 7777;
    root /var/www/tp_docker;
    index index.html;
}
```

bon j avoue que pour cette partie je me suis un peu fait aider, ducoup j ai relancé le conteneur en montant les fichiers personnalisé dedans (j ai du le faire 5 fois avant que ca marche)

```bash
docker run --name meow -d \
  -v ~/nginx_custom/html:/var/www/tp_docker \
  -v $(realpath ~/nginx_custom/conf/custom.conf):/etc/nginx/conf.d/custom.conf \
  -p 8888:7777 \
  --memory="512m" \
  nginx
```


Bon le moment de vérité je curl avec l ip publique et le port 8888 pour voir si le site est accessible 

```bash
curl http://52.224.242.182:8888
```

et sa me retourne bien ma conf perso
```
C:\Users\touff>curl http://52.224.242.182:8888
<h1>Bienvenue ici tu es chez moi. \!</h1>
```


## Partie 2

Je vais commencer par crée le répertoire adequat 

```bash
mkdir ~/apache_custom && cd ~/apache_custom
```

je fais un petit fichier html personnalisé 

```bash
echo "<h1>Bienvenue etre humain</h1>" > index.html
```

On crée ensuite le fichier de conf apache

```
nano apache2.conf
```


et on vas mettre notre petite conf dedans

```bash
Listen 80

LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"

DirectoryIndex index.html
DocumentRoot "/var/www/html/"

ErrorLog "/var/log/apache2/error.log"
LogLevel warn
```


Maintenant on passe à notre petit docker file on vas donc crée le fichier

```
nano Dockerfile
```

on met notre conf dedans 

```bash
FROM debian:latest

RUN apt update -y && apt install -y apache2

COPY index.html /var/www/html/index.html

COPY apache2.conf /etc/apache2/apache2.conf

CMD [ "/usr/sbin/apache2ctl", "-D", "FOREGROUND" ]
```

on crée mtn l image personnalisé

```bash
docker build . -t my_apache
```

des que le build se finis j obtient ceci

```
root@test:~/apache_custom# docker build . -t my_apache
[+] Building 29.2s (9/9) FINISHED                                                                                          docker:default
 => [internal] load build definition from Dockerfile                                                                                 0.0s
 => => transferring dockerfile: 242B                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/debian:latest                                                                     0.1s
 => [internal] load .dockerignore                                                                                                    0.0s
 => => transferring context: 2B                                                                                                      0.0s
 => [1/4] FROM docker.io/library/debian:latest@sha256:35286826a88dc879b4f438b645ba574a55a14187b483d09213a024dc0c0a64ed               3.9s
 => => resolve docker.io/library/debian:latest@sha256:35286826a88dc879b4f438b645ba574a55a14187b483d09213a024dc0c0a64ed               0.1s
 => => sha256:0cc7b2132ebf8d535c3a8efd12be46902dd58c496dae66c478a6e64d9588a38d 1.02kB / 1.02kB                                       0.0s
 => => sha256:d4ccddb816ba27eaae22ef3d56175d53f47998e2acb99df1ae0e5b426b28a076 453B / 453B                                           0.0s
 => => sha256:155ad54a8b2812a0ec559ff82c0c6f0f0dddb337a226b11879f09e15f67b69fc 48.48MB / 48.48MB                                     1.1s
 => => sha256:35286826a88dc879b4f438b645ba574a55a14187b483d09213a024dc0c0a64ed 8.52kB / 8.52kB                                       0.0s
 => => extracting sha256:155ad54a8b2812a0ec559ff82c0c6f0f0dddb337a226b11879f09e15f67b69fc                                            2.1s
 => [internal] load build context                                                                                                    0.0s
 => => transferring context: 104B                                                                                                    0.0s
 => [2/4] RUN apt update -y && apt install -y apache2                                                                               21.1s
 => [3/4] COPY index.html /var/www/html/index.html                                                                                   0.2s
 => [4/4] COPY apache2.conf /etc/apache2/apache2.conf                                                                                0.1s
 => exporting to image                                                                                                               3.5s
 => => exporting layers                                                                                                              3.4s
 => => writing image sha256:e58291beb4b188866a02ff1b25b68bc5f9f95f2b68e804e48bae6815bbe752d8                                         0.0s
 => => naming to docker.io/library/my_apache    
```

le build a donc fonctionné

je vérifie ensuite que l image a bien était crée

```
root@test:~/apache_custom# docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
my_apache    latest    e58291beb4b1   About a minute ago   253MB
nginx        latest    b52e0b094bc0   5 weeks ago          192MB
```

L image est bien présente.

Maintenant je lance mon conteneur

```bash
docker run --name apache_custom -d -p 8080:80 my_apache
```

Je curl pour vérifié l'accès

```bash
root@test:~/apache_custom# curl http://52.224.242.182:8080
<h1>Bienvenue etre humain</h1>
```

Le site est accessible tout fonctionne (j aime tellement quand ça marche)



---


## Partie 3




🌞 Installation de WikiJS

Je crée le dossier et le fichier docker-compose.yml:

```bash
mkdir ~/wikijs && cd ~/wikijs nano docker-compose.yml
```


**Contenu du fichier** :

```bash
version: "3.8"

services:
  db:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"

  web:
    build: .
    restart: always
    ports:
      - "8888:8888"
    depends_on:
      - db
    volumes:
      - .:/app
```



J'exécute ensuite la commande pour lancer WikiJS :

```bash
docker compose up -d
```

Après quelques secondes, les conteneurs sont bien démarrés :

```bash
docker ps
```

J'obtiens :

```
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                      NAMES
b57405726fac   requarks/wiki:latest   "docker-entrypoint.s…"   30 seconds ago   Up 29 seconds   0.0.0.0:3000->3000/tcp, 3443/tcp          wikijs-wikijs-1
f79ed6f28f86   postgres:13            "docker-entrypoint.s…"   30 seconds ago   Up 29 seconds   5432/tcp                                  wikijs-db-1
```

J'ouvre ensuite mon navigateur et me rends à l'adresse :

```
http://52.224.242.182:3000
```

La page d'installation s'affiche et je configure mon compte administrateur avec mon email et un mot de passe.  

---

### 🌞 **Déploiement de l'application Flask + Redis**

Je crée le dossier et j'y place l'application Flask et le fichier `docker-compose.yml` :

```bash
mkdir ~/flask_redis && cd ~/flask_redis
nano docker-compose.yml
```

#### **Contenu du fichier :**

```yaml
version: "3.8"

services:
  db:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"

  web:
    build: .
    restart: always
    ports:
      - "8888:8888"
    depends_on:
      - db
    volumes:
      - .:/app
```

Je crée ensuite le fichier `app.py` :

```bash
nano app.py
```

#### **Contenu du fichier :**

```python
import redis
import time
import socket
from flask import Flask, request, render_template

time.sleep(5)

r = redis.StrictRedis(host='db', port=6379, db=0)

app = Flask(__name__)

@app.route('/')
@app.route('/index')
def index():
    hostname = socket.gethostname()
    return f"<h1>Ici le roi c'est moi</h1><p>Hébergé sur : {hostname}</p>"

@app.route('/add', methods=['POST'])
def add():
    r.set(request.form['key'], request.form['value'])
    return f'Successfully added key {request.form["key"]}'

@app.route('/get', methods=['POST'])
def get():
    key = request.form['key']
    value = r.get(key)
    if value:
        return f'You asked about key {key}. Value: {value.decode("utf-8")}'
    return f'Key {key} does not exist.'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888)
```

Ensuite, je crée un fichier `Dockerfile` :

```bash
nano Dockerfile
```

#### **Contenu du fichier :**

```dockerfile
FROM python:3.9

WORKDIR /app

COPY app.py .
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

Je crée enfin un fichier `requirements.txt` :

```bash
nano requirements.txt
```

#### **Contenu du fichier :**

```
flask
redis
```

Je construis l’image et je lance les conteneurs :

```bash
docker compose up -d --build
```

Après quelques secondes, je vérifie que les conteneurs tournent :

```bash
docker ps
```

Résultat :

```
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                      NAMES
a8b54595a1e3   flask_redis-web        "python app.py"          29 hours ago     Up 24 minutes   0.0.0.0:8888->8888/tcp                     flask_redis-web-1
f038c0f02160   redis:latest           "docker-entrypoint.s…"   29 hours ago     Up 24 minutes   0.0.0.0:6379->6379/tcp                     flask_redis-db-1
```

---

### **Test de l’application Flask + Redis**

J’accède au site via :

```
http://52.224.242.182:8888
```

Cela affiche :

```
Ici le roi c'est moi
Hébergé sur : flask_redis-web-1
```

Je teste ensuite l’ajout et la récupération de données via Redis :

```bash
curl -X POST -d "key=test&value=FlaskOK" http://52.224.242.182:8888/add
curl -X POST -d "key=test" http://52.224.242.182:8888/get
```

Si tout fonctionne bien, j’obtiens :

```
Successfully added key test
You asked about key test. Value: FlaskOK
```

---

## Partie 3




🌞 Installation de WikiJS

Je crée le dossier et le fichier docker-compose.yml:

```bash
mkdir ~/wikijs && cd ~/wikijs nano docker-compose.yml
```


**Contenu du fichier** :

```bash
version: "3.8"

services:
  db:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"

  web:
    build: .
    restart: always
    ports:
      - "8888:8888"
    depends_on:
      - db
    volumes:
      - .:/app
```



J'exécute ensuite la commande pour lancer WikiJS :

```bash
docker compose up -d
```

Après quelques secondes, les conteneurs sont bien démarrés :

```bash
docker ps
```

J'obtiens :

```
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                      NAMES
b57405726fac   requarks/wiki:latest   "docker-entrypoint.s…"   30 seconds ago   Up 29 seconds   0.0.0.0:3000->3000/tcp, 3443/tcp          wikijs-wikijs-1
f79ed6f28f86   postgres:13            "docker-entrypoint.s…"   30 seconds ago   Up 29 seconds   5432/tcp                                  wikijs-db-1
```

J'ouvre ensuite mon navigateur et me rends à l'adresse :

```
http://52.224.242.182:3000
```

La page d'installation s'affiche et je configure mon compte administrateur avec mon email et un mot de passe.  

---

### 🌞 **Déploiement de l'application Flask + Redis**

Je crée le dossier et j'y place l'application Flask et le fichier `docker-compose.yml` :

```bash
mkdir ~/flask_redis && cd ~/flask_redis
nano docker-compose.yml
```

#### **Contenu du fichier :**

```yaml
version: "3.8"

services:
  db:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"

  web:
    build: .
    restart: always
    ports:
      - "8888:8888"
    depends_on:
      - db
    volumes:
      - .:/app
```

Je crée ensuite le fichier `app.py` :

```bash
nano app.py
```

#### **Contenu du fichier :**

```python
import redis
import time
import socket
from flask import Flask, request, render_template

time.sleep(5)

r = redis.StrictRedis(host='db', port=6379, db=0)

app = Flask(__name__)

@app.route('/')
@app.route('/index')
def index():
    hostname = socket.gethostname()
    return f"<h1>Ici le roi c'est moi</h1><p>Hébergé sur : {hostname}</p>"

@app.route('/add', methods=['POST'])
def add():
    r.set(request.form['key'], request.form['value'])
    return f'Successfully added key {request.form["key"]}'

@app.route('/get', methods=['POST'])
def get():
    key = request.form['key']
    value = r.get(key)
    if value:
        return f'You asked about key {key}. Value: {value.decode("utf-8")}'
    return f'Key {key} does not exist.'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888)
```

Ensuite, je crée un fichier `Dockerfile` :

```bash
nano Dockerfile
```

#### **Contenu du fichier :**

```dockerfile
FROM python:3.9

WORKDIR /app

COPY app.py .
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

Je crée enfin un fichier `requirements.txt` :

```bash
nano requirements.txt
```

#### **Contenu du fichier :**

```
flask
redis
```

Je construis l’image et je lance les conteneurs :

```bash
docker compose up -d --build
```

Après quelques secondes, je vérifie que les conteneurs tournent :

```bash
docker ps
```

Résultat :

```
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                      NAMES
a8b54595a1e3   flask_redis-web        "python app.py"          29 hours ago     Up 24 minutes   0.0.0.0:8888->8888/tcp                     flask_redis-web-1
f038c0f02160   redis:latest           "docker-entrypoint.s…"   29 hours ago     Up 24 minutes   0.0.0.0:6379->6379/tcp                     flask_redis-db-1
```

---

### **Test de l’application Flask + Redis**

J’accède au site via :

```
http://52.224.242.182:8888
```

Cela affiche :

```
Ici le roi c'est moi
Hébergé sur : flask_redis-web-1
```

Je teste ensuite l’ajout et la récupération de données via Redis :

```bash
curl -X POST -d "key=test&value=FlaskOK" http://52.224.242.182:8888/add
curl -X POST -d "key=test" http://52.224.242.182:8888/get
```

Si tout fonctionne bien, j’obtiens :

```
Successfully added key test
You asked about key test. Value: FlaskOK
```

---


## Partie 4

### 1. Le groupe Docker 

J'ai vérifié les groupes auxquels appartient mon utilisateur :

```bash
ubuntu@test:~$ groups
ubuntu adm cdrom sudo dip lxd
```

J'ai ajouté mon utilisateur au groupe Docker :

```bash
sudo usermod -aG docker $(whoami)
```

Après une reconnexion, j'ai confirmé que l'ajout avait fonctionné :

```bash
ubuntu@test:~$ groups
ubuntu adm cdrom sudo dip lxd docker
```

Puis j'ai vérifié que Docker fonctionnait sans `sudo` :

```bash
ubuntu@test:~$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS
                                NAMES
b57405726fac   requarks/wiki:latest   "docker-entrypoint.s…"   20 minutes ago   Up 20 minutes   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp, 3443/tcp   wikijs-wikijs-1
f79ed6f28f86   postgres:13            "docker-entrypoint.s…"   20 minutes ago   Up 20 minutes   5432/tcp
                                wikijs-db-1
a8b54595a1e3   flask_redis-web        "python app.py"          29 hours ago     Up 44 minutes   0.0.0.0:8888->8888/tcp, [::]:8888->8888/tcp             flask_redis-web-1
f038c0f02160   redis:latest           "docker-entrypoint.s…"   29 hours ago     Up 44 minutes   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp             flask_redis-db-1
```

#### 🌞 Preuve d'élévation de privilèges via Docker

J'ai lancé un conteneur privilégié qui a accès à la racine de l'hôte :

```bash
docker run --rm -v /:/mnt --privileged -it alpine chroot /mnt sh
```

Dans ce shell, j'ai vérifié mon niveau d'accès :

```bash
# whoami
root
```

J'ai aussi pu lire des fichiers normalement protégés :

```bash
# cat /etc/shadow
root:*:20140:0:99999:7:::
daemon:*:20140:0:99999:7:::
bin:*:20140:0:99999:7:::
...
ubuntu:$6$p4eYPoDDI6$N.2wGw1R9GQ8X/yPeRHGfoyh3nI/HFIz6JJnG0PPesaWtkqffAhKc65GOFT0PJykXRbD.4Fw.pjdz2LUVDs.R.:20162:0:99999:7:::
```

 Docker me permet  donc une escalade de privilèges à root hehehe.

---

### 2. Scan de vulnérabilités avec Trivy

#### 🌞 Installation de Trivy

J'ai installé Trivy pour scanner les images utilisées :

```bash
sudo apt install -y trivy
```

#### 🌞 Scan des images

J'ai scanné l'image de **WikiJS** :

```bash
trivy image requarks/wiki:latest
```

J'ai scanné l'image de **PostgreSQL** utilisée par WikiJS :

```bash
trivy image postgres:13
```

J'ai scanné l'image d'**Apache** que j'avais buildée :

```bash
trivy image my_apache
```

Enfin, j'ai scanné l'image **NGINX officielle** utilisée dans la première partie :

```bash
trivy image nginx:latest
```


---

### 3. Benchmark de sécurité Docker

J'ai téléchargé l'outil Docker Bench for Security :

```bash
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
```

J'ai exécuté le benchmark :

```bash
sudo sh docker-bench-security.sh
```

