# Cours Docker

Cours dispensé par OpenClassrooms

## Table of contents

- [1 - Découvrez les conteneurs](#1)
- [2 - Découvrez ce qu'est Docker](#2)
- [3 - Installez Docker](#3)
- [4 - Premier conteneur en local](#4)
- [5 - Premier Dockerfile](#5)
- [6 - Utiliser des images partagé via Docker Hub](#6)
- [7 - Installez et découvrez Docker Compose](#7)
- [8 - Création d'un premier docker-compose](#8)


## 1 - Découvrez les conteneurs <a name="1"></a>

**Les Machines Virtuelles (virtualisation)**

- virtualisation lourde, on recrée un système complet dans le système hôte, pour qu’il ait ses propres ressources.
- l’isolation des VMs se fait au niveau matérielles (CPU/RAM/Disque) avec un accès virtuel aux ressources de l'hôte via l'hyperviseur
- Si on est intéressé que par l'application qui tourne sur la VM et pas toute l'infrastructure, on fait appel à la conteneurisation

| + | - |
| :--------------- | :--------------- |
| totalement isolée du système hôte  |  prend du temps à démarrer |
| ressources attribuées à une machine virtuelle lui sont totalement réservées  | réserve les ressources (CPU/RAM) sur le système hôte. (notamment pour faire tourner la partie OS)|


**Conteneurisation**

- Un conteneur va s'exécuter sous Linux de manière native et va partager le noyau de la machine hôte avec d'autres conteneurs (gain en mémoire)
- Un conteneur Linux est un processus ou un ensemble de processus isolés du reste du système, tout en étant légers.
- virtualisation légère, c'est-à-dire qu'il ne virtualise pas les ressources, il ne crée qu'une isolation des processus.
- Le conteneur partage donc les ressources avec le système hôte.

| + |
| :--------------- |
| Ne réservez que les ressources nécessaires |
| Démarrez rapidement vos conteneurs |
| Donnez plus d'autonomie à vos développeurs |

![](./images/conteuneurisation.png)

## 2 - Découvrez ce qu'est Docker <a name="2"></a>

- Dans la vision Docker, un conteneur ne doit faire tourner qu'un seul processus (une stack avec 4 services = 4 conteneurs différents)
- un des premiers usages de Docker se trouve dans la création d'environnements locaux
- On retrouve aussi Docker dans les domaines de la CI/CD pour créer rapidement des espaces isolés pour faire tourner vos tests

**Stateless et immutabilité**

- stateful vs stateless
    - stateful = stock un état (base de données MySQL)
    - stateless = ne stock pas d'état (protocole HTTP)
- immuabilité
    - def = ce qui ne change pas ou peu
    - Un conteneur ne doit pas stocker de données qui doivent être pérennes, car il les perdra
    - mettre en local une base de données dans un conteneur Docker, vous devez créer un volume pour que celui-ci puisse stocker les données de façon pérenne.

##  3 - Installez Docker <a name="3"></a>

[*Lien vers la doc*](https://docs.docker.com/engine/install/ubuntu/)

Pour installer Docker sur Linux (Ubuntu), il faut suivre les commandes suivantes :

1. Mettre à jour l'index du package `apt` et installer les paquets pour permettre l'utilisation d'un repertoire avec le protocole HTTPS
2. Ajouter la clé GPG Docker officiel
3. Mettre en place le repertoire `stable`
4. Installer Docker Engine
5. Vérifier que Docker est bien installé

```console
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world
```

Si l'on souhaite se passer de `sudo` en ce donnant les privilèges administrateur :

```
# list of users
cut -d: -f1 /etc/passwd

# get privileges
sudo usermod -aG docker <your-user>
```

Si on obtient l'erreur suivante: 

```
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
```

Lancez la commande `sudo dockerd`, dockerd est le daemon pour les conteneurs Docker.

Si vous avez besoin de vous connectez à Docker Hub, lancez la commande `docker login`

##  4 - Premier conteneur en local <a name="4"></a>

Docker Hub = la registry officielle de Docker
- Une registry est un logiciel qui permet de partager des images à d'autres personnes.

Quelques commandes Docker pour lancer et arrêter un conteneur Docker en local : 

| Commande | Description |
| :--------------- | :--------------- |
|  `docker ps`| Liste des conteneurs Docker en marche
|  `docker images`| Liste de nos images Docker
|  `docker pull <nom_image>`| Récupère une image Docker du Docker Hub
| `docker run -it -d -p 8080:80 <nom_image>` | Lance un conteneur docker (en mode "detached" et "interactive"), qui transfère le trafic du port 8080 vers le port 80 du conteneur
|  `docker stop <conteneur_id>`| Arrêt du conteneur docker spécifié en argument
|  `docker rm <conteneur_id>`| Supprime le conteneur et son contenu
|  `docker system prune`| Supprime l'ensemble des ressources en local (conteneurs, réseaux, images, caches)
|  `docker build -t <nom_image> <chemin_Dockerfile>`| Construit une image docker à partir du Dockerfile situé au chemin spécifié

##  5 - Premier Dockerfile <a name="5"></a>

- Pour créer sa propre image, il faut créer un fichier que l'on appellera `Dockerfile`. Ce fichier peut-être comparé à `package.json` en JavaScript.
- Chaque instruction va créer une nouvelle **layer** = étape de construction de l'image

Quelques instructions pour les Dockerfile : 

| Commande | Description |
| :--------------- | :--------------- |
|  `FROM` | Pour définir l'image que l'on utilise comme base
|  `RUN` | Pour exécuter une commande dans notre conteneur
|  `ADD` | Pour copier/télécharger des fichiers dans l'image
|  `WORKDIR` | Pour modifier le répertoire courant (=`cd`)
|  `EXPOSE` | Pour indiquer le port sur lequel votre application écoute
|  `VOLUME` | Pour indiquer quel répertoire vous voulez partager avec votre host
|  `CMD` | Pour indiquer à notre conteneur quelle commande il doit exécuter lors de son démarrage, à placer toujours à la fin d'un Dockerfile

Il suffit de construire l'image avec la commande : 

```
docker build -t <nom_image> <chemin_creation>`
docker images
```

Exempe de Dockerfile:

```
FROM debian:9

RUN apt-get update -yq \
&& apt-get install curl gnupg -yq \
&& curl -sL https://deb.nodesource.com/setup_10.x | bash \
&& apt-get install nodejs -yq \
&& apt-get clean -y

ADD . /app/
WORKDIR /app
RUN npm install

EXPOSE 2368
VOLUME /app/logs

CMD npm run start
```
**.dockerignore**

- A l'image de `.gitignore`, permet de ne pas copier certains fichiers dans notre conteneur lors de l'exécution de l'instruction `ADD`.
- A mettre à la racine de notre projet (au même niveau que le Dockerfile)

##  6 - Utiliser des images partagé via Docker Hub <a name="6"></a>

Pour mettre à disposition nos images à d'autres utilisateurs, il faut exécuter les commandes suivantes :

```
docker tag <nom_image>:latest <nom_utilisateur>/<nom_image>:latest
docker push <nom_utilisateur>/<nom_image>:latest
```

Vérifiez que vous êtes bien authentifié avec `docker login`.

On peut également chercher et télécharger des images Docker avec les commandes : 

```
docker search <nom_image>
docker pull <nom_image>
```

##  7 - Installez et découvrez Docker Compose <a name="7"></a>

**Installer Docker Compose**

Docker Compose est un outil écrit en Python qui permet de décrire, dans un fichier YAML, plusieurs conteneurs **comme un ensemble de services.**
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose && sudo chmod +x /usr/bin/docker-compose
```

**Découverte de Docker Compose**

*Stack = ensemble de conteneurs Docker*

| Commande | Description |
| :--------------- | :--------------- |
|  `docker-compoe up -d` | Démarre l'ensemble des conteneurs en arrière-plan
|  `docker-compoe ps` | Voir le statut de l'ensemble de votre stack
|  `docker-compoe logs -f --tail 5` | Affiche les logs de votre stack
|  `docker-compoe stop` | Arrête l'ensemble des services d'une stack
|  `docker-compoe down` | Détruit l'ensemble des ressources d'une stack
|  `docker-compoe config` | Valide la syntaxe de votre fichier `docker-compose.yml`

##  8 - Création d'un premier docker-compose <a name="8"></a>

Un fichier ``docker-compose.yml` commence toujours par l'argument version

```
version: '3'
```
Ensuite on décrit une liste de services.

```yml
service:
    <nom_service>:
    ...
```

Pour chaque service on peut (entre autres) définir:
- son nom
- son image
- (son build, avec l'argument `build:` en lui spécifiant le chemin vers un Dockerfile)
- son volume pour faire persister les données  
    - `db_data:/var/lib/mysql` permet de créer un volume `db_data` et de stocker les données au chemin spécifié
- sa politique de rédemarrage
    - un conteneur = monoprocessus
    - la valeur `always` = si le serveur s'arrête, il redémarrera automatiquement
- ses variables d'environnements

Exemple complet d'un fichier `docker-compose` avec 2 services :
```yml
version: '3'
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data: {}
```

Il suffit de lancer `docker-compose up -d` pour lancer les deux services/conteneurs