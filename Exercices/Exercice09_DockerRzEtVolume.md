# Exercice 9 - Docker : réseau et volume

### Informations

- Type de travail : individuel
- Durée estimée : 4 heures
- Système d'exploitation : Ubuntu client, Windows ou macOS
- Environnement : Docker

### Objectifs

- Distinguer les réseaux Docker `bridge`, `none` et `host`.
- Publier le port d'un conteneur vers l'hôte.
- Monter un répertoire de l'hôte dans un conteneur (bind mount), en lecture-écriture et en lecture seule.
- Conserver les données d'une base de données à l'aide d'un volume.
- Faire communiquer plusieurs conteneurs (WordPress et MySQL).

**Note :** je vous recommande de vous créer un répertoire de travail nommé `Exercice09`.

## Section 1 - Busybox

### Étape 1 - Réseau par défaut (bridge)

- Ouvrir **deux** terminaux, PowerShell ou bash, et sur chacun, lancer un conteneur de l'image `busybox` en mode interactif.

```bash
# Exemple pour le premier conteneur.
docker container run -it --rm --name busy1 busybox
```

- Trouver l'adresse IP de chaque conteneur.

<details>
  <summary markdown="span">Quelle commande ?</summary>

```bash
ip -4 address
```

</details>
<br>  

- Effectuer une requête ICMP echo request (`ping`) d'un conteneur vers l'autre. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Ils peuvent communiquer à l'intérieur de Docker.

</details>
<br>  

- Essayer de faire une requête ICMP echo request vers `www.google.com`. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Ils peuvent communiquer à l'extérieur.

</details>
<br>  

- Quitter les conteneurs : `exit`.
- Nettoyer vos conteneurs en supprimant, si nécessaire, ceux créés dans cette étape.

### Étape 2 - Réseau none

- Lancer un conteneur, en mode interactif, de l'image `busybox` avec l'option `--network none`.

<details>
  <summary markdown="span">La commande.</summary>

```bash
docker container run --rm --network none -it busybox
```

</details>
<br>  

- Essayer de trouver l'adresse IP du conteneur. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Il n'y a pas d'adresse. Il n'y a même pas d'interface réseau.

</details>
<br>  

- Essayer de faire une requête ICMP echo request vers `www.google.com`. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Ce n'est pas possible, car nous ne sommes pas reliés à un réseau.

</details>
<br>  

- Nettoyer votre conteneur en supprimant, si nécessaire, celui créé dans cette étape.

### Étape 3 - Réseau host

- Lancer un conteneur, en mode interactif, de l'image `busybox` avec l'option `--network host`.
- Essayer de trouver l'adresse IP du conteneur. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Vous avez reçu une adresse dans le même réseau que votre hôte.

</details>
<br>  

- Essayer de faire une requête ICMP echo request vers `www.google.com`. Qu'observez-vous ?

<details>
  <summary markdown="span">Réponse.</summary>

Vous pouvez communiquer à l'extérieur.

</details>
<br>  

- Nettoyer votre conteneur en supprimant, si nécessaire, celui créé dans cette étape.

**Note :** pour les utilisateurs de Docker Desktop sous macOS ou Windows, le réseau host nécessite Docker Desktop 4.34 ou plus récent et doit être activé dans *Settings > Resources > Network > Enable host networking*. Même activé, comme vous êtes dans une VM, vous allez être connecté au réseau de l'hyperviseur. Donc, vous ne recevrez pas une adresse de votre réseau.

## Section 2 - Nginx - réseau

Dans cette section, vous allez devoir utiliser l'image du serveur Web Nginx afin de créer un serveur Web ([Documentation Nginx](https://hub.docker.com/_/nginx)).

### Étape 1 - Nginx - premiers tests

- Sur un premier terminal, lancer en arrière-plan (`-d`) l'image `nginx` sans spécifier de redirection de port avec le nom `mon_nginx` (`--name`).
- Faire la commande `docker container inspect mon_nginx` afin de trouver l'adresse du conteneur.
- À partir de votre navigateur favori, essayer d'ouvrir une page Web à l'adresse IP de votre conteneur Nginx. Essayer avec l'adresse locale : `localhost`.

<details>
  <summary markdown="span">Qu'observez-vous et pourquoi ?</summary>

Avec `localhost`, il n'est pas possible d'accéder au serveur Web Nginx. Il ne serait également pas possible à d'autres systèmes sur le réseau d'y accéder, car nous n'avons pas créé de lien vers le monde extérieur de Docker. Le conteneur est accessible par les autres conteneurs de Docker qui sont dans le même réseau Docker. Sous Linux, il est également accessible par votre hôte avec son adresse IP, car Docker a créé une interface virtuelle dans le même réseau que votre conteneur. Avec Docker Desktop (Windows ou macOS), ce n'est pas le cas : les conteneurs s'exécutent dans une VM et l'hôte ne peut pas joindre directement leur adresse IP.

</details>
<br>  

- Nettoyer vos conteneurs en supprimant ceux créés dans cette étape.

### Étape 2 - Nginx - liaison

- Recréer un conteneur, mais cette fois-ci, faire une liaison entre le port 1234 de l'hôte vers le port 80 du conteneur.
- Refaire les tests de l'étape précédente (utiliser `http://localhost:1234` pour le navigateur local).
- Nettoyer vos conteneurs en supprimant ceux créés dans cette étape.

## Section 3 - Nginx - volume

### Étape 1 - Montage d'un répertoire de l'hôte (bind mount)

- Créer un répertoire de travail temporaire et créer un fichier `index.html`. Dans ce fichier, écrire un code HTML simple afin d'afficher "Un fichier de mon hôte servi à partir d'un conteneur".
- Par défaut, l'image `nginx` utilise le répertoire `/usr/share/nginx/html` comme base de recherche de ses fichiers statiques. Vous allez devoir créer un point de montage (bind mount) afin que votre répertoire soit monté vers ce chemin au moment du démarrage du conteneur (consulter la partie "Hosting some simple static content" de la [documentation Nginx](https://hub.docker.com/_/nginx)). Vous devez également faire une liaison entre le port 1234 de l'hôte vers le port 80 du conteneur.
	- Pour spécifier le chemin (path) de l’hôte, nous pouvons écrire le chemin complet ou nous pouvons utiliser la variable shell `$(pwd)`.
	- Remarque : pour PowerShell utiliser `${pwd}` et pour cmd.exe utiliser `%cd%`.

<details>
  <summary markdown="span">Commande.</summary>

```bash
docker container run --name some-nginx -v $(pwd):/usr/share/nginx/html -d -p 1234:80 nginx
```

</details>
<br>  

- Vérifier que vous voyez bien votre page dans votre navigateur.

### Étape 2 - Montage en lecture seule

- Supprimer votre conteneur et en lancer un nouveau en faisant maintenant un montage en lecture seule (`:ro` comme dans la documentation de Nginx).
- Vérifier que le tout fonctionne toujours.
- Nettoyer votre conteneur en supprimant celui créé dans cette étape.

## Section 4 - MySQL

### Étape 1 - MySQL - Base

- Installer MySQL Workbench ([https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/), choisir la version pour Debian, Windows ou macOS).

```bash
cd ~/Downloads
sudo apt install ./mysql-workbench*.deb
# Il se peut que vous ayez à faire
sudo apt --fix-broken install
```

- Lancer un conteneur à partir de l'image `mysql` (référence : [https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql)) :
	- Nom : `mysql`
	- Spécifier que le conteneur doit être détaché (`-d`).
	- Fournir la valeur `Passw0rd` pour la variable d'environnement `MYSQL_ROOT_PASSWORD` (voir utilisation du `-e` dans la documentation).
	- Lier le port 3306 local vers le port 3306 du conteneur.

<details>
  <summary markdown="span">Commande.</summary>

```bash
docker container run -d --rm --name mysql -e MYSQL_ROOT_PASSWORD=Passw0rd -p 3306:3306 mysql
```

</details>
<br>  

- Avec MySQL Workbench, vérifier que le tout marche :
	- Se connecter.
	- Créer la base de données `test` (`create database test;`).
	- Vérifier que la base de données est bien existante (`show databases;`).
- Arrêter et supprimer le conteneur `mysql`.
- Recréer un nouveau conteneur `mysql`, se connecter de nouveau à MySQL et lister les bases de données. Que constatez-vous et pourquoi ?

<details>
  <summary markdown="span">Réponse.</summary>

La base de données ne s'y trouve pas. Un nouveau conteneur utilise un nouveau volume. Il aurait fallu lier les conteneurs au même volume, créé précédemment.

</details>
<br>  

- Supprimer votre conteneur.

### Étape 2 - MySQL - Volume

- Créer un répertoire qui va contenir vos données pour MySQL.
- Réutiliser votre ligne de commande pour créer un conteneur MySQL avec les mêmes caractéristiques que précédemment et ajouter le montage d'un volume qui lie le répertoire de données que vous venez de créer au répertoire `/var/lib/mysql`.

<details>
  <summary markdown="span">Commande.</summary>

```bash
docker container run -d --rm --name mysql -e MYSQL_ROOT_PASSWORD=Passw0rd -p 3306:3306 -v $(pwd)/mysql:/var/lib/mysql mysql
```

</details>
<br>  

- Avec MySQL Workbench, vérifier que le tout marche :
	- Se connecter.
	- Créer la base de données `test` (`create database test;`).
	- Vérifier que la base de données est bien existante (`show databases;`).
- Arrêter et supprimer le conteneur `mysql`.
- Recréer un nouveau conteneur `mysql` identique, se connecter de nouveau à MySQL et lister les bases de données. Que constatez-vous et pourquoi ?

<details>
  <summary markdown="span">Réponse.</summary>

La base de données s'y trouve. Les deux conteneurs utilisent le même volume.

</details>
<br>  

- Supprimer votre conteneur.

## Section 5 - WordPress - On fait parler plusieurs conteneurs ?

### Étape 1 - WordPress - Version manuelle

- Créer un nouveau répertoire pour stocker une autre installation de bases de données.
- Créer un nouveau conteneur avec les options précédentes en modifiant le chemin du montage ainsi qu'en ajoutant les variables d'environnement suivantes :
	- `MYSQL_DATABASE=wordpress`
	- `MYSQL_USER=wordpress`
	- `MYSQL_ROOT_PASSWORD=Passw0rd`
	- `MYSQL_PASSWORD=Passw0rd`
- Avec MySQL Workbench, essayer de se connecter à la base de données `wordpress` avec le nom d'utilisateur `wordpress` et le mot de passe `Passw0rd`.
- Chercher l'adresse IP du conteneur `mysql`.
- Créer un conteneur à partir de l'image `wordpress` ([https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)) en liant le port local 8080 au port 80 et avec les variables d'environnement suivantes (remplacer l'adresse IP par celle que vous avez trouvée) :
	- `WORDPRESS_DB_HOST=172.17.0.2`
	- `WORDPRESS_DB_USER=wordpress`
	- `WORDPRESS_DB_PASSWORD=Passw0rd`
	- `WORDPRESS_DB_NAME=wordpress`
	- `WORDPRESS_TABLE_PREFIX=wp_`
- Tester votre site Web (`localhost` au port 8080) et vérifier que ça fonctionne.
- Que pensez-vous de toutes ces manipulations ?

<details>
  <summary markdown="span">Solution.</summary>

```bash
docker container run -d --rm --name mysql -e MYSQL_ROOT_PASSWORD=Passw0rd -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=Passw0rd -p 3306:3306 -v $(pwd)/mysql_wp:/var/lib/mysql mysql

docker container run --rm --name wordpress -d -e WORDPRESS_DB_HOST=172.17.0.2 -e WORDPRESS_DB_USER=wordpress -e WORDPRESS_DB_PASSWORD=Passw0rd -e WORDPRESS_DB_NAME=wordpress -e WORDPRESS_TABLE_PREFIX=wp_ -p 8080:80 wordpress
```

</details>
<br>  

### Étape 2 - WordPress - Un début d'orchestration ?

- Consulter les informations présentées à la page suivante : [https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/wordpress/](https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/wordpress/)
	- Nous allons voir Docker Compose dans le prochain exercice. Mais si ça vous le dit, vous pouvez essayer de réaliser ce petit projet. Cependant, certaines des instructions données sont désuètes.
	- Comme indiqué dans le bas de la page, pour l'arrêter, utiliser la commande `docker compose down`.
- Quels sont les avantages d'un tel système ?

## Références
- [https://docs.docker.com/engine/network/drivers/host/](https://docs.docker.com/engine/network/drivers/host/)
- [https://hub.docker.com/_/nginx](https://hub.docker.com/_/nginx)
- [https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql)
- [https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)
- [https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/)
- [https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/wordpress/](https://github.com/docker/awesome-compose/tree/master/official-documentation-samples/wordpress/)

---

*Version 2026-11-23*
