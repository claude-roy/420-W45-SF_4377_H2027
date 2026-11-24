# Exercice 10 - Docker : démarrer avec Docker Compose

### Informations

- Type de travail : individuel
- Durée estimée : 3 heures
- Système d'exploitation : Ubuntu client, Windows ou macOS
- Environnement : Docker

### Objectifs

- Décrire une application multiconteneur dans un fichier `compose.yaml`.
- Lancer, consulter et arrêter une application avec les commandes `docker compose`.
- Utiliser le DNS interne de Docker pour faire communiquer des services.
- Construire une image à partir d'un Dockerfile avec Compose.
- Paramétrer une application à l'aide d'un fichier `.env`.
- Contrôler l'ordre de démarrage des services avec un healthcheck et `depends_on`.
- Utiliser Compose Watch pour mettre à jour une application en cours d'exécution.
- Conserver les données d'un service à l'aide d'un volume nommé.
- Séparer une application en plusieurs fichiers Compose (`include`).
- Inspecter et déboguer une application avec `docker compose config`, `logs` et `exec`.

## Section 1 - Commandes de base

Dans cette section, vous allez utiliser les fichiers contenus dans [exercice10.zip](extra/exercice10.zip).

### Étape 1 - Analyse des fichiers

- Décompresser le fichier `exercice10.zip`.
- À l’aide d’un éditeur de texte (je recommande Visual Studio Code), ouvrir les fichiers dans le répertoire.
- Observer le fichier `compose.yaml`.

La partie `services` décrit les conteneurs que nous allons utiliser.  
Le premier conteneur va se nommer `proxy`, nous allons utiliser l’image alpine de nginx et le port 80 sera publié sur l’hôte.

```yaml
  proxy:
    image: nginx:alpine # on utilise la version alpine
    ports:
      - '80:80' # publie le port 80 de l'hôte vers le port 80 du conteneur
```

Ici, nous utilisons le paramètre `volumes` pour faire un bind mount du fichier `nginx.conf` dans notre hôte au fichier `/etc/nginx/conf.d/default.conf` dans le conteneur. Le fichier sera en lecture seulement dans le conteneur.

```yaml
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

Le deuxième conteneur va se nommer `serveurweb` et nous allons utiliser l’image la plus récente d’Apache.

```yaml
  serveurweb:
    image: httpd  # on utilise httpd:latest
```

- Observer le fichier `nginx.conf`. Pouvez-vous trouver à quel endroit on utilise le DNS de Docker pour référencer notre deuxième conteneur ? (vous n’avez pas à comprendre le fichier `nginx.conf`)

<details>
  <summary markdown="span">Réponse.</summary>

Avec le paramètre `proxy_pass`. Il référence le nom du service `serveurweb`, résolu par le DNS de Docker.

```nginx
proxy_pass         http://serveurweb;
```

</details>
<br>

### Étape 2 - Lancement des conteneurs

- Lancer ces conteneurs. Vous devez vous placer dans le répertoire des fichiers pour les lancer (avec Visual Studio Code, vous pouvez ouvrir un shell dans l’éditeur de texte).

```bash
docker compose up
```

Avec cette commande, vous allez lancer vos conteneurs, créer un réseau privé virtuel, faire un bind mount du fichier spécifié, publier le port sur l’hôte et envoyer les journaux à l’écran.

- Ouvrir un navigateur et aller à `http://localhost`. Oui, vous êtes dans le serveur Apache et non Nginx. Nginx sert de proxy ici. Si vous consultez vos journaux à la ligne de commande, vous pouvez voir que le `proxy` est appelé en premier et après le `serveurweb`.
- Arrêter vos conteneurs : faire `Ctrl-c`.

### Étape 3 - Conteneurs en arrière-plan

- Relancer vos conteneurs, mais en arrière-plan, et vérifier le fonctionnement avec un navigateur Web.

```bash
docker compose up -d
```

- Vérifier les commandes disponibles avec `docker compose`.

```bash
docker compose --help
```

Vous pouvez constater que de nombreuses commandes que vous avez déjà utilisées se retrouvent dans `docker compose`.

- Consulter les journaux.

```bash
docker compose logs
```

- Lister les conteneurs qui s’exécutent.

```bash
docker compose ps
```

- Lister les processus qui s’exécutent dans les conteneurs.

```bash
docker compose top
```

- Finalement, arrêter vos conteneurs.

```bash
docker compose down
```

## Section 2 - Application complète

Source : [Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)

Dans cette section, vous construisez une application Web simple en Python fonctionnant sur Docker Compose. L'application utilise le framework Flask et maintient un compteur de visites dans Redis. Bien que l'exemple utilise Python, les concepts démontrés ici devraient être compréhensibles même si vous n'êtes pas familier avec ce langage.

**Conditions préalables :** s'assurer d'avoir installé Docker Engine et Docker Compose. Vous n'avez pas besoin d'installer Python ou Redis, car ils sont tous deux fournis par les images Docker.

### Étape 1 - Configuration

Définir les dépendances de l'application.

- Créer un répertoire pour le projet :

```bash
mkdir composetest
cd composetest
```

- Créer un fichier appelé `app.py` dans le répertoire de votre projet et y coller ceci :

```python
import os
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(
    host=os.getenv("REDIS_HOST", "redis"),
    port=int(os.getenv("REDIS_PORT", "6379")),
)

@app.route("/")
def hello():
    count = cache.incr("hits")
    return f"Hello from Docker! I have been seen {count} time(s).\n"

```

Dans cet exemple, le nom d'hôte et le port de Redis viennent des variables `REDIS_HOST` et `REDIS_PORT`. Par défaut, ils valent `redis`, le nom du service sur le réseau de l'application, et `6379`, le port par défaut de Redis.

- Créer un autre fichier appelé `requirements.txt` dans le répertoire de votre projet et y coller le code suivant :

```text
flask
redis
```

Vous allez créer un Dockerfile qui construit une image Docker. L'image contient toutes les dépendances dont l'application Python a besoin, y compris Python lui-même.

- Dans le répertoire de votre projet, créer un fichier nommé `Dockerfile` et coller ce qui suit :

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt .
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```

Cela indique à Docker de :

- Construire une image en commençant par l'image Python 3.12.
- Définir le répertoire de travail à `/code`.
- Définir les variables d'environnement utilisées par la commande `flask`.
- Installer gcc et les autres dépendances.
- Copier le fichier `requirements.txt`.
- Installer les dépendances de Python.
- Ajouter des métadonnées à l'image pour décrire que le conteneur écoute sur le port 5000.
- Copier le répertoire actuel `.` dans le projet vers le répertoire de travail `.` dans l'image.
- Définir la commande par défaut du conteneur à `flask run --debug`.

Pour plus d'informations sur la rédaction de Dockerfiles, consulter [la référence Dockerfile](https://docs.docker.com/reference/dockerfile/).

- Créer un fichier `.env` contenant les variables d'environnement :

```text
APP_PORT=8000
REDIS_HOST=redis
REDIS_PORT=6379
```

Compose lit automatiquement le fichier `.env` et rend ces valeurs disponibles pour une interpolation dans votre fichier `compose.yaml`.

Dans cet exemple, les avantages sont limités, mais en pratique, le fait de séparer la configuration du fichier Compose facilite les opérations suivantes :

- Modifier des valeurs selon les environnements sans avoir à modifier le fichier YAML.
- Éviter d'inclure des données sensibles (secrets) dans le système de contrôle de version.
- Réutiliser des valeurs entre plusieurs services.

- Créer un fichier `.dockerignore` afin d'exclure les fichiers inutiles de votre contexte de construction :

```text
.env
*.pyc
__pycache__
redis-data
```

Lors de la construction d'une image, Docker envoie l'intégralité du contenu de votre répertoire de projet au démon Docker. En l'absence de fichier `.dockerignore`, cela inclut votre fichier `.env` (qui peut contenir des informations confidentielles) ainsi que tout bytecode Python mis en cache. Les exclure permet de maintenir une construction rapide et évite d'intégrer accidentellement des données sensibles dans une couche de l'image.

### Étape 2 - Définir les services dans un fichier Compose

Compose simplifie le contrôle de l'ensemble de votre pile d'applications, facilitant la gestion des services, des réseaux et des volumes dans un seul fichier de configuration YAML compréhensible.

- Créer un fichier appelé `compose.yaml` dans le répertoire de votre projet et entrer ce qui suit :

```yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}

  redis:
    image: redis:alpine
```

Ce fichier Compose définit deux services : `web` et `redis`.

#### Service Web

Le service `web` utilise une image créée à partir du `Dockerfile` dans le répertoire actuel. Le port de l'hôte défini par `APP_PORT` dans `.env` (`8000`) est publié vers le port `5000` du conteneur, le port par défaut du serveur Web Flask.

#### Service Redis

Le service `redis` utilise une image `redis` publique extraite du registre Docker Hub.

Pour plus d'informations sur le fichier `compose.yaml`, consulter [Comment fonctionne Compose](https://docs.docker.com/compose/intro/compose-application-model/).

### Étape 3 - Créer et exécuter votre application avec Compose

Avec une seule commande, vous créez et démarrez tous les services à partir de votre fichier de configuration.

- Depuis le répertoire de votre projet, démarrer votre application en exécutant `docker compose up`.

```bash
docker compose up
```

Compose extrait une image Redis, construit une image pour votre code et démarre les services que vous avez définis. Dans ce cas, le code est copié statiquement dans l'image au moment de la construction.

- Entrer `http://localhost:8000/` dans un navigateur pour voir l'application fonctionner.

Si cela ne fonctionne pas, vous pouvez également essayer `http://127.0.0.1:8000`.

Vous devriez voir un message dans votre navigateur disant :

```text
Hello from Docker! I have been seen 1 time(s).
```

- Actualiser la page. Le nombre devrait s'incrémenter.

```text
Hello from Docker! I have been seen 2 time(s).
```

- Passer à une autre fenêtre de terminal et exécuter `docker image ls` pour lister les images locales.

```bash
docker image ls
```

L'énumération des images à ce stade devrait retourner `redis` et `web` (plus précisément, *nom_répertoire*-web).

Vous pouvez inspecter les images avec `docker image inspect <tag ou id>`.

Cette configuration minimale fonctionne, mais elle présente deux problèmes que vous corrigerez dans les prochaines étapes :

- Conflit au démarrage : le service `web` démarre en même temps que Redis. Si Redis n'est pas encore prêt, l'application Flask ne parvient pas à se connecter et plante.
- Absence de persistance : si vous exécutez `docker compose down` suivi de `docker compose up`, le compteur revient à zéro. La commande `docker compose down` supprime les conteneurs, et avec eux, toutes les données écrites dans la couche inscriptible du conteneur. La commande `docker compose stop`, quant à elle, conserve les conteneurs, ce qui permet aux données de subsister ; toutefois, vous ne pouvez pas compter sur ce comportement en production, où les conteneurs sont régulièrement remplacés.

- Arrêter l'application en exécutant `docker compose down` depuis le répertoire de votre projet.

### Étape 4 - Résoudre le problème de concurrence au démarrage grâce aux vérifications de santé (health checks)

Pour résoudre ce problème de concurrence, Compose doit attendre que Redis soit déclaré « en bonne santé » (healthy) avant de démarrer le service `web`.

- Mettre à jour le fichier `compose.yaml` :

```yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s
```

Le bloc `healthcheck` indique à Compose comment vérifier si Redis est prêt :

- `test` correspond à la commande que Compose exécute à l'intérieur du conteneur pour vérifier son état. La commande `redis-cli ping` se connecte à Redis et attend une réponse « PONG » ; si elle est reçue, le conteneur est considéré comme étant en bonne santé.
- `start_period` accorde à Redis 10 secondes pour s'initialiser avant que les vérifications de santé ne commencent. Les échecs survenant durant cette période ne sont pas comptabilisés dans la limite des tentatives.
- `interval` définit la fréquence des vérifications (toutes les 5 secondes) une fois la période initiale (`start_period`) écoulée.
- `timeout` alloue 3 secondes à chaque vérification pour obtenir une réponse ; au-delà, la vérification est considérée comme un échec.
- `retries` détermine le nombre d'échecs consécutifs autorisés avant que Compose ne considère le conteneur comme étant dans un état non sain (unhealthy). Avec `interval: 5s` et `retries: 5`, Compose attendra jusqu'à 25 secondes avant d'abandonner.

- Lancer la pile (stack) pour confirmer que l'ordre est bien établi :

```bash
docker compose up -d
```

Vous devriez voir un résultat similaire à celui-ci :

```text
[+] up 2/2
✔ Container composetest-redis-1  Healthy                       0.0s
✔ Container composetest-web-1    Started                       0.0s
```

Dans votre navigateur, entrer `http://localhost:8000` pour vérifier que l'application fonctionne toujours, puis arrêter la pile avant de poursuivre :

```bash
docker compose down
```

### Étape 5 - Éditer le fichier Compose pour utiliser Compose Watch

Sans Compose Watch, chaque modification du code vous oblige à arrêter la pile (stack), à reconstruire l’image et à redémarrer les conteneurs. Compose Watch supprime ce cycle en synchronisant automatiquement les modifications dans votre conteneur en cours d’exécution, au fur et à mesure que vous enregistrez vos fichiers.

- Modifier le fichier `compose.yaml` dans le répertoire de votre projet pour utiliser `develop.watch` :

```yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code
        - action: rebuild
          path: requirements.txt

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s
```

Le bloc `watch` définit deux règles :

- L'action `sync+restart` surveille le répertoire de votre projet (`.`) sur l'hôte. Lorsqu'un fichier est modifié, Compose copie tous les fichiers modifiés dans `/code` à l'intérieur du conteneur en cours d'exécution, puis redémarre le conteneur. Étant donné que le conteneur redémarre avec les fichiers mis à jour déjà en place, Flask commence à lire directement le nouveau code – aucune reconstruction ou redémarrage manuel n'est nécessaire.
- L'action `rebuild` sur `requirements.txt` déclenche une reconstruction complète de l'image chaque fois que vous ajoutez une nouvelle dépendance, car l'installation des paquets nécessite la reconstruction de l'image, pas seulement la synchronisation des fichiers.

Pour plus d'informations sur le fonctionnement de Compose Watch, consulter la section [Utiliser Compose Watch](https://docs.docker.com/compose/how-tos/file-watch/). Vous pouvez également consulter la section [Gérer les données dans les conteneurs](https://docs.docker.com/engine/storage/volumes/) pour d'autres options.

**Remarque :** l'option `--debug` du `Dockerfile` active le mode de développement de Flask (rechargement du code et messages d'erreur détaillés). Ici, c'est l'action `sync+restart` qui redémarre le conteneur après chaque synchronisation. Le navigateur ne s'actualise pas automatiquement : il faut actualiser la page.

### Étape 6 - Reconstruire et exécuter l’application avec Compose

- Depuis le répertoire de votre projet, exécuter `docker compose watch` ou `docker compose up --watch` pour construire et lancer l’application, puis activer le mode de surveillance des fichiers.
- Consulter à nouveau le message « Hello from Docker! » dans un navigateur Web et actualiser la page pour voir l’incrémentation du compteur.

### Étape 7 - Mettre à jour l’application

Pour voir Compose Watch en action :

- Dans votre éditeur, modifier le message d’accueil dans `app.py` et l'enregistrer. Par exemple, remplacer le message « Hello from Docker! » par « Hello from Compose Watch! » :

```python
# Le reste du fichier n'est pas affiché.

def hello():
    count = cache.incr("hits")
    return f"Hello from Compose Watch! I have been seen {count} time(s).\n"
```

Dans le terminal de `docker compose watch`, vous devriez voir :

```text
Syncing service "web" after 1 changes were detected
Container composetest-web-1 Restarting
service(s) ["web"] restarted
```

- Actualiser l’application dans votre navigateur. Le message d’accueil devrait être mis à jour et le compteur devrait continuer à s’incrémenter.

```text
Hello from Compose Watch! I have been seen 3 time(s).
```

- Une fois terminé, faire `Ctrl-c` ou exécuter `docker compose down` à partir d'un autre terminal.

### Étape 8 - Persister les données grâce à des volumes nommés

À chaque fois que vous arrêtez puis redémarrez la pile (stack), le compteur de visites revient à zéro. Les données de Redis résident à l'intérieur du conteneur ; elles disparaissent donc lorsque celui-ci est supprimé. Un volume nommé permet de résoudre ce problème en stockant les données sur l'hôte, en dehors du cycle de vie du conteneur.

- Mettre à jour le fichier `compose.yaml` :

```yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code
        - action: rebuild
          path: requirements.txt

  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s

volumes:
  redis-data:
```

L'entrée `redis-data:/data` sous `redis.volumes` monte le volume nommé sur `/data`, le chemin où Redis écrit ses fichiers de données. La clé `volumes` (au niveau racine) l'enregistre auprès de Docker, ce qui permet aux données de persister entre les cycles `docker compose down` et `docker compose up`.

- Lancer la pile (stack) avec `docker compose up --watch` et actualiser la page `http://localhost:8000` à plusieurs reprises pour faire augmenter le compteur.

- Arrêter la pile avec `Ctrl-c`, puis la relancer avec `docker compose up --watch`.

- Ouvrir `http://localhost:8000` : le compteur reprend là où il s'était arrêté.

- Maintenant, arrêter les conteneurs et « réinitialiser » le compteur à l'aide de la commande `docker compose down -v`.

L'option `-v` supprime les volumes nommés en même temps que les conteneurs. À utiliser en toute connaissance de cause : **elle entraîne la suppression définitive des données stockées**.

### Étape 9 - Séparer vos services

L’utilisation de plusieurs fichiers Compose vous permet de personnaliser une application Compose pour différents environnements ou flux de travail (workflows). Ceci est utile pour les applications volumineuses pouvant utiliser des dizaines de conteneurs, dont la propriété est répartie entre plusieurs équipes.

- Dans le dossier de votre projet, créer un fichier Compose appelé `infra.yaml`.
- Couper le service et le volume Redis de votre fichier `compose.yaml` et le coller dans votre nouveau fichier `infra.yaml`. S'assurer d’ajouter l’attribut de niveau supérieur `services` en haut de votre fichier. Votre fichier `infra.yaml` devrait maintenant ressembler à ceci :

```yaml
services:
  redis:
    image: redis:alpine
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s

volumes:
  redis-data:
```

- Dans votre fichier `compose.yaml`, ajouter l’attribut de niveau supérieur `include` ainsi que le chemin d’accès au fichier `infra.yaml`.

```yaml
include:
  - path: ./infra.yaml
services:
  web:
    build: .
    ports:
      - "${APP_PORT}:5000"
    environment:
      - REDIS_HOST=${REDIS_HOST}
      - REDIS_PORT=${REDIS_PORT}
    depends_on:
      redis:
        condition: service_healthy
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /code
        - action: rebuild
          path: requirements.txt
```

- Exécuter `docker compose watch` pour construire l'application avec les fichiers Compose mis à jour, puis l'exécuter. Vous devriez voir le message « Hello from Compose Watch! » dans votre navigateur.

Il s'agit d'un exemple simplifié, mais il illustre le principe de base de l'inclusion et comment elle peut faciliter la modularisation d'applications complexes en sous-fichiers Compose. Pour plus d'informations sur l'inclusion et l'utilisation de plusieurs fichiers Compose, consulter la section « [Utiliser plusieurs fichiers Compose](https://docs.docker.com/compose/how-tos/multiple-compose-files/) ».

Nous allons vérifier le fonctionnement de `watch`.

- Modifier le message d'accueil dans `app.py` pour remplacer le message `Hello from Compose Watch!` par `Hello from Votre_nom!` et l'enregistrer.
- Actualiser l'application dans votre navigateur. Le message d'accueil devrait être mis à jour, et le compteur devrait continuer à s'incrémenter.

- Une fois terminé, exécuter `docker compose down` pour arrêter votre pile.

### Étape 10 - Inspecter et déboguer votre pile en cours d'exécution

Une fois votre pile entièrement configurée, vous pouvez observer ce qui se passe à l'intérieur de vos conteneurs sans rien interrompre. Cette étape présente les commandes essentielles pour inspecter la configuration finale, consulter les journaux (logs) en temps réel et exécuter des commandes au sein d'un conteneur actif.

- Avant de démarrer la pile, vérifier que Compose a bien résolu vos variables définies dans le fichier `.env` et fusionné correctement tous les fichiers :

```bash
docker compose config
```

La commande `docker compose config` ne nécessite pas que la pile soit en cours d'exécution ; elle se base uniquement sur vos fichiers. Voici quelques points à noter dans le résultat affiché :

- Les variables `${APP_PORT}`, `${REDIS_HOST}` et `${REDIS_PORT}` ont été remplacées par les valeurs présentes dans votre fichier `.env`.
- La notation abrégée des ports (« 8000:5000 ») est développée en ses champs détaillés (cible, port publié, protocole).
- Les noms par défaut des réseaux et des volumes sont explicités, avec le préfixe correspondant au nom du projet, `composetest`.
- Le résultat correspond à la configuration entièrement résolue ; tous les fichiers intégrés via la directive `include` sont fusionnés en une vue unique.

Utiliser `docker compose config` chaque fois que vous souhaitez vérifier ce que Compose va réellement appliquer, notamment lors du débogage de la substitution de variables ou lorsque vous travaillez avec plusieurs fichiers Compose.

- Démarrer maintenant la pile en mode détaché (« detached mode »), afin que votre terminal reste disponible pour les commandes suivantes :

```bash
docker compose up -d
```

- Afficher les journaux (logs) de tous les services en temps réel :

```bash
docker compose logs -f
```

L'option `-f` permet de suivre le flux des journaux en temps réel ; les sorties des différents conteneurs sont entrelacées et précédées du nom du service, affiché avec un code couleur. Actualiser la page `http://localhost:8000` à plusieurs reprises et observer l'apparition des journaux de requêtes Flask. Pour suivre les journaux d'un service spécifique, indiquer son nom :

```bash
docker compose logs -f web
```

Appuyer sur `Ctrl-c` pour arrêter le suivi des journaux. Les conteneurs continuent de fonctionner.

#### Exécuter des commandes au sein d'un conteneur en cours d'exécution

La commande `docker compose exec` permet d'exécuter une instruction dans un conteneur déjà actif, sans avoir à en démarrer un nouveau. Il s'agit de l'outil privilégié pour le débogage en temps réel.

- Vérifier que les variables d'environnement sont correctement définies :

```bash
docker compose exec web env | grep REDIS
```

Résultat :

```text
REDIS_HOST=redis
REDIS_PORT=6379
```

- Vérifier que le conteneur `web` peut accéder à Redis en utilisant le nom du service comme nom d'hôte :

```bash
docker compose exec web python -c "import redis; r = redis.Redis(host='redis'); print(r.ping())"
```

Résultat :

```text
True
```

Cette opération utilise la même bibliothèque `redis` que celle employée par votre application ; l'obtention de la réponse `True` confirme donc que la découverte de services, la configuration réseau et la connexion à Redis fonctionnent correctement de bout en bout.

- Inspecter la valeur actuelle du compteur de visites dans Redis :

```bash
docker compose exec redis redis-cli GET hits
```

Résultat (la valeur dépend du nombre de visites) :

```text
"5"
```

Exécuter `docker compose --help` pour voir les autres commandes disponibles.

Si vous avez démarré Compose avec `docker compose up -d`, vous pouvez arrêter les services sans les supprimer :

```bash
docker compose stop
# On vérifie que les conteneurs existent toujours.
docker container ls -a
```

Je vous rappelle que vous pouvez tout démonter, en supprimant entièrement les conteneurs, avec la commande `down`. Si vous ajoutez le paramètre `--volumes` ou `-v`, vous supprimez également les volumes de données utilisés par les conteneurs :

```bash
docker compose down --volumes
# On vérifie que les conteneurs et les volumes sont supprimés.
docker container ls -a
docker volume ls
```

À ce stade, vous avez vu les bases du fonctionnement de Compose.

## Références

- [https://docs.docker.com/compose/gettingstarted/](https://docs.docker.com/compose/gettingstarted/)
- [https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)
- [https://docs.docker.com/compose/intro/compose-application-model/](https://docs.docker.com/compose/intro/compose-application-model/)
- [https://docs.docker.com/compose/how-tos/multiple-compose-files/](https://docs.docker.com/compose/how-tos/multiple-compose-files/)
- [https://docs.docker.com/compose/how-tos/file-watch/](https://docs.docker.com/compose/how-tos/file-watch/)
- [https://docs.docker.com/engine/storage/volumes/](https://docs.docker.com/engine/storage/volumes/)
- [https://docs.docker.com/compose/how-tos/environment-variables/](https://docs.docker.com/compose/how-tos/environment-variables/)
- [https://docs.docker.com/compose/how-tos/startup-order/](https://docs.docker.com/compose/how-tos/startup-order/)

---

*Version 2026-11-24*
