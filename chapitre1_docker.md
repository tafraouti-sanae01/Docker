# Créer un conteneur Docker

## les conteneur :
Les conteneurs sont créés à partir d'images de conteneurs.
**Explication :** Un conteneur est une instance en cours d'exécution d'une image. C'est comme un processus isolé qui contient votre application.

## les images :
Les images sont un système de fichiers compressé et pré-packagé qui contient votre application ainsi que son environnement et sa configuration, avec des instructions sur la façon de démarrer votre application. Cette instruction s'appelle le point d'entrée.
**Explication :** Une image Docker est comme un modèle (template) immuable. Elle contient tout ce dont votre application a besoin pour fonctionner : code, bibliothèques, dépendances, etc. 

```bash
# Syntaxe générale pour créer un conteneur
$ docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]

# Exemple : créer un conteneur à partir de l'image hello-world:linux
$ docker container create hello-world:linux

Unable to find image 'hello-world:linux' locally
linux: Pulling from library/hello-world
Digest: sha256:53cc1017c16ab2500aa5b5367e7650dbe2f753651d88792af1b522e5af328352
Status: Downloaded newer image for hello-world:linux
d576ebe76945c4e24b93eeb561c87eb9fa7ad8b1395c1fba46bfc44131910d5d  # ID du conteneur créé
```
**Important :** Cette commande crée des conteneurs, mais ne les démarre pas. Le conteneur est créé mais reste en état "Created". 

```bash
$ docker ps  # Afficher les conteneurs qui sont en cours d'exécution 
$ docker ps --all  # Afficher tous les conteneurs (actifs et arrêtés)

$ docker ps -all  # Forme longue

$ docker ps -a  # Forme courte (équivalent à --all)

# Démarrer un conteneur avec son ID complet
$  docker container start d576ebe76945c4e24b93eeb561c87eb9fa7ad8b1395c1fba46bfc44131910d5d
d576ebe76945c4e24b93eeb561c87eb9fa7ad8b1395c1fba46bfc44131910d5d

# Voir les logs d'un conteneur (3 premiers caractères de l'ID suffisent)
$ docker logs d57

# Démarrer et s'attacher au conteneur pour voir la sortie en direct
$ docker container start --attach d57
```
**Astuce :** Docker permet d'utiliser seulement les premiers caractères de l'ID d'un conteneur (généralement 3-4) pour le référencer.

```bash
docker ps --all
CONTAINER ID   IMAGE               COMMAND    CREATED          STATUS                     PORTS     NAMES
d576ebe76945   hello-world:linux   "/hello"   20 minutes ago   Exited (0) 2 minutes ago             awesome_germain
```

**Comprendre le code de sortie :**
- **Exited (0)** : Le point d'entrée a été exécuté avec succès. Le code 0 indique qu'il n'y a pas eu d'erreur.
- **Tout autre code de sortie** (par exemple Exited (1), Exited (137)) : Cela signifie probablement qu'il y a eu un problème lors de l'exécution. 

# Créer un conteneur Docker : la méthode la plus courte
```bash
# Cette commande crée ET démarre le conteneur en une seule étape
$ docker run hello-world:linux
```
**Explication :** `docker run` = `docker create` + `docker start` + `docker attach`. C'est la façon la plus rapide de lancer un conteneur.

Nous avons créé des conteneurs à partir d'images Docker publiées sur Docker Hub (le registre public d'images Docker). 

# Créer un conteneur Docker à partir de Dockerfiles, partie 1

## Dockerfile : 03_05\Dockerfile

**Les instructions principales d'un Dockerfile :**

**FROM :** Indique à Docker sur quelle image Docker existante baser votre image Docker.
→ C'est toujours la première instruction. Ex: `FROM ubuntu:22.04`

**USER :** Indique à Docker quel utilisateur utiliser pour toutes les commandes du Dockerfile situées en dessous. Par défaut, Docker utilise l'utilisateur racine "root" pour exécuter les commandes.
→ Utiliser un utilisateur non-root améliore la sécurité.

**COPY :** Copie les fichiers d'un répertoire fourni à la commande de génération Docker vers l'image du conteneur.
→ Ex: `COPY app.py /app/` copie app.py dans le dossier /app/ du conteneur.

**Le contexte de build :** Le répertoire fourni à la build Docker s'appelle le contexte. Le contexte est généralement votre répertoire de travail, mais ce n'est pas obligatoire.

**RUN :** Les instructions RUN sont des commandes qui personnalisent notre image.
→ Ex: `RUN apt-get update` ou `RUN pip install flask`

**ENTRYPOINT :** Le point d'entrée indique à Docker quelle commande les conteneurs créés à partir de cette image doivent exécuter au démarrage.
→ C'est la commande principale qui s'exécute quand le conteneur démarre.

```bash
# Construire une image Docker avec un tag (nom)
$ docker build -t our-first-image . 
# -t : donner un nom (tag) à l'image
# . : utiliser le répertoire courant comme contexte

# Exécuter un conteneur à partir de cette image
$ docker run our-first-image
```
**Note :** Le point `.` à la fin indique que le Dockerfile et les fichiers nécessaires sont dans le répertoire actuel.

# Créer un conteneur Docker avec BuildKit
**BuildKit** est le nouveau moteur de build de Docker, plus rapide et avec plus de fonctionnalités.

```bash
# Construire avec BuildKit
$ docker buildx build -t our-first-image --load . 
# --load : charger l'image dans le daemon Docker local après la construction

# Exécuter le conteneur avec suppression automatique après arrêt
$ docker run --rm our-first-image
# --rm : supprimer automatiquement le conteneur quand il s'arrête
```

# Interagissez avec votre conteneur
 ```bash
# Construire avec un Dockerfile spécifique
$ docker build --file server.Dockerfile --tag  our-first-server . 
# --file : spécifier un fichier Dockerfile différent du nom par défaut

# Lancer le conteneur (mode attaché, bloque le terminal)
$ docker run our-first-server

# Arrêter brutalement le conteneur
$ docker kill e0b
# kill : arrêt immédiat (SIGKILL)

# Lancer le conteneur en arrière-plan (mode détaché)
$ docker run -d our-first-server
# -d : mode détaché, le conteneur s'exécute en arrière-plan

# Exécuter une commande dans un conteneur en cours d'exécution
$ docker exec 4368 date
# Affiche la date actuelle à l'intérieur du conteneur

# Ouvrir un shell interactif dans le conteneur
$ docker exec --interactive --tty 4368 bash
# -i : mode interactif
# -t : allouer un pseudo-terminal
# Permet d'interagir avec le conteneur comme si vous y étiez connecté

```
# Arrêt et retrait du récipient
**Important :** Docker n'arrête pas ou ne supprime pas les conteneurs automatiquement pour vous. Vous devez les gérer manuellement.

```bash
# Arrêter un conteneur proprement (SIGTERM puis SIGKILL après 10s)
$ docker stop 436 

# Lancer un nouveau serveur en arrière-plan
$  docker run -d our-first-server 

# Lister les conteneurs actifs
$  docker ps

# Arrêter immédiatement sans timeout
$  docker stop -t 0 64b0 
# -t 0 : timeout de 0 secondes, arrêt immédiat

# Lister tous les conteneurs (actifs et arrêtés)
$ docker ps -a 

# Supprimer un conteneur arrêté
$  docker rm 64b0
# ⚠️ Cette commande n'arrêtera pas les conteneurs en cours d'exécution

# Forcer la suppression d'un conteneur en cours d'exécution
$ docker rm -f 
# -f : force la suppression même si le conteneur est actif

# Obtenir les IDs de tous les conteneurs
$ docker ps -aq 
# -a : tous les conteneurs
# -q : afficher uniquement les IDs (quiet mode)

# Supprimer tous les conteneurs en une seule commande
$ docker ps -aq | xargs docker rm  # Linux/Mac
# ou pour Windows PowerShell :
$ docker ps -aq | ForEach-Object { docker rm $_ }

# Vérifier qu'ils sont tous supprimés
$ docker ps -a

# Lister toutes les images
$ docker images 

# Supprimer une image spécifique
$ docker rmi our-first-server:latest
# rmi : remove image

# Forcer la suppression d'une image (même si utilisée)
$ docker rmi -f
```
# Liaison des ports à votre conteneur
**Pourquoi mapper les ports ?** Par défaut, les conteneurs sont isolés. Pour accéder à un service web dans le conteneur, il faut relier un port du conteneur à un port de votre machine.

```bash
# Construire l'image du serveur web
$ docker build -t our-web-server -f web-server.Dockerfile .

# Lancer le conteneur en arrière-plan avec un nom
$ docker run -d our-web-server
# Équivalent à : docker run -d --name our-web-server our-web-server

# Vérifier que le conteneur est actif
$ docker ps

# Consulter les logs du conteneur en utilisant son nom
$ docker logs our-web-server
# Utiliser le nom du conteneur au lieu de son ID est plus lisible
Server started. Visit http://localhost:5000 to use it.
# ⚠️ Cela ne fonctionne pas ! Le port 5000 est à l'intérieur du conteneur
# Il faut mapper le port 5000 du conteneur à un port sur notre machine réelle

# Arrêter et supprimer le conteneur
$ docker rm -f our-web-server
# Au lieu de faire : docker stop our-web-server && docker rm our-web-server

# Relancer avec le mapping de ports
$ docker run -d --name our-web-server -p 5001:5000 our-web-server 
```

**Explication du mapping de ports :**
- **-p 5001:5000** → fait le mapping de ports :
  - **5001** = port extérieur (celui de votre machine hôte que vous ouvrez dans le navigateur)
  - **5000** = port intérieur (celui exposé par votre conteneur)
- Maintenant vous pouvez accéder au serveur via http://localhost:5001


# Enregistrement des données des conteneurs
**Principe important :** Les conteneurs sont censés être jetables. Lorsqu'ils sont supprimés, ils sont supprimés pour de bon. Cela inclut toutes les données que vous y avez enregistrées. Lorsque le conteneur est supprimé, les données sont aussi supprimées.

**La solution :** Utiliser des volumes pour persister les données en dehors du conteneur. 
```bash
# Créer un fichier dans le conteneur et le lire
$ docker run --rm --entrypoint sh ubuntu -c  "echo 'Hello.'> /tmp/file && cat /tmp/file"
# --entrypoint sh : utiliser sh comme point d'entrée au lieu du point d'entrée par défaut
# -c : exécuter la commande qui suit
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
Digest: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Status: Downloaded newer image for ubuntu:latest
Hello.  # Le fichier existe dans le conteneur

# Essayer de lire le fichier depuis la machine hôte
$ cat /tmp/file
cat : Impossible de trouver le chemin d'accès « C:\tmp\file », car il n'existe pas.
# ❌ Le fichier n'existe pas sur la machine hôte car il était seulement dans le conteneur
# Le conteneur a été supprimé avec --rm, donc le fichier a disparu
Au caractère Ligne:1 : 1
+ cat /tmp/file
+ ~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\tmp\file:String) [Get-Content], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetContentCommand
```

**Solution :** On peut utiliser la fonction de montage de volume pour contourner ce problème. Cela permet à Docker de mapper un dossier de notre ordinateur à un dossier dans le conteneur. Les données persistent même après la suppression du conteneur.

```bash
# Monter un volume pour persister les données
$ docker run --rm --entrypoint sh -v /tpm/container:/tmp ubuntu -c  "echo 'Hello.'> /tmp/file && cat /tmp/file"
# -v /tmp/container:/tmp : monter (lier) le dossier de l'hôte au dossier du conteneur
Hello.
```

**Explication du volume :**
Nous avons mappé `/tmp/container` sur notre ordinateur à `/tmp` dans le conteneur.
- **Côté hôte :** /tmp/container (votre machine)
- **Côté conteneur :** /tmp
- Maintenant, nous devrions nous attendre à un fichier appelé `file` à l'intérieur de `/tmp/container` sur notre machine. 

```bash
# Exemple avec un chemin Windows
$ docker run --rm --entrypoint sh -v C:\tmp\container:/tmp ubuntu -c "echo 'Hello.'> /tmp/file && cat /tmp/file"
Hello.

# Lire le fichier depuis la machine hôte Windows
$ cat C:\tmp\container\file
Hello.
# ✅ Cette fois, le fichier existe sur notre machine car on a utilisé un volume!

# Créer un fichier sur l'hôte Windows (PowerShell)
$  ni C:\tmp\change_this_file -ItemType File
# ni : alias de New-Item (créer un nouveau fichier)


    Répertoire : C:\tmp


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        25/02/2026     11:48              0 change_this_file 

# Créer un fichier sur l'hôte Linux/Unix
$ touch /tmp/change_this_file 
# touch : créer un fichier vide

# Monter un fichier spécifique (pas seulement un dossier)
$ docker run --rm --entrypoint sh -v C:\tmp\change_this_file:/tmp/file ubuntu -c "echo 'Hello.'> /tmp/file && cat /tmp/file"
Hello.
# Le conteneur écrit dans /tmp/file, qui est lié à C:\tmp\change_this_file

# Vérifier que le fichier sur l'hôte a été modifié
$ cat C:\tmp\change_this_file
Hello.
# ✅ Le fichier de l'hôte contient maintenant "Hello."
```
**Note :** Vous pouvez monter des fichiers individuels, pas seulement des répertoires entiers.
# Présentation du Docker Hub
**Docker Hub** est le registre public officiel de Docker. C'est comme un "GitHub pour les images Docker". Vous pouvez y trouver des milliers d'images publiques et y publier vos propres images.

# Envoi d'images vers le registre Docker
**Pourquoi publier sur Docker Hub ?**
- Partager vos images avec d'autres
- Déployer facilement vos applications sur différentes machines
- Versionner vos images
```bash
# Se connecter à Docker Hub
$  docker login
# Utilise les identifiants stockés ou demande username/password
Authenticating with existing credentials... [Username: tafraoutisanae]

i Info → To login with a different account, run 'docker logout' followed by 'docker login'


Login Succeeded

# Taguer l'image avec votre nom d'utilisateur Docker Hub
$ docker tag our-web-server tafraoutisanae/our-web-server:0.0.1
# Format : docker tag <image-locale> <username>/<nom-image>:<version>

# Pousser (uploader) l'image vers Docker Hub
$ docker push tafraoutisanae/our-web-server:0.0.1
# L'image est maintenant disponible publiquement sur Docker Hub

# Créer une nouvelle version (tag)
$ docker tag our-web-server tafraoutisanae/our-web-server:0.0.2
# Même image, mais avec un tag différent pour indiquer une nouvelle version
 
# Pousser la nouvelle version
$ docker push tafraoutisanae/our-web-server:0.0.2
The push refers to repository [docker.io/tafraoutisanae/our-web-server]
282916e9436e: Already exists
77e4953fca22: Layer already exists
e3fd5ea081cf: Layer already exists
01d7766a2e4a: Layer already exists
4f4fb700ef54: Layer already exists
8b95e8db4e55: Layer already exists
0.0.2: digest: sha256:b5a0842aef78109fc9d16a497012d726f4f5b3a3de76631b8cf5891c05e2fcc6 size: 856
```
# Au-delà de Docker Hub : Autres registres de conteneurs populaires

**Alternatives à Docker Hub :**
 1. **GitHub Container Registry** (ghcr.io) - Intégré avec GitHub
 2. **GitLab Container Registry** - Intégré avec GitLab
 3. **Amazon ECR** (Elastic Container Registry)
 4. **Google Container Registry** (GCR)
 5. **Azure Container Registry** (ACR)

→ Ces registres peuvent être **privés** (accès restreint) contrairement à Docker Hub qui est principalement public.

# Démarrer NGINX
**NGINX** est un serveur web très populaire. Cette commande lance NGINX avec votre site web local.

```bash
$ docker run --name website -v "$PWD/website:/usr/share/nginx/html" -p 8081:80 --rm nginx
# --name website : nommer le conteneur "website"
# -v "$PWD/website:/usr/share/nginx/html" : monter le dossier website actuel dans le conteneur
# $PWD : variable pour "Present Working Directory" (répertoire actuel)
# -p 8081:80 : mapper le port 80 du conteneur (NGINX) au port 8081 de l'hôte
# --rm : supprimer automatiquement le conteneur quand il s'arrête
# nginx : nom de l'image à utiliser
```
**Résultat :** Votre site sera accessible sur http://localhost:8081
# Aide! Je n’arrive pas à créer plus de conteneurs
**Problème :** Vous manquez peut-être d'espace disque. Docker occupe beaucoup d'espace avec les images et conteneurs.

```bash
# Tester avec une image Java
$ docker run --name=app --rm eclipse-temurin:19 java -version
# Alternative : docker run --name=app --rm openjdk:19

# Vérifier l'espace disque disponible sur Windows
$ Get-PSDrive
# Affiche tous les lecteurs et leur espace disponible

# Vérifier l'espace disque sur Linux/Mac
$ df -h
# -h : format "human-readable" (Go, Mo, etc.)

# Lister toutes les images Docker (et leur taille)
$ docker images 

# Supprimer des images spécifiques pour libérer de l'espace
$ docker rmi nginx ubuntu

# Nettoyer tous les conteneurs arrêtés, réseaux inutilisés, images sans tag
$ docker system prune
# ⚠️ Cette commande libère beaucoup d'espace!
# Vous pouvez ajouter -a pour supprimer aussi les images inutilisées

# Réessayer après le nettoyage
$ docker run --name=app --rm eclipse-temurin:19 java -version
```

**Raccourcis terminal utiles :**
- **Ctrl+L** : vider/nettoyer le terminal
- **Ctrl+R** : chercher dans l'historique des commandes. Tapez un mot-clé et appuyez sur Ctrl+R pour naviguer entre les résultats.

# Aide! Mon conteneur est vraiment lent
**Problème :** Votre conteneur consomme trop de ressources (CPU, mémoire). Voici comment diagnostiquer.

## 1ère commande : docker stats
**docker stats** affiche l'utilisation des ressources en temps réel (comme le Gestionnaire des tâches).

```bash
# Lancer un conteneur Alpine qui dort indéfiniment
$ docker run --name=alpine --entrypoint=sleep -d alpine infinity
# alpine : image très légère basée sur Alpine Linux
# sleep infinity : garder le conteneur actif indéfiniment

# Vérifier que le conteneur tourne
$ docker ps

# Monitorer les statistiques du conteneur en temps réel
$ docker stats alpine
# Affiche : CPU%, MEM%, NET I/O, BLOCK I/O, etc.


# Dans un autre terminal : simuler une charge

# Ouvrir un shell dans le conteneur
$  docker exec -i -t alpine sh 

# Taper la commande "yes" : elle imprime "y" indéfiniment
# Cette commande consomme beaucoup de CPU

# Dans le 1er terminal, on peut voir que l'utilisation du processeur (CPU%) 
# a considérablement augmenté à cause de cela
```
**Astuce :** Utilisez Ctrl+C pour arrêter `docker stats` ou la commande `yes`.
## 2ème commande : docker top 
**docker top** affiche les processus en cours d'exécution dans le conteneur (comme la commande `top` Linux).

```bash
# Lancer plusieurs processus sleep en arrière-plan dans le conteneur
$ docker exec -d alpine sleep infinity 
$ docker exec -d alpine sleep infinity 
$ docker exec -d alpine sleep infinity 
# -d : mode détaché, chaque commande s'exécute en arrière-plan

# Afficher les processus en cours dans le conteneur
$  docker top alpine
# Montre tous les processus, leurs PIDs, et l'utilisation CPU

# Inspecter toutes les informations détaillées du conteneur
$ docker inspect alpine
# Retourne un JSON avec toutes les métadonnées : config réseau, volumes, variables d'environnement, etc.

# Afficher les infos avec pagination (Windows)
$ docker inspect alpine | more
# Sur Linux : docker inspect alpine | less
# Permet de naviguer page par page dans le résultat

```
**Note :** `docker inspect` est très utile pour déboguer des problèmes de configuration.
# Je n’arrive pas à utiliser le client Docker !
```bash
$ docker context ls
NAME              DESCRIPTION                               DOCKER ENDPOINT                             ERROR
default           Current DOCKER_HOST based configuration   npipe:////./pipe/docker_engine
desktop-linux *   Docker Desktop                            npipe:////./pipe/dockerDesktopLinuxEngine
# Le context vous permer de creer des raccourcis qui configurent la machine que le client Docker utilise pour effectuer des taches liees a docker 
# Ceux-ci vous permmettent de creer et d'executer des conteneurs a distance, ce qui peut etre tres utile si vous utilisez Docker dans un scenario automatise, comme dans un script ou dans une execution de pipline CI/CD 

$ docker context use default

$ docker context use desktop-linux

$ echo $env:DOCKER_CONTEXT ; echo $env:DOCKER_HOST #dans linux : echo $DOCKER_CONTEXT ; echo $DOCKER_HOST

$ export DOCKER_CONTEXT=definitely-not-working 

$ unset DOCKER_CONTEXT
```

# Réparer un conteneur cassé
**Scénario :** Votre conteneur ne fonctionne pas comme prévu. Voici le processus de débogage.

```bash
# Construire l'image
$ docker build -t app .

# Ouvrir le Dockerfile pour l'inspecter ou le modifier
$ notepad Dockerfile  # Windows
# vim Dockerfile      # Linux avec Vim
# code Dockerfile     # Avec VS Code

# Lancer le conteneur en mode interactif pour voir les erreurs
$ docker run -it app
# -it : mode interactif avec terminal (pour voir les erreurs en direct)

# Lancer avec un nom pour faciliter le débogage
$ docker run -it --name=app_container app

# Dans un autre terminal : monitorer les ressources
$ docker stats app_container
# Vérifier si le conteneur consomme trop de CPU ou mémoire

# Voir les processus en cours dans le conteneur
$ docker top app_container

# Retour au 1er terminal : corriger les problèmes

# Modifier le Dockerfile
$ notepad Dockerfile

# Modifier le script de l'application
$ code app.sh

# Reconstruire l'image après modifications
$  docker build -t app .
# Grâce au cache Docker, la reconstruction est très rapide!
# Docker réutilise les couches inchangées
```
**Astuce :** Utilisez `docker logs app_container` pour voir les logs même après que le conteneur se soit arrêté.
# Container Image Scanner 
**Sécurité :** Les outils d'analyse d'images de conteneur inspectent chaque couche d'images Docker et vous avertissent des couches connues pour être malveillantes, ainsi que des couches contenant des fichiers susceptibles d'être nuisibles (vulnérabilités, malware, secrets exposés, etc.).

## Quelques exemples de scanners populaires :
1. **Clair** - Scanner open-source développé par CoreOS
2. **Trivy** - Scanner de vulnérabilités simple et complet
3. **Dagda** - Outil d'analyse de vulnérabilités et de malware
4. **Docker Scout** - Scanner intégré à Docker Desktop
5. **Snyk** - Scanner commercial avec version gratuite

**Pourquoi scanner vos images ?**
- Détecter les vulnérabilités de sécurité connues (CVE)
- Identifier les dépendances obsolètes
- Trouver des secrets accidentellement exposés (mots de passe, clés API)
- Se conformer aux standards de sécurité

# Docker Compose
**Docker Compose** est un outil fourni par Docker qui facilite l'exécution et la connexion de plusieurs conteneurs sur une seule machine.

**Comment ça marche ?**
Avec Compose, vous utilisez un seul fichier appelé **docker-compose.yml** (Compose manifest) pour définir tous vos conteneurs et leurs relations les uns avec les autres. Démarrer ces conteneurs ensemble est aussi simple que d'exécuter `docker-compose up`.

**Avantages :**
- ✅ Définir toute votre stack applicative (base de données, API, frontend) dans un seul fichier
- ✅ Démarrer tous les services en une commande
- ✅ Les conteneurs peuvent communiquer facilement via des noms
- ✅ Intégrer des infrastructures d'applications entières localement

**Cas d'usage typique :**
Une application avec un backend (Node.js), une base de données (PostgreSQL), et un cache (Redis) peut être définie et lancée ensemble automatiquement.

**Commandes principales :**
```bash
docker-compose up      # Démarrer tous les services
docker-compose down    # Arrêter et supprimer tous les services
docker-compose logs    # Voir les logs de tous les services
docker-compose ps      # Lister les services en cours
``` 

# Passez au niveau supérieur avec Kubernetes

## Résumé : Docker vs Kubernetes

**Docker seul :** Limité en production (pas de multi-hôtes, pas d'auto-scaling, pas d'équilibrage de charge).

**Kubernetes :** Orchestrateur leader qui automatise tout (planification, réseau, scaling, sécurité).

**Alternatives :** Docker Swarm, AWS ECS, Azure AKS, Google GKE, Nomad.

## Les 4 atouts de Kubernetes

1. **Système distribué** - Multi-machines, du Raspberry Pi au cloud
2. **Auto-scaling** - S'adapte automatiquement à la charge
3. **Sécurité réseau** - Routage intelligent, chiffrement
4. **Écosystème riche** - Helm, Istio, Prometheus...
### Analogie simple :

**Docker** = Préparer un repas dans une boîte  
**Kubernetes** = Service de livraison qui clone et expédie des milliers de ces boîtes dans le monde entier



