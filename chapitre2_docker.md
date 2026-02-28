# Application utilisant Docker

## Introduction

Avec Docker, leurs développeurs pourront simplement télécharger une image sans avoir à installer manuellement tout ce dont ils ont besoin. Cela améliore considérablement le temps nécessaire aux développeurs pour démarrer un nouveau projet. Docker satisfait les développeurs en fournissant des outils pour automatiser les tâches manuelles et sujettes aux erreurs qui causent des casse-têtes. 

## Exemple : Big Star Collectibles

Big Star Collectibles utilise Python et Flask pour son site web. Flask est un framework d'applications web écrit en Python. Il est léger et vous permet de lancer rapidement une application web en utilisant un seul fichier Python.

**Avantages de Docker pour ce projet :**
- Installation automatisée de Python et Flask
- Environnement reproductible sur toutes les machines
- Démarrage rapide pour les nouveaux développeurs
- Isolation des dépendances du projet

# Recherche d’images dans Docker Hub

Docker Hub est le registre public où vous pouvez trouver des milliers d'images Docker prêtes à l'emploi. Voici comment rechercher des images efficacement :

```bash
# Recherche basique d'images Python
$ docker search python

# Filtrer uniquement les images officielles (maintenues par Docker et les éditeurs officiels)
$ docker search --filter is-official=true python

# Filtrer les images ayant au moins 100 étoiles (popularité)
$ docker search --filter stars=100 python

# Filtrer les images automatisées (construites automatiquement depuis le code source)
$ docker search --filter is-automated=true python

# Combiner plusieurs filtres pour une recherche précise
$ docker search --filter is-official=true --filter stars=100 --filter is-automated=true python

# Limiter le nombre de résultats affichés (ici 4 résultats)
$ docker search --limit 4 python

# Formater l'affichage : nom et description complète
$ docker search --format "{{.Name}}: {{.Description}}" --no-trun python

# Afficher les résultats sous forme de tableau avec plusieurs colonnes
$ docker search --format "table {{.Name}}\t{{.IsOfficial}}\t{{.Description}}\t{{.StarCount}}\t" python
```
# Travailler avec des images personnalisées

Cette section couvre la gestion des images Docker : téléchargement, listage et filtrage.

```bash
# Télécharger l'image Python avec le tag "latest" par défaut
$ docker image pull python

# Télécharger une version spécifique de Python (ici la version 3.12-rc avec Debian Bookworm)
$ docker image pull python:3.12-rc-bookworm

# Lister toutes les images disponibles localement
$ docker image ls 

# Afficher toutes les images, y compris les couches intermédiaires
$ docker iamge ls --all

# Afficher l'ID complet des images (sans troncature)
$ docker image ls --no-trunc

# Afficher uniquement les IDs des images (utile pour les scripts)
$ docker image ls --quiet

# Afficher le digest (hash SHA256) des images pour vérifier leur intégrité
$ docker image ls --digests

# Filtrer les images dont le nom commence par "python"
$ docker image ls --filter reference=python*

# Filtrer les images créées avant une image spécifique
$ docker image ls --filter before=python*

# Afficher les images "dangling" (sans tag, souvent après une reconstruction)
$ docker image ls --filter dangling=true

# Filtrer les images par label personnalisé
$ docker image ls --filter label=python*

# Formater l'affichage des résultats de recherche sous forme de tableau personnalisé
$ docker search --format "table {{.ID}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedSince}}\t{{.CreatedAt}}\t{{.Digest}}" python 
```

# Taguer et étiquetage des images

Les tags permettent de versionner et identifier vos images Docker de manière organisée.

```bash
# Syntaxe générale pour construire une image avec un tag
$ docker build -t SOURCE-IMAGE[:TAG] .

# Construire sans cache avec plusieurs tags simultanément (v1 et latest)
# --no-cache : force la reconstruction complète sans utiliser les couches en cache
$ docker build --no-cache -t big-star-collectibles:v1 -t big-star-collectibles .

# Syntaxe générale pour créer un nouveau tag pour une image existante (alias)
$ docker tag SOURCE-IMAGE[:TAG] TARGET-IMAGE[:TAG]

# Exemple : ajouter un tag descriptif à une image existante
$ docker tag big-star-collectibles:v1 big-star-collectibles:v1-python

# Construire une nouvelle version de l'image (v2) sans utiliser le cache
$ docker build --no-cache -t big-star-collectibles:v2 .
```

## Utilisation des étiquettes (labels) dans les images
Une autre facon de garder vos imager organisees consiste a utiliser l'instruction d'etiquette dans votre fichier Dockerfile.  Cela permet d’ajouter des métadonnées à une image à l’aide de paires clé-valeur. Vous pouvez inclure plusieurs commandes d’étiquette et certaines des plus courantes sont fournisseur, version et description. Pour inclure des espaces dans une valeur d’étiquette, utilisez des guillemets et des barres obliquées comme dans l’analyse en ligne de commande.  

### Option 1 : Une étiquette par ligne (recommandé pour la lisibilité)

```dockerfile
• Add labels to the image
  LABEL Portactis Upcion 3
LABEL "com. example.vendor"="Big Star collectibles"
LABEL versions"1.0"
LABEL description="the Big Star Collectibles website \ using the python base images."
```

Une autre syntaxe que vous pouvez utiliser est une commande d'étiquette, puis d'afficher plusieurs étiquettes. J'aime ajouter une barre oblique inversée après chaque étiquette pour qu'elles soient listées sur des lignes séparées et plus faciles à lire. Lorsque vous reconstruisez votre image, l'étiquette sera ajoutée aux métadonnées. 

### Option 2 : Plusieurs étiquettes sur une seule ligne LABEL (syntaxe compacte)

```dockerfile
• LaBeL Portatting Option 2
LABEL "com.example.vendor"="Big Star Collectibles" version="1.0" description="The Big Star Collectibles website \ using the Python base image."
```

# Travailler avec un dépôt d’images privé

Docker Hub est une plateforme puissante qui permet aux développeurs de créer et de gérer facilement leur propre dépôt d'images privé.

```bash
# Étape 1 : Taguer l'image locale avec le nom du dépôt privé
# Format : nom-utilisateur/nom-depot:tag
$ docker tag big-star-collectibles:v2 sbenhoff/big-star-collectibles-repo:big-star-collectibles

# Étape 2 : Pousser (uploader) l'image vers Docker Hub
# Nécessite d'être authentifié avec "docker login" au préalable
$ docker push sbenhoff/big-star-collectibles-repo:big-star-collectibles

# Étape 3 : Télécharger l'image depuis le dépôt privé
# Peut être exécuté sur n'importe quelle machine après authentification
$ docker pull sbenhoff/big-star-collectibles-repo:big-star-collectibles
```
# Inspection des images

L'inspection permet de voir toutes les métadonnées et la configuration d'une image Docker.

```bash
# Inspecter une image par son ID ou son nom
# Affiche toutes les informations : configuration, couches, labels, taille, etc.
$ docker image inspect ID
```

vous pouvez voir l'ID photo complet ainsi que d'autres informations utiles...

```bash
# Extraire uniquement les labels de l'image au format JSON
# Utilise le template Go pour formater la sortie
$ docker image inspect --format='{{ json .Config.Labels }}' big-star-collectibles:v2
```
# Suppression des images

Commandes pour supprimer des images Docker et libérer de l'espace disque.

```bash
# Supprimer une image par son tag en forçant la suppression
# -f : force la suppression même si des conteneurs utilisent cette image
$ docker rmi -f big-star-collectibles:v2 

# Supprimer une image par son ID (les 12 premiers caractères suffisent)
$ docker rmi ID

# Supprimer l'image avec tous ses tags en forçant
$ docker rmi -f ID # suuprimer l'images avec ses tags

# Supprimer une image spécifique en utilisant son digest (identifiant unique)
$ docker rmi -f NomImage@digest

```
# Conteneurs de liste

Différentes façons de lister et filtrer les conteneurs Docker.

```bash
# Afficher le dernier conteneur créé (peu importe son état : running, stopped, etc.)
$ docker ps -n 1 # dernier conteneur (compris tous les etats)

# Afficher la taille totale des fichiers de tous les conteneurs en cours
# -s : montre l'espace disque utilisé par chaque conteneur
$ docker ps -s # affiche la taille totale du fichier de tous les contneurs

# Afficher le dernier conteneur créé (équivalent de -n 1)
$ docker ps -l # indque le dernier conteneur cree

# Afficher les conteneurs sans tronquer les informations (IDs et noms complets)
$ docker ps --no-trunc

# Filtrer les conteneurs par label personnalisé
$ doocker ps --filter label="com. example.vendor"="Big Star collectibles"

# Afficher tous les conteneurs (-a) qui se sont terminés avec le code de sortie 137
# 137 = conteneur tué par un signal SIGKILL
$ docker ps -a --filter 'exited=137'
$ 
```
# Inspection des contenants

L'inspection des conteneurs permet de voir leur configuration complète et leur état.

```bash
# Inspecter un conteneur par son nom ou son ID
# Retourne toutes les métadonnées : configuration, réseau, volumes, état, etc.
$ docker  inspect Name

# Extraire uniquement l'image utilisée par le conteneur
$ docker inspect --format='{{ .Config.Image }}' elastic_perlman

# Afficher l'ID complet du conteneur
$ docker inspect --format='{{ .Id}}' elastic_perlman

# Afficher le chemin du fichier de logs du conteneur
$ docker inspect --format='{{ .LogPath}}' elastic_perlman

# Extraire toute la configuration au format JSON
# Utile pour analyser ou sauvegarder la configuration
$ docker inspect --format='{{json .Config }}' elastic_perlman

``` 
# Révision des fichiers journaux du conteneur

Les logs permettent de déboguer et surveiller l'activité des conteneurs.

```bash
# Syntaxe générale pour afficher les logs d'un conteneur
$ docker logs [OPTIONS] CONTAINER

# Afficher les 1000 dernières lignes de logs et suivre en temps réel
# --tail 1000 : affiche les 1000 dernières lignes
# -f : suit les logs en continu (comme tail -f sous Linux)
$ docker logs --tall 1000 -f elastic_perlman
```
# Travailler avec des volumes et des montures

Les volumes Docker permettent de persister les données en dehors du cycle de vie des conteneurs.

```bash
# Créer un nouveau volume nommé
# Les volumes sont stockés par Docker et persistent après la suppression des conteneurs
$ docker volume create big-star-collectibles-vol

# Lister tous les volumes disponibles sur la machine
$ docker volume ls

# Inspecter un volume pour voir ses détails
# Affiche le point de montage, le driver utilisé, les options, etc.
$ docker volume inspect big-star-collectibles-vol

```