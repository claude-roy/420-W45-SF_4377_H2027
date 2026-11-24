# Exercice 11 - DNS

### Informations

- Type de travail : individuel
- Durée estimée : 2 heures
- Système d'exploitation : Ubuntu client et Windows
- Environnement : virtuel et Docker

### Objectifs

- Interroger un serveur DNS avec `nslookup` et `host`.
- Distinguer une réponse qui fait autorité d'une réponse qui ne fait pas autorité.
- Reproduire une résolution itérative à partir d'un serveur racine.
- Identifier la configuration DNS d'un poste Linux (`nmcli`, `/etc/hosts`, `systemd-resolved`).
- Faire communiquer des conteneurs par leur nom grâce au DNS de Docker.

Dans cet exercice, vous allez reproduire à la main le travail d'un serveur DNS récursif, en partant d'un serveur racine d'Internet.

Vous allez également comprendre comment les conteneurs se « trouvent » dans Docker et comprendre que le DNS est la clé pour faciliter les communications entre conteneurs. Vous allez aussi découvrir comment cela fonctionne par défaut avec les réseaux personnalisés.

Vous avez vu que les conteneurs se retrouvent dans un réseau virtuel de Docker et chacun des conteneurs se voit attribuer une adresse IP. Par contre, les conteneurs ne doivent pas compter sur les adresses IP pour l'intercommunication, car ces adresses peuvent changer. Docker possède un DNS intégré qui utilise les noms des conteneurs (il est également possible de créer des alias) comme enregistrement. Il est donc possible d’utiliser ce DNS dans vos réseaux personnalisés (le réseau `bridge` par défaut n’offre pas de résolution de noms).

## Section 1 - DNS et récursivité

### Étape 1 - Sous Windows

Utiliser un poste Windows, soit votre propre ordinateur ou une VM, et répondre aux questions.

- Quel est le serveur de noms (DNS) pour votre poste client ?

<details>
  <summary markdown="span">Réponse.</summary>

La réponse varie.

</details>
<br>

- Quelle commande avez-vous utilisée ?

<details>
  <summary markdown="span">Réponse.</summary>

`nslookup`

</details>
<br>

- Faire un `nslookup` sur le nom de domaine : `csfoy.ca`

Qui vous a répondu ? Est-ce que la réponse fait autorité (selon votre version de Windows, il se peut que vous n'ayez pas l'information) ?

<details>
  <summary markdown="span">Réponse.</summary>

Votre serveur DNS.  
Si vous êtes à l'intérieur du cégep, oui. Sinon, non.

</details>
<br>

- Faire un `nslookup` sur le nom de domaine : `quebec.ca`

Qui vous a répondu ? Est-ce que la réponse fait autorité (selon votre version de Windows, il se peut que vous n'ayez pas l'information) ?

<details>
  <summary markdown="span">Réponse.</summary>

Votre serveur DNS.  
Non, elle ne fait pas autorité.

</details>
<br>

- Essayer à nouveau avec un `nslookup` sur le nom de domaine : `www.google.com`

Qui vous a répondu ? Est-ce que la réponse fait autorité (selon votre version de Windows, il se peut que vous n'ayez pas l'information) ?

<details>
  <summary markdown="span">Réponse.</summary>

Votre serveur DNS.  
Non, elle ne fait pas autorité.

</details>
<br>

- À partir des expériences précédentes, à quel moment la réponse fait autorité ?

<details>
  <summary markdown="span">Réponse.</summary>

La réponse fait autorité lorsque le serveur DNS qui nous répond est un des serveurs responsables de la zone (le domaine) demandée, c'est-à-dire un serveur qui contient lui-même les enregistrements de ce domaine.

</details>
<br>

### Étape 2 - Sous Linux

- Utiliser la commande `nmcli device show` pour trouver les informations suivantes :

| Information | Valeur |
|:--|:--|
| Nom de l’interface de sortie | |
| Adresse IP | |
| Préfixe réseau | |
| Passerelle | |
| **Configuration DNS** | |
| Serveurs | |
| Domaine | |
| Interfaces | |

<details>
  <summary markdown="span">Réponse.</summary>

Les valeurs varient selon votre poste. Dans le résultat de `nmcli device show`, ces informations se trouvent dans les champs `GENERAL.DEVICE` (nom de l’interface), `IP4.ADDRESS[1]` (adresse IP et préfixe), `IP4.GATEWAY` (passerelle), `IP4.DNS[1]` (serveurs DNS) et `IP4.DOMAIN[1]` (domaine). La commande `resolvectl status` affiche aussi la configuration DNS de chaque interface.

</details>
<br>

- Exécuter la commande `cat /etc/hostname`. Quelle est l'utilité de ce fichier ?

<details>
  <summary markdown="span">Réponse.</summary>

Il contient le nom de l'hôte.

</details>
<br>

- Exécuter la commande `cat /etc/hosts`. Quelle est l'utilité de ce fichier ?

<details>
  <summary markdown="span">Réponse.</summary>

Il permet de faire une correspondance entre un nom d'hôte et une adresse IP. Le système le consulte avant le serveur DNS (voir la ligne `hosts:` de `/etc/nsswitch.conf`). Par contre, `nslookup` et `host` l'ignorent : ils interrogent directement le serveur DNS.

</details>
<br>

**Remarque :** si la commande `nslookup` ou `host` est introuvable, installer le paquet `bind9-dnsutils` :

```bash
sudo apt install bind9-dnsutils
```

- Exécuter la commande `nslookup`, puis entrer `localhost` et appuyer sur `Entrée`. Quel est le serveur de noms (DNS) pour votre hôte ?

<details>
  <summary markdown="span">Réponse.</summary>

Le serveur est l'hôte local, `127.0.0.53`, au port 53, le service `systemd-resolved`.

</details>
<br>

- À partir de `nslookup`, rechercher le nom de domaine `csfoy.ca`

Qui vous a répondu ? Est-ce que la réponse fait autorité ?

<details>
  <summary markdown="span">Réponse.</summary>

Le serveur de l'hôte local (`systemd-resolved`).  
La réponse ne fait pas autorité, même sur le réseau du cégep, car `systemd-resolved` relaie la question.

</details>
<br>

- Encore à partir de `nslookup`, rechercher le nom de domaine `quebec.ca`.

Qui vous a répondu ? Est-ce que la réponse fait autorité ?

<details>
  <summary markdown="span">Réponse.</summary>

Le serveur de l'hôte local.  
Essayer d'entrer une des adresses IP reçues dans `nslookup` (recherche inverse).  
La réponse ne fait pas autorité.

</details>
<br>

- Quitter `nslookup` (la commande `exit`) et exécuter maintenant la commande `host csfoy.ca`.

La réponse est-elle la même qu'avec la commande `nslookup` ?

<details>
  <summary markdown="span">Réponse.</summary>

L'adresse IP est la même.

</details>
<br>

- Ajouter maintenant l'option `-v` (`host -v csfoy.ca`).

Quelle est la différence ?

<details>
  <summary markdown="span">Réponse.</summary>

Nous avons beaucoup plus de détails sur la configuration du serveur DNS du domaine `csfoy.ca`.

</details>
<br>

- Exécuter maintenant la commande `host -v www.csfoy.ca`.

Que remarquez-vous de différent ? De quel type d'enregistrement est-il question ? Où pointe-t-elle ?

<details>
  <summary markdown="span">Réponse.</summary>

L'information de l'entrée `www.csfoy.ca` a été ajoutée.  
C'est un enregistrement de type A : nom vers une adresse IPv4.  
Elle pointe vers la même adresse IP que `csfoy.ca`.

</details>
<br>

Notre serveur DNS fait une recherche récursive pour trouver l'information de `www.csfoy.ca`. Vous allez reproduire ce cheminement avec la commande `host`.

- Nous allons utiliser l'outil `host` pour chercher à résoudre le FQDN `www.csfoy.ca` non plus en posant la question à notre serveur DNS récursif, mais en partant d'un serveur racine d'Internet (le serveur `j.root-servers.net` avec l'adresse IP `192.58.128.30`), en utilisant la commande comme ceci :

```bash
host -v www.csfoy.ca 192.58.128.30 | less
# Appuyer sur q pour sortir de less.
```

- Le serveur `j.root-servers.net` ne nous répond pas directement, comme nous pouvions nous en douter. En revanche, il nous envoie la liste des serveurs DNS compétents dans le TLD `ca`. Reposons donc la question au premier de la liste :

```bash
host -v www.csfoy.ca 185.159.196.2
```

**Attention :** utiliser l'adresse IP affichée à votre écran plutôt que celle donnée ici ; il se peut qu'elle soit identique.

Qui fait autorité pour le domaine du cégep ?

<details>
  <summary markdown="span">Réponse.</summary>

Plusieurs serveurs font autorité sur le domaine du cégep. Vous avez des serveurs du RISQ, car le cégep est associé au RISQ (Réseau d'informations scientifiques du Québec).  
Les autres serveurs, `margelle.cegep-ste-foy.qc.ca` et `temporel.cegep-ste-foy.qc.ca`, sont ceux du cégep. Vous pouvez remarquer qu'ils sont encore inscrits sous l'ancien nom de domaine du cégep.

</details>
<br>

- Essayer de refaire le cheminement à partir d'un serveur racine de votre choix (voir la liste à [https://www.root-servers.org/](https://www.root-servers.org/)) avec un domaine possédant un TLD (Top Level Domain) différent.

<details>
  <summary markdown="span">Réponse.</summary>

La réponse varie selon le domaine choisi. Le cheminement reste le même : le serveur racine donne la liste des serveurs du TLD, un serveur du TLD donne la liste des serveurs qui font autorité pour le domaine, et un de ces serveurs donne la réponse.

</details>
<br>

Et voilà le travail. Nous pouvons constater à quel point il peut être fastidieux de trouver une résolution d’adresse sans un serveur récursif. Heureusement, nous disposons d'un bon serveur DNS récursif, qui fait tout ce travail à notre place. C'est exactement ainsi qu'il s'y prend pour obtenir la réponse.

Un DNS récursif garde en mémoire ce qu'il trouve en effectuant cette recherche et s'en resservira pour d'éventuelles résolutions futures. Les serveurs « qui font autorité » indiquent une durée de validité pour les informations qu'ils donnent. Ainsi, les serveurs récursifs devront rafraîchir le contenu de leur cache en fonction de cette durée de validité.

## Section 2 - DNS sous Docker

Dans cette section, vous allez essayer de comprendre la communication entre conteneurs avec le DNS interne de Docker.

### Étape 1 - Communication dans le réseau bridge par défaut

- Lancer un conteneur Nginx.

```bash
docker container run -d --name serveurweb nginx
```

- Trouver l'adresse IP du conteneur.

```bash
docker container inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' serveurweb
```

- Nous allons essayer de faire un ping sur l'adresse IP et sur le nom du conteneur. Pour ce faire, nous allons utiliser un conteneur `alpine`.

```bash
# Remplacer <adresse_IP> par l'adresse IP trouvée à l'étape précédente.
# L'option --rm supprime le conteneur alpine à la fin du ping.
docker container run --rm alpine ping -c 4 <adresse_IP>
docker container run --rm alpine ping -c 4 serveurweb
```

Avez-vous réussi à faire un ping avec le nom du conteneur ?

<details>
  <summary markdown="span">Réponse.</summary>

Non, seulement avec l'adresse IP. Le serveur DNS de Docker n'est pas fonctionnel dans le réseau `bridge` par défaut.

</details>
<br>

- Supprimer votre conteneur.

```bash
docker container rm -f serveurweb
```

### Étape 2 - Communication dans un réseau personnalisé

- Nous allons maintenant créer un réseau personnalisé et relier le conteneur à ce réseau.

```bash
docker network create net_mon_app
docker container run -d --name serveurweb --network net_mon_app nginx
```

- Vérifier que tout est fonctionnel.

```bash
docker network ls
docker container ls
docker network inspect net_mon_app
```

- Vous allez faire un ping sur le conteneur `serveurweb` avec un conteneur `alpine` dans le même réseau.

```bash
docker container run --rm --network net_mon_app alpine ping -c 4 serveurweb
```

Avez-vous réussi à faire un ping avec le nom du conteneur ?

<details>
  <summary markdown="span">Réponse.</summary>

Oui, le serveur DNS de Docker est fonctionnel dans notre réseau personnalisé.

</details>
<br>

Vous pouvez constater que la résolution DNS fonctionne.

- Supprimer le conteneur et le réseau.

```bash
docker container rm -f serveurweb
docker network rm net_mon_app
```

## Références

- [https://www.root-servers.org/](https://www.root-servers.org/)
- [https://networkmanager.dev/docs/api/latest/nmcli.html](https://networkmanager.dev/docs/api/latest/nmcli.html)
- [https://docs.docker.com/engine/network/](https://docs.docker.com/engine/network/)
- [https://docs.docker.com/engine/network/drivers/bridge/](https://docs.docker.com/engine/network/drivers/bridge/)

---

*Version 2026-11-24*
