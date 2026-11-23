# Exercice 7 - Docker : Prise en main des conteneurs Docker

### Informations

- Type de travail : individuel, formative à remettre  
- Durée estimée : 2 heures
- Système d'exploitation : Ubuntu client, Windows ou macOS
- Environnement : Docker

### Objectifs

- Analyser la documentation technique des services à installer. 
- Distinguer les services à installer sur les serveurs.
- Connecter les serveurs au réseau physique ou virtuel.  
- Lancer, nommer et détacher un conteneur.
- Publier des ports.
- Consulter les journaux.
- Arrêter et supprimer des conteneurs.

## Préambule - Installation

- Sous Windows  
	- [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/).  
	- [Intégration à WSL](https://docs.docker.com/desktop/windows/wsl/).
- Sous Linux : voir l'exercice 1.  
- [Sous macOS](https://docs.docker.com/desktop/install/mac-install/). 
  
<details>
    <summary markdown="span">Docker et WSL ?</summary>

Docker Desktop pour Windows utilise WSL 2 (Windows Subsystem for Linux 2) comme moteur par défaut, ce qui évite de créer une machine virtuelle dans Hyper-V. WSL 2 doit être activé avant l'installation de Docker Desktop (commande `wsl --install` dans un terminal administrateur).
La configuration minimale de Windows est indiquée sur la page d'installation de Docker Desktop.

</details>


## Section 1 - Premier conteneur

### Étape 1 - Vérification de l'installation de Docker

Vous allez vérifier le bon fonctionnement de votre installation de Docker.

À partir d’une ligne de commande, vérifier la version de Docker.

```bash
docker version
```

Quelle est la version de votre Docker ?
<details>
  <summary markdown="span">Réponse.</summary>
	La réponse varie.
</details>

Vous pouvez vérifier les paramètres de la configuration de l’engin Docker.

```bash
docker info
```

Pour connaître les commandes disponibles de Docker, entrer la commande `docker` suivie de `Enter`.

Si vous remontez la liste, vous allez trouver la section _Management Commands_.  

Le format d’une commande Docker est :

```bash
docker <management_command> <command> (options)
```

L’ancienne méthode, qui fonctionne encore, était :

```bash
docker <command> (options)
```

Par exemple, si vous voulez lancer un conteneur :

```bash
docker container run (option)
```

### Étape 2 - Lancement d'un premier conteneur

Votre première image sera l’image du serveur Web Nginx. Le registre par défaut d’images de Docker est docker hub ([hub.docker.com](https://hub.docker.com)).

Pour lancer votre premier conteneur, exécuter la commande suivante :

```bash
docker container run --publish 80:80 nginx
```

L’utilisation de la commande `docker container run` :

1.	Recherche l’image nginx localement dans le cache d'images, ne la trouve pas.
2.	Recherche ensuite dans le registre d'images distant (par défaut Docker Hub).
3.	Télécharge la dernière version (nginx:latest par défaut).
4.	Crée un nouveau conteneur basé sur cette image et se prépare à le lancer.
5.	Lui donne une adresse IP virtuelle sur un réseau privé à l'intérieur de l’engin docker.
6.	Ouvre le port 80 sur l'hôte et transfère au port 80 dans le conteneur.
7.	Démarre le conteneur en utilisant la commande spécifiée (à voir plus loin).

Ouvrir un navigateur et entrer localhost (ou 127.0.0.1) à la barre d’adresses (http://localhost).  

Qu’avez-vous reçu ?
<details>
  <summary markdown="span">Réponse.</summary>
	La page par défaut du serveur Web Nginx.
</details>

Pour arrêter le conteneur, faire `Ctrl-c` à la fenêtre de commande.  

**Attention :** dans Windows, il se peut que le conteneur continue de s’exécuter en arrière-plan. Après le `Ctrl-c`, vous devez faire la commande `docker container stop <conteneur id/nom>`, nous allons voir cette commande plus en détail plus loin.

### Étape 3 - Conteneur en arrière-plan

Nous allons changer la commande pour nous permettre de reprendre le contrôle de la ligne de commande.

```bash
docker container run --publish 80:80 --detach nginx
```

Vérifier sur votre navigateur que le conteneur s’exécute.

Pour vérifier les conteneurs qui s’exécutent, exécuter la commande :

```bash
docker container ls
```

![docker container ls](../images/container_list.png)

Chaque conteneur a un ID et un nom (NAMES). Si l’on ne spécifie pas de nom, un nom au hasard est utilisé. L’ID et le nom doivent être uniques pour tous nos conteneurs.  

La colonne PORTS indique que toutes nos adresses IP sont reliées au port 80 qui est redirigé au port 80 TCP du conteneur.  

Pour arrêter le conteneur, exécuter la commande : 

```bash
docker container stop container_id
```

**Remarque :** vous devez entrer juste assez de caractères de l’ID pour le rendre unique.  

Vérifier que le conteneur est arrêté :

```bash
docker container ls
```

Cette commande affiche les conteneurs qui s’exécutent.

Pour afficher tous les conteneurs, exécuter la commande :

```bash
docker container ls -a
```

Avez-vous seulement un conteneur ?
<details>
  <summary markdown="span">Réponse.</summary>
	Non, plusieurs. Tant que vous n'avez pas effacé votre conteneur <code>docker container rm ...</code>, il reste présent sur la machine.
</details>


À chaque fois que vous exécutez la commande `docker container run`, un nouveau conteneur est lancé.  

Lancer à nouveau le conteneur, mais en spécifiant un nom :

```bash
docker container run --publish 80:80 --detach --name serveurweb nginx
```

Vérifier vos conteneurs avec la commande : `docker container ls -a`  

Avez-vous un nouveau conteneur ? Si oui, quel est son nom ?
<details>
  <summary markdown="span">Réponse.</summary>
	Oui, serveurweb.
</details>

### Étape 4 - Journaux et processus

Utiliser un navigateur pour vérifier que le conteneur Nginx s’exécute. Rafraichir la page de votre navigateur quelques fois.  

Avant de « détacher » votre conteneur, les journaux s’affichaient à l’écran, mais plus maintenant.  

Exécuter la commande suivante pour voir vos journaux :

```bash
docker container logs serveurweb
```

Ajouter le paramètre `--help` à la fin de la commande et indiquer le paramètre qui permet l’affichage suivant.  

Afficher les journaux en continu :
<details>
  <summary markdown="span">Réponse.</summary>
	
	-f

</details>

Afficher les journaux depuis un temps en particulier :
<details>
  <summary markdown="span">Réponse.</summary>

	--since

</details>

Afficher les 10 dernières lignes :  
<details>
  <summary markdown="span">Réponse.</summary>

	--tail 10

</details>

Exécuter la commande suivante pour afficher les processus qui s’exécutent dans le conteneur.

```bash
docker container top serveurweb
```

### Étape 5 - Suppression des conteneurs

Afficher tous les conteneurs et prendre en note les trois premiers caractères de l’ID de chacun.  

La commande pour effacer des conteneurs est `docker container rm ID`. (ID doit contenir assez de caractères pour le rendre unique.)  

Utiliser la commande suivante pour effacer tous vos conteneurs.

```bash
docker container rm ID1 ID2 ID3
```

Avez-vous réussi à effacer tous les conteneurs ?
<details>
  <summary markdown="span">Réponse.</summary>
	Non.
</details>

Normalement, un conteneur doit être arrêté pour pouvoir l’effacer.

Utiliser la commande suivante pour forcer à effacer le conteneur qui s’exécute toujours.

```bash
docker container rm -f ID
```

Un petit truc pour supprimer tous les conteneurs (Linux ou Git Bash sur Windows) :

```bash
docker container stop $(docker ps -a -q)
docker container rm $(docker ps -a -q)
# ou
docker container rm -f $(docker ps -a -q)
```

Vérifier que les conteneurs sont effacés : `docker container ls -a`.

Voici un bref résumé de la commande `docker container run`.  
![docker container run](../images/docker_container_run.png)

## Section 2 - Gestion de plusieurs conteneurs

Dans cette partie, vous allez lancer par vous-même plusieurs conteneurs.  

Je vous recommande de consulter le site Web [docs.docker.com](https://docs.docker.com).  

Utiliser le paramètre `--help` pour vous aider dans les commandes Docker.  

### Étape 1 - Lancement de trois conteneurs
1.	Lancer les serveurs nginx, mysql et httpd (apache).
2.	Les conteneurs doivent être « détachés » : utiliser `--detach` (ou `-d`).
3.	Nommer vos conteneurs (`--name`).
4.	Pour les ports d’écoute : nginx devrait écouter sur 80:80, httpd sur 8080:80, mysql sur 3306:3306.
5.	Lors de l'exécution de mysql, utiliser l'option `--env` (ou `-e`) pour passer `MYSQL_RANDOM_ROOT_PASSWORD=yes`.
6.	Utiliser les journaux du conteneur mysql pour trouver le mot de passe aléatoire qu'il a créé au démarrage.
7.	Nettoyer tout avec les commandes `docker container stop` et `docker container rm` (les deux peuvent accepter plusieurs noms ou ID).
8.	Utiliser la commande `docker container ls` pour vous assurer que tout est correct avant et après le nettoyage.

<details>
  <summary markdown="span">Réponse.</summary>

```bash
docker container run -p 80:80 --name serveurweb -d nginx
docker container run -p 8080:80 --name apache -d httpd
docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes -p 3306:3306 mysql
docker container logs mysql | grep PASSWORD
docker container ls
curl localhost
curl localhost:8080
docker container ls
docker container stop {ContainerIDs}
docker container ls -a
docker container rm {ContainerIDs}
docker container ls -a
docker image ls  
```  
	
</details>



## Pour vérification
Remettre une capture d'écran (image jpg, png, gif) démontrant vos 3 conteneurs s'exécutant.  

![Exemple de réponse.](../images/exer7Remise.png)  

## Références
[https://docs.docker.com/](https://docs.docker.com/)

---

*Version 2026-11-18*
