#  Exercice 10 :  Démarrez avec Docker Compose 

### Informations
- Évaluation : formative.
- Durée estimée : 2 heures.
- Système d'exploitation : Ubuntu client ou Windows.
- Environnement : Docker.  

### Objectifs  

- Analyser les différents scénarios de déploiement proposés dans les documents de conception.  
- Distinguer les services à installer sur les serveurs.  
- Distinguer les services à installer sur le réseau.  
- Déterminer les étapes à entreprendre pour installer et configurer les services réseau. 

## Section 1 : commandes de base

Dans cette section, vous allez utiliser les fichiers du document [exercice10.zip](extra/exercice10.zip).

### Étape 1  

1- Décompressez le document <code>exercice10.zip</code>.

2- À l’aide d’un éditeur de texte (je recommande Visual Studio Code), ouvrer les fichiers dans le répertoire.  
Observer le fichier <code>compose.yaml</code>.  
La partie <code>services</code> décrit les conteneurs que nous allons utiliser.  
Le premier conteneur va se nommer <code>proxy</code>, nous allons utiliser l’image alpine de nginx et le port 80 sera exposé sur l’hôte.

```yaml
  proxy:
    image: nginx:alpine # on utilise la version alpine
    ports:
      - '80:80' # expose 80 a hote envoie a 80 au conteneur
```

Ici, nous utilisons le paramètre <code>volumes</code> pour faire un bind mount du fichier <code>nginx.conf</code> dans notre hôte au fichier <code>/etc/nginx/conf.d/default.conf</code> dans le conteneur. Le fichier sera en lecture seulement dans le conteneur.

```yaml
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

Le deuxième conteneur va se nommer <code>serveurweb</code> et nous allons utiliser l’image la plus récente d’Apache.  

```yaml
serveurweb:
    image: httpd  # on utilise httpd:latest
```

<details>
	<summary>Observer le fichier <code>nginx.conf</code>. Pouvez-vous trouver à quel endroit on utilise le DNS de Docker pour référencer notre deuxième conteneur ?(vous n’avez pas à comprendre le fichier <code>nginx.conf</code>) </summary>

Avec le paramètre <code>proxy_pass</code>. Il référence le nom docker DNS du fichier docker compose.
	
```bash
proxy_pass         http://serveurweb;
```
</details>

3- Lancer ces conteneurs. Vous devez vous placer dans le répertoire des fichiers pour les lancer (avec Visual Studio, vous pouvez ouvrir un shell dans l’éditeur de texte).

```bash
docker compose up
```

Avec cette commande, vous allez lancer vos conteneurs, créer un réseau privé virtuel, bind mount le fichier spécifié, ouvert le port sur l’hôte et envoyer les journaux à l’écran.

Ouvrez un navigateur et allez à http://localhost. Oui, vous êtes dans le serveur Apache et non nginx. Nginx sert de proxy ici. Si vous consultez vos journaux à la ligne de commande, vous pouvez voir que le <code>proxy</code> est appelé en premier et après le serveurweb.

Arrêter vos conteneurs.

```bash
Ctrl-c
```

4- Relancer vos conteneurs, mais en arrière-plan et vérifier le fonctionnement avec un navigateur Web.

```bash
docker compose up -d
```

Vérifier les commandes disponibles avec docker compose.

```bash
docker compose --help
```

Vous pouvez constater que de nombreuses commandes que vous avez déjà utilisées se retrouvent dans docker compose.

Consulter les journaux.

```bash
docker compose logs
```

Lister les conteneurs qui s’exécutent.

```bash
docker compose ps
```

Lister les services qui s’exécutent dans les conteneurs.

```bash
docker compose top
```

Finalement, arrêter vos conteneurs.

```bash
docker compose down
```

## Section 2 : application complète

Source : [Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)

Sur cette page, vous construisez une application web simple en Python  fonctionnant sur Docker Compose. L'application utilise le framework Flask et maintient un compteur de hits dans Redis. Bien que l'exemple utilise Python, les concepts démontrés ici devraient être compréhensibles même si vous n'êtes pas familier avec ce langage.

### Conditions préalables : 

Assurez-vous que vous avez déjà installé Docker Engine et Docker Compose. Vous n'avez pas besoin d'installer Python ou Redis, car ils sont tous deux fournis par les images Docker.

### Étape 1 : Configuration

Définissez les dépendances de l'application.

1- Créez un répertoire pour le projet :

```bash
 mkdir composetest
 cd composetest
```  

2- Créez un fichier appelé app.py dans le répertoire de votre projet et collez-y ceci :

```python
import time

import redis 
from flask import Flask 

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

Dans cet exemple, redis est le nom d'hôte du conteneur redis sur le réseau de l'application. Nous utilisons le port par défaut de Redis, 6379.

>Gestion des erreurs transitoires  
Notez la façon dont la fonction get\_hit\_count est écrite. Cette boucle de réessaie de base nous permet de tenter notre requête plusieurs fois si le service Redis n'est pas disponible. Ceci est utile au démarrage lorsque l'application est en ligne, mais rend également notre application plus résiliente si le service Redis doit être redémarré à tout moment pendant la durée de vie de l'application. Dans un cluster, cela permet également de gérer les coupures momentanées de connexion entre les nœuds.   

3- Créez un autre fichier appelé `requirements.txt` dans le répertoire de votre projet et collez-y le code suivant :

```python
flask
redis
```

4- Vous allez créer un Dockerfile qui construit une image Docker. L'image contient toutes les dépendances dont l'application Python a besoin, y compris Python lui-même.

Dans le répertoire de votre projet, créez un fichier nommé `Dockerfile` et collez ce qui suit :


```Dockerfile
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]

```  

Cela indique à Docker de :

- Construisez une image en commençant par l'image Python 3.10.
- Définir le répertoire de travail à /code.
- Définir les variables d'environnement utilisées par la commande flask.
- Installer gcc et les autres dépendances
- Copier le fichier requirements.txt
- Installer les dépendances de Python.
- Ajouter des métadonnées à l'image pour décrire que le conteneur écoute sur le port 5000.
- Copier le répertoire actuel . dans le projet vers le répertoire de travail . dans l'image.
- Définir la commande par défaut du conteneur à flask run.

Pour plus d'informations sur la rédaction de Dockerfiles, consultez [le guide de l'utilisateur de Docker](https://docs.docker.com/engine/reference/builder/).  

### Étapes 2 : Définir les services dans un fichier compose

Compose simplifie le contrôle de l'ensemble de votre pile d'applications, facilitant la gestion des services, des réseaux et des volumes dans un seul fichier de configuration YAML compréhensible.  

Créez un fichier appelé `compose.yaml` dans le répertoire de votre projet et entrez ce qui suit:

```compose.yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

Ce fichier Compose définit deux services : `web` et `redis`.


#### Service Web

Le service `web` utilise une image créée à partir du `Dockerfile` dans le répertoire actuel. Il lie ensuite le conteneur et la machine hôte au port exposé, `8000`. Cet exemple de service utilise le port par défaut pour le serveur Web Flask, `5000`.

#### Service Redis

Le service `redis` utilise une image `Redis` publique extraite du registre Docker Hub.

Pour plus d'informations sur le fichier `compose.yaml`, consultez [Comment fonctionne Compose](https://docs.docker.com/compose/intro/compose-application-model/).

### Étapes 3 : Céez et exécutez votre application avec Compose  

Avec une seule commande, vous créez et démarrez tous les services à partir de votre fichier de configuration.  

1- Depuis le répertoire de votre projet, démarrez votre application en exécutant `docker compose up`.

```bash
docker compose up
```

Compose extrait une image Redis, construit une image pour votre code et démarre les services que vous avez définis. Dans ce cas, le code est copié statiquement dans l'image au moment de la construction.

2- Entrez http://localhost:8000/ dans un navigateur pour voir l'application fonctionner.

Si ça ne résout pas, vous pouvez également essayer http://127.0.0.1:8000.   

Vous devriez voir un message dans votre navigateur disant :

![Image](https://docs.docker.com/compose/images/quick-hello-world-1.png)

3- Rafraichissez votre page, le nombre devrait s'incrémenter.

![Image](https://docs.docker.com/compose/images/quick-hello-world-2.png)

4- Passez à une autre fenêtre de terminal, et tapez `docker image ls` pour lister les images locales.

L'énumération des images à ce stade devrait retourner `redis` et `web` (plus précisément, *nom_répertoire*-web).

```bash
docker image ls
```

Vous pouvez inspecter les images avec `docker inspect <tag ou id>`.

5- Arrêtez l'application, soit en exécutant `docker compose down` depuis le répertoire de votre projet dans le second terminal, soit en appuyant sur CTRL+C dans le terminal d'origine où vous avez démarré l'application.

### Étapes 4 : Éditer le fichier compose pour utiliser Compose Watch  

Modifiez le fichier `compose.yaml` dans le répertoire de votre projet pour utiliser `watch` afin de prévisualiser vos services Compose en cours d'exécution, qui sont automatiquement mis à jour lorsque vous modifiez et enregistrez votre code :

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```  

Chaque fois qu'un fichier est modifié, Compose le synchronise avec l'emplacement correspondant sous `/code` dans le conteneur. Une fois copié, le bundler met à jour l'application en cours d'exécution sans redémarrage.

Pour plus d'informations sur le fonctionnement de Compose Watch, consultez la section [Utiliser Compose Watch](https://docs.docker.com/compose/how-tos/file-watch/). Vous pouvez également consulter la section [Gérer les données dans les conteneurs](https://docs.docker.com/engine/storage/volumes/) pour d'autres options.

**Remarque :**  
Pour que cet exemple fonctionne, l'option `--debug` est ajoutée au `Dockerfile`. L'option `--debug` dans Flask active le rechargement automatique du code, ce qui permet de travailler sur l'API backend sans redémarrer ni reconstruire le conteneur. Après avoir modifié le fichier `.py`, les appels d'API suivants utiliseront le nouveau code, mais l'interface utilisateur du navigateur ne s'actualisera pas automatiquement dans ce petit exemple. La plupart des serveurs de développement front-end incluent une prise en charge native du rechargement en direct compatible avec Compose.  


### Étape 5 : Recompiler et exécuter l’application avec Compose

Depuis le répertoire de votre projet, saisissez `docker compose watch` ou `docker compose up --watch` pour compiler et lancer l’application, puis activer le mode de surveillance des fichiers.

Consultez à nouveau le message « Hello World » dans un navigateur web et actualisez la page pour voir l’incrémentation du compteur.

### Étape 6 : Mettre à jour l’application

Pour voir Compose Watch en action :

1- Modifiez le message d’accueil dans `app.py` et enregistrez-le. Par exemple, remplacez le message « Hello World !» par « Hello from Docker! » :  


```python
### Le début n'est pas affiché.

def hello():
    count = get_hit_count()
    return 'Hello World from Docker! I have been seen {} times.\n'.format(count)
```

2- Actualisez l’application dans votre navigateur. Le message d’accueil devrait être mis à jour et le compteur devrait continuer à s’incrémenter.

![Hello world Dans le navigateur 3.](https://docs.docker.com/compose/images/quick-hello-world-3.png)

Une fois terminé, faire Ctrl-c ou exécutez `docker compose down`.

### Étape 7 : Séparer vos services  

L’utilisation de plusieurs fichiers Compose vous permet de personnaliser une application Compose pour différents environnements ou workflows. Ceci est utile pour les applications volumineuses pouvant utiliser des dizaines de conteneurs, dont la propriété est répartie entre plusieurs équipes.

1- Dans le dossier de votre projet, créez un fichier Compose appelé `infra.yaml`.  

2- Coupez le service Redis de votre fichier `compose.yaml` et collez-le dans votre nouveau fichier `infra.yaml`. Assurez-vous d’ajouter l’attribut de niveau supérieur `services` en haut de votre fichier. Votre fichier `infra.yaml` devrait maintenant ressembler à ceci :

```yaml
services:
  redis:
    image: "redis:alpine"
```  

3- Dans votre fichier `compose.yaml`, ajoutez l’attribut de niveau supérieur `include` ainsi que le chemin d’accès au fichier `infra.yaml`.  

```yaml
include:
   - infra.yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
```  

4- Exécutez ``docker compose watch`` pour compiler l'application avec les fichiers Compose mis à jour, puis exécutez-la. Vous devriez voir le message « Hello world » dans votre navigateur.

Il s'agit d'un exemple simplifié, mais il illustre le principe de base de l'inclusion et comment elle peut faciliter la modularisation d'applications complexes en sous-fichiers Compose. Pour plus d'informations sur l'inclusion et l'utilisation de plusieurs fichiers Compose, consultez la section « [Utiliser plusieurs fichiers Compose](https://docs.docker.com/compose/how-tos/multiple-compose-files/) ».

### Pour vérification de watch

Modifiez le message d'accueil dans `app.py` pour remplacez le message `Hello World from Docker!` par `Hello World from Votre_nom!` et enregistrez-le.  

Actualisez l'application dans votre navigateur. Le message d'accueil devrait être mis à jour, et le compteur devrait continuer à s'incrémenter.  

### Étapes 8 : Expérimentez d'autres commandes

Je vous rappelle que si vous souhaitez exécuter vos services en arrière-plan, vous pouvez passer le paramètre `-d` (pour le mode "détaché") à `docker compose up` et utiliser `docker compose ps` pour voir ce qui est en cours d'exécution :

```bash
$docker compose up -d
[+] Running 2/2
 ⠿ Container composetest-redis-1  Started
 ⠿ Container composetest-web-1    Started

$docker compose ps 
```

La commande `docker compose run` vous permet d'exécuter des commandes ponctuelles pour vos services. Par exemple, pour voir quelles variables d'environnement sont disponibles pour le service `web` :

```bash
$docker compose run web env
```

Voir `docker compose --help` pour voir les autres commandes disponibles.

Vous pouvez également installer la complétion de commande pour les shells bash et zsh, qui vous indique également les commandes disponibles.

Si vous avez démarré Compose avec `docker compose up -d`, vous pouvez seulement arrêter les services :

```bash
$docker compose stop
$docker container ls -a
```

Vous pouvez tout démonter, en supprimant entièrement les conteneurs, avec la commande `down`. Ajoutez le paramètre `--volumes` pour supprimer également un volume de données utilisé par un conteneur :

```bash
$docker compose down --volumes
$docker container ls -a
```

À ce stade, vous avez vu les bases du fonctionnement de Compose.

## Références

<https://docs.docker.com/compose/gettingstarted/>  
<https://docs.docker.com/engine/reference/builder/>  
<https://docs.docker.com/compose/intro/compose-application-model/>  
<https://docs.docker.com/compose/how-tos/multiple-compose-files/>  
<https://docs.docker.com/compose/how-tos/file-watch/>  
<https://docs.docker.com/engine/storage/volumes/>  
<https://docs.docker.com/compose/intro/compose-application-model/>  