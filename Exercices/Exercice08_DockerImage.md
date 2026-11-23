# Exercice 8 - Docker : image

### Informations

- Type de travail : individuel
- Durée estimée : 2 heures
- Système d'exploitation : Ubuntu client (ou WSL sous Windows)
- Environnement : Docker

### Objectifs

- Découvrir la création d'image de conteneur.
- Écrire un Dockerfile simple.
- Construire une image avec `docker build`.
- Distinguer les instructions `ENTRYPOINT` et `CMD`.
- Créer une image à partir d'une image officielle.
- Gérer l'espace disque utilisé par Docker.

## Section 1 - Créer le programme hello-world

### Étape 1 - Installation de g++

- Sous Windows : lancer une console Ubuntu.
- Sous Linux : lancer un terminal.
- Vérifier si le programme g++ est installé (`g++ --help`).
- S'il n'est pas installé, l'installer (package g++).

<details>
  <summary markdown="span">Installation de g++</summary>

```bash
sudo apt update && sudo apt install g++ -y
```

Si vous avez des problèmes avec l'installation de g++ : [https://stackoverflow.com/questions/66869441/unable-to-install-g-and-build-essentials-in-ubuntu-20-04](https://stackoverflow.com/questions/66869441/unable-to-install-g-and-build-essentials-in-ubuntu-20-04).

</details>

### Étape 2 - Premier programme et première image

- Créer le fichier `premier-programme.cpp` avec le contenu suivant :

```cpp
#include <iostream>

int main(int argc, char** argv) {
    std::cout << "Bonjour à tous" << std::endl;
}
```

- Pour compiler votre programme, utiliser la commande `g++ premier-programme.cpp -o premier-programme -static`.
- Créer, dans le même répertoire que votre fichier `premier-programme` précédemment compilé, le fichier `Dockerfile` avec le contenu suivant :

```dockerfile
FROM scratch
COPY premier-programme /
ENTRYPOINT ["/premier-programme"]
```

- À partir du fichier et de la documentation, décrire ce que fait le fichier Dockerfile.

<details>
  <summary markdown="span">Réponse.</summary>

- `FROM scratch` : part d'une image vide (aucun système d'exploitation), d'où la compilation avec `-static`.
- `COPY premier-programme /` : copie l'exécutable compilé sur l'hôte à la racine de l'image.
- `ENTRYPOINT ["/premier-programme"]` : définit le programme exécuté au démarrage du conteneur.

</details>
<br>  

- Taper la commande `docker build --tag premier-programme:latest .`
- Vérifier que l'image a bien été créée : `docker image ls`.
- Lancer un conteneur qui utilise votre image :

```bash
docker container run --rm premier-programme
```

### Étape 3 - Passage d'un paramètre

- Modifier le programme précédent pour qu'il corresponde au listage suivant (n'oubliez pas de compiler de nouveau) :

```cpp
#include <iostream>

int main(int argc, char** argv) {
    std::cout << argv[1] << std::endl;
}
```

- Modifier votre Dockerfile afin que par défaut, le programme reçoive "!!Bonjour à tous!!" en paramètre.

<details>
  <summary markdown="span">Modification du Dockerfile</summary>

```dockerfile
FROM scratch
COPY premier-programme /
ENTRYPOINT ["/premier-programme"]
CMD [ "!!Bonjour à tous!!" ]
```

</details>
<br>  

- Reconstruire votre image de conteneur.
- Essayer de lancer un nouveau conteneur en utilisant les mêmes lignes de commandes que précédemment.
- Essayer de lancer un nouveau conteneur en ajoutant à la fin la chaîne de caractères `"Un paramètre qui vient de l'extérieur du conteneur"`.

## Section 2 - Créer une image à partir d'une autre image

Une image est construite à partir d’un fichier `Dockerfile` : si vous voulez utiliser un autre nom de fichier, vous devez utiliser la commande :

```bash
docker build -f un_fichier_dockerfile .
```

### Étape 1 - Consultation de la documentation

- Consulter le Dockerfile de nginx : [https://github.com/nginx/docker-nginx/blob/master/mainline/debian/Dockerfile](https://github.com/nginx/docker-nginx/blob/master/mainline/debian/Dockerfile)
- Consulter le site [https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/) pour comprendre les commandes `FROM`, `ENV`, `RUN`, `EXPOSE`, `ENTRYPOINT`, `CMD`. Vous pouvez constater que nginx s'exécute à partir d'une image Debian minimale.

Quelle est la différence entre `ENTRYPOINT` et `CMD` ?
<details>
  <summary markdown="span">Réponse.</summary>

[The Difference Between CMD and ENTRYPOINT in Docker Images](https://www.howtogeek.com/devops/the-difference-between-cmd-and-entrypoint-in-docker-images/)

</details>

### Étape 2 - Construction d'une image à partir de nginx

- Créer un répertoire de travail : par exemple `Docker`.
- Dézipper le fichier [dockerfile-exemple.zip](extra/dockerfile-exemple.zip) dans votre répertoire de travail.
- Aller dans le répertoire `dockerfile-exemple`. Ouvrir le fichier `Dockerfile`. Le fichier `index.html` va nous servir « d’application » à intégrer dans notre serveur nginx.
- Voici ce que fait le `Dockerfile` :
	- Nous allons utiliser une image officielle (la commande `FROM`). On peut utiliser une image officielle comme image de départ, c’est même recommandé et ça rend notre tâche beaucoup plus simple.
	- Par la suite, nous allons nous placer dans le répertoire `/usr/share/nginx/html` (la commande `WORKDIR`). Contrairement à `RUN cd …`, qui ne s'applique qu'à cette seule commande `RUN`, `WORKDIR` change le répertoire pour toutes les instructions suivantes.
	- Finalement, nous allons copier le fichier `index.html` de notre répertoire hôte dans le répertoire `/usr/share/nginx/html` de notre conteneur : on n’inclut pas le chemin (path), car nous sommes déjà dans le répertoire.
	- On n'a pas à spécifier `EXPOSE` pour les ports ou `CMD`, car ils sont dans l'image initiale incluse avec la commande `FROM`. De là l’avantage d’utiliser une image existante.
- Exécuter la commande pour construire votre image et exécuter un conteneur (le `-t` ajoute un tag à notre image, il y a un point à la fin).

```bash
docker image build -t monimage .
docker container run -p 8080:80 --rm monimage
```

- Ouvrir un navigateur à http://localhost:8080.
- Votre fichier index s’affiche-t-il ? (Si vous avez la page de nginx, vous pouvez essayer de rafraichir la page ou de vider le cache de votre navigateur.)

<details>
  <summary markdown="span">Réponse.</summary>

Oui, la page personnalisée « Vous venez d'exécuter avec succès un conteneur avec un fichier personnalisé copié dans l'image au moment de la construction! » remplace la page par défaut de nginx.

</details>

- Faire `Ctrl-c` pour arrêter votre conteneur.
- Vérifier les images de votre hôte pour voir que votre nouvelle image s’y trouve.

## Section 3 - Un peu de nettoyage

### Étape 1 - Les commandes prune

Les images et les conteneurs que vous utilisez prennent de l’espace sur votre disque, il faut donc parfois faire un peu de ménage.  
Vous pouvez utiliser les commandes `prune` pour nettoyer les images, les volumes, le cache de construction (*build cache*) et les conteneurs.  
Par exemple, pour nettoyer les images « pendantes » :

```bash
docker image prune
```

Ou, pour tout nettoyer (conteneurs arrêtés, réseaux inutilisés, images pendantes et cache de construction ; les volumes ne sont supprimés qu'avec l'option `--volumes`) :

```bash
docker system prune
```

Pour nettoyer toutes les images non utilisées :

```bash
docker image prune -a
```

### Étape 2 - Nettoyage et vérification de l'espace

- Nettoyer les images non utilisées de votre système.
- Utiliser la commande suivante pour voir l'utilisation de l'espace par Docker :

```bash
docker system df
```

N'oubliez pas que chacune de ces commandes a des options que vous découvrez avec `--help`.  
Par contre, Docker Hub limite le nombre de téléchargements d’images (*pulls*) par période de 6 heures : 100 sans connexion (par adresse IP) et 200 avec un compte personnel gratuit connecté (`docker login`). Voir [https://docs.docker.com/docker-hub/usage/pulls/](https://docs.docker.com/docker-hub/usage/pulls/). Vous devez donc bien équilibrer la gestion de votre espace local et le téléchargement de nouvelles images.

## Références
- [https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)
- [https://www.howtogeek.com/devops/the-difference-between-cmd-and-entrypoint-in-docker-images/](https://www.howtogeek.com/devops/the-difference-between-cmd-and-entrypoint-in-docker-images/)
- [https://docs.docker.com/docker-hub/usage/pulls/](https://docs.docker.com/docker-hub/usage/pulls/)

---

*Version 2026-11-23*
