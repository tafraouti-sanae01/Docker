# Implémenter Docker Compose

## Qu'est-ce que Docker Compose ?

Docker Compose est un outil qui **simplifie l'utilisation de Docker** en développement local.

**Avantages** :
- Rend le développement plus facile et plus rapide
- Permet aux développeurs d'être plus productifs
- Gère plusieurs conteneurs Docker à la fois avec un seul fichier

Dans ce chapitre, vous apprendrez :
- ✓ Comment Docker Compose fonctionne
- ✓ Toutes les fonctionnalités de base avec des exemples réels
- ✓ Comment rendre vos systèmes plus flexibles et maintenables

## Comment fonctionne Docker Compose ?

Docker Compose est un outil livré avec Docker. Il utilise un **fichier de configuration YAML** pour gérer plusieurs conteneurs Docker ensemble.

---

# Où utiliser Docker Compose ?

## Les cas d'utilisation idéals

Docker Compose est **idéal pour** :
- **Développement local** : tester votre application sur votre ordinateur
- **Serveurs intermédiaires** : avant de déployer en production
- **Tests d'intégration continue** : vérifier que tous les services fonctionnent ensemble

## Cas d'utilisation INADÉQUATS

Docker Compose **N'EST PAS adapté** pour :
- **Systèmes distribués** : fonctionne sur un seul serveur seulement
- **Production avec beaucoup de trafic** : car il n'a pas de mise à l'échelle automatique

### Pourquoi pas en production ?

En production (quand le site a beaucoup de visiteurs), Docker Compose a des limitations graves :

**Limitation 1 : Pas de mise à l'échelle automatique**
- Vous ne pouvez pas augmenter automatiquement les conteneurs
- Vous devez le faire manuellement

**Limitation 2 : Pas de mise à l'échelle sélective**
- Vous ne pouvez multiplier que **tous les services en même temps**
- Impossible de multiplier seulement 1 service

### Exemple réel : KinetEco

KinetEco est une entreprise d'énergie propre avec deux applications :
1. **Vitrine en ligne** : vend des panneaux solaires
2. **Planificateur** : organise les installations

**Scenario problématique** :
- Une vente promotionnelle arrive !
- La vitrine reçoit **10x plus de visiteurs**
- Le planificateur continue à fonctionner normalement

**Avec Docker Compose** : vous devez multiplier les DEUX services (gaspillage !)

**Solution** : Utilisez **Kubernetes** ou **Docker Swarm** pour la production (ils permettent la mise à l'échelle sélective)

---

# YAML : Le format de configuration

**YAML** signifie : **Y**et **A**nother **M**arkup **L**anguage

C'est un **format de configuration simple et lisible** utilisé par Docker Compose.

---

# Écriture d'une configuration Docker Compose

## Étape 1 : Créer le fichier

Pour commencer à utiliser Docker Compose, créez un fichier `docker-compose.yaml` dans le répertoire de votre application.

**Règles importantes** :
- ✓ Le fichier **DOIT** s'appeler `docker-compose.yaml`
- ✓ Le format **DOIT** être YAML (similaire à JSON, mais plus lisible)
- ✓ Ce fichier **décrit TOUS les conteneurs** que vous voulez lancer

## Étape 2 : Spécifier la version

```yaml
version: '3.8'  # Version de Docker Compose à utiliser
```

**Pourquoi** ? Chaque version de Compose a des fonctionnalités différentes.

## Étape 3 : Définir les services

Listez tous les conteneurs Docker à lancer :

```yaml
services:
  # Premier service/conteneur
  service1:
    ...
  
  # Deuxième service/conteneur
  service2:
    ...
```

**Important** : L'indentation en YAML est **OBLIGATOIRE** pour montrer la hiérarchie !

## Exemple complet : KinetEco 

L'application KinetEco a besoin de 3 services :
1. **storefront** : vitrine en ligne (vend les produits)
2. **scheduler** : planificateur (organise les installations)
3. **database** : base de données MySQL (stocke les données)

### Structure du projet réel

```
kineteco/
  docker-compose.yaml    ← Le fichier de configuration
  .env                   ← Variables d'environnement
  storefront/
    Dockerfile
    index.html
    kineteco_consumer_files/  ← Fichiers web
  scheduler/
    Dockerfile
    index.html
    kineteco_scheduler_files/ ← Fichiers web
  mysql/
    env_vars            ← Variables pour MySQL
    db.sql              ← Dump de la base de données
```

### Le vrai fichier docker-compose.yaml

```yaml
version: "3.9"

services:
  # Service 1 : Vitrine en ligne (nginx)
  storefront:
    build: storefront/.  # Construit l'image à partir du Dockerfile
    ports:
      - "80:80"      # Port HTTP
      - "443:443"    # Port HTTPS (monitoring)
    depends_on:
      - database     # Dépend de la base de données
  
  # Service 2 : Planificateur (nginx)
  scheduler:
    build: scheduler/.  # Construit l'image à partir du Dockerfile
    ports:
      - "81:80"      # Port HTTP (81 pour éviter conflit avec storefront)
    depends_on:
      - database
  
  # Service 3 : Base de données MySQL
  database:
    image: "mysql:${TAG:-latest}"  # Utilise la variable TAG du .env
    env_file:
      - ./mysql/env_vars  # Charge les variables MySQL
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d:ro  # Charge le script SQL
      - kineteco:/var/lib/mysql                 # Sauvegarde les données

# Volume nommé pour la persistance
volumes:
  kineteco:  # Garde les données MySQL entre les redémarrages
```

### Fichier .env

```bash
# .env
TAG=latest  # Version de MySQL à utiliser
```

**Utilisation** : `image: mysql:${TAG:-latest}` — si TAG est vide, utilise "latest"

### Fichier mysql/env_vars

```bash
# mysql/env_vars
MYSQL_ROOT_PASSWORD=]p0.3617SR
MYSQL_DATABASE=ldcsites_wp
MYSQL_USER=ldcsites_wp
MYSQL_PASSWORD=]p0.3617S
```

Ceci définit :
- ✓ Le mot de passe admin MySQL
- ✓ Le nom de la base de données
- ✓ L'utilisateur de la base
- ✓ Le mot de passe de l'utilisateur

### Dockerfiles réels

**storefront/Dockerfile** :
```dockerfile
ARG REGION=us-east-1         # Argument de construction
FROM nginx:alpine            # Image de base
COPY ./index.html /usr/share/nginx/html/
COPY ./kineteco_consumer_files /usr/share/nginx/html/kineteco_consumer_files
```

**scheduler/Dockerfile** :
```dockerfile
FROM nginx:alpine
COPY ./index.html /usr/share/nginx/html/
COPY ./kineteco_scheduler_files /usr/share/nginx/html/schedule_files
```

Les DEUX utilisent **nginx:alpine** (serveur web léger)

### Comment lancer le projet KinetEco réel

```bash
# 1. Aller dans le dossier du projet
$ cd kineteco/

# 2. Lancer tous les services
$ docker-compose up

# Résultat :
# - Construit l'image storefront
# - Construit l'image scheduler
# - Télécharge l'image mysql:latest (ou la version du TAG)
# - Lance tous les 3 conteneurs

# 3. Accéder aux applications
# Vitrine : http://localhost:80
# Planificateur : http://localhost:81
# MySQL : localhost:3306 (pour les outils de gestion)
```

### Détails : qu'est-ce qui se passe réellement ?

**Étape 1 : Construit les images (build)**
```bash
$ docker build -t kineteco_storefront storefront/.
$ docker build -t kineteco_scheduler scheduler/.
```

**Étape 2 : Crée les conteneurs**
```bash
$ docker create --name kineteco_storefront kineteco_storefront
$ docker create --name kineteco_scheduler kineteco_scheduler
$ docker create --name kineteco_database mysql:latest
```

**Étape 3 : Démarre les conteneurs**
```bash
$ docker start kineteco_database
$ docker start kineteco_storefront
$ docker start kineteco_scheduler
```

Docker Compose fait TOUT cela avec une seule commande : `docker-compose up` ! 🎉

## Conseil : Nommer les services clairement

Les noms de service :
- Peuvent être n'importe quoi (`storefront`, `database`, `Alice`, `Bob`, etc.)
- **DOIVENT être clairs et significatifs**
- Aident les autres développeurs à comprendre le rôle

---

# Commandes Docker Compose de base

## Les 4 commandes principales

```bash
# 1. Démarrer tout : built l'image + crée + lance les conteneurs
$ docker-compose up

# 2. Arrêter tout : éteint tout et supprime les conteneurs
$ docker-compose down

# 3. Pause les conteneurs : sans les supprimer (économise CPU/RAM)
$ docker-compose stop

# 4. Redémarrer : arrête puis relance (utile pour recharger configs)
$ docker-compose restart
```

## Tableau comparatif

| Commande | Action | Conteneurs | Volumes |
|----------|--------|-----------|---------|
| `up` | Lance tout | Crée | Gardes |
| `down` | Éteint tout | Supprime | Gardés |
| `stop` | Pause | Gardés | Gardés |
| `restart` | Relance | Redémarres | Gardés |

---

# Variables d'environnement dans les conteneurs

## Qu'est-ce qu'une variable d'environnement ?

Une **variable d'environnement** est une valeur que vous passez à un conteneur at démarrage. Le conteneur l'utilise comme configuration.

## Cas d'utilisation courants

### Cas 1 : Configuration du logging (journalisation)

```bash
# Plus de détails en logs
LOG_LEVEL=debug

# Moins de détails en logs
LOG_LEVEL=info
```

### Cas 2 : Configuration selon l'environnement

```python
# En test : désactiver les paiements
if (runtime_env == "test"):
    disable_payments()  # Pas de vrai paiement en test

# En production : activer les paiements
if (runtime_env == "production"):
    enable_payments()  # Paiements réels
```

## 2 façons de passer les variables

### Méthode 1 : Directement dans le docker-compose.yaml

```yaml
services:
  database:
    environment:
      MYSQL_ROOT_PASSWORD: password123
      MYSQL_DATABASE: my_db
      MYSQL_USER: john
```

### Méthode 2 : Depuis un fichier (récommandée pour les secrets)

```yaml
services:
  database:
    env_file:
      - ./mysql/env_vars  # Charge les variables du fichier
```

**Pourquoi Méthode 2 ?**
- ✓ Les mots de passe ne sont pas en clair dans le YAML
- ✓ Facile de changer les variables sans éditer le docker-compose.yaml
- ✓ Plus sécurisé pour les données sensibles

## Exemple réel : KinetEco

### Option 1 : Directement (NON RECOMMANDÉE pour les mots de passe)

```yaml
services:
  database:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: ]p0.3617SR     # ❌ Pas bon : en clair !
      MYSQL_DATABASE: ldcsites_wp
      MYSQL_USER: ldcsites_wp
      MYSQL_PASSWORD: ]p0.3617S
```

### Option 2 : Depuis un fichier (RECOMMANDÉE !)

**docker-compose.yaml** :
```yaml
services:
  database:
    image: mysql:latest
    env_file:
      - ./mysql/env_vars  # ✓ Bon : fichier séparé
```

**mysql/env_vars** :
```bash
MYSQL_ROOT_PASSWORD=]p0.3617SR
MYSQL_DATABASE=ldcsites_wp
MYSQL_USER=ldcsites_wp
MYSQL_PASSWORD=]p0.3617S
```

**Avantages** :
- ✓ Le fichier env_vars peut être dans .gitignore (pas de secrets sur GitHub)
- ✓ Structure plus propre
- ✓ Les secrets restent privés

---

# Volumes de montage

## Qu'est-ce qu'un volume ?

Un **volume** est un **dossier partagé** entre votre ordinateur et le conteneur.

**But** : Sauvegarder les données hors du conteneur (pour qu'elles ne se perdent pas)

## Modes d'accès

| Mode | Signification | Description | Sécurité |
|------|---------------|-------------|----------|
| `rw` | Read-Write | Le conteneur peut lire **ET** écrire | ⚠️ Moins sûr |
| `ro` | Read-Only | Le conteneur peut **seulement** lire | ✓ Plus sûr |

**Exemple** :
```yaml
volumes:
  - ./my-data:/app/data:rw  # Lecture + écriture (défault)
  - ./config:/app/config:ro # Lecture seulement
```

## 2 types de volumes : chemins vs nommés

### Volumes avec chemins (host-bound)

Le conteneur partage un **dossier de votre ordinateur** :

```yaml
volumes:
  - ./mysql:/docker-entrypoint-initdb.d:ro  # Chemin local
  - /var/log:/app/logs:rw                   # Chemin absolu
```

**Cas pratique** : Charger un script SQL au démarrage de MySQL

```yaml
database:
  image: mysql:latest
  volumes:
    - ./mysql:/docker-entrypoint-initdb.d:ro  # ← SQL scripts ici
```

Quand MySQL démarre, il exécute tous les fichiers `.sql` du dossier `/docker-entrypoint-initdb.d`.

### Volumes nommés (data volumes)

Le conteneur stocke les données dans un **volume géré par Docker** :

```yaml
volumes:
  - kineteco:/var/lib/mysql  # Volume nommé
```

**Cas pratique** : Persister les données de la base de données

---

# Volumes nommés (les volumes qui restent)

## Pourquoi utiliser des volumes nommés ?

Les volumes nommés permettent à Compose de **gérer les données automatiquement** et d'éviter du gaspillage d'espace disque.

## Problème sans volume nommé

Quand vous montez un volume **sans nom** :
```yaml
storefront:
  volumes:
    - /var/lib/mysql  # PAS de nom !
```

**Problèmes** :
- ❌ Docker crée un volume avec un nom aléatoire à chaque `docker-compose up`
- ❌ Vous accumulez des volumes **fantasmes** inutiles
- ❌ Après quelques mois : **40 GB de données perdues** ! (cas réel)
- ❌ L'ordinateur ralentit énormément

**Symptôme** : "Pourquoi mon ordinateur est à la traîne ?"

## Solution : utiliser un volume nommé

```yaml
services:
  database:
    image: mysql:latest
    volumes:
      - kinetico_data:/var/lib/mysql  # Volume nommé "kinetico_data"

# IMPORTANT : définir le volume au même niveau que services
volumes:
  kinetico_data:  # Définition du volume nommé
```

**Avantages** :
- ✓ Pas de volumes fantasmes
- ✓ Compose gère automatiquement les données
- ✓ Pas de perte de données au redémarrage
- ✓ Noms clairs et lisibles

## Nettoyer les volumes

```bash
# Arrête ET supprime les volumes nommés
$ docker-compose down --volumes
```

Cela libère de l'espace disque (recommandé !).

## Exemple réel : comment fonctionne le volume kineteco

### Le vrai cas KinetEco

Le fichier docker-compose.yaml a :

```yaml
database:
  image: mysql:latest
  volumes:
    - ./mysql:/docker-entrypoint-initdb.d:ro  # Charger le SQL
    - kineteco:/var/lib/mysql                  # Sauvegarder les données

volumes:
  kineteco:  # Définir le volume nommé
```

### Ce qu'il se passe au démarrage

**Commande** :
```bash
$ docker-compose up
```

**Résultat** :
1. ✓ Crée un volume nommé `kineteco` (s'il n'existe pas)
2. ✓ Démarre le conteneur MySQL
3. ✓ Exécute les scripts SQL du dossier `./mysql` (initdb)
4. ✓ MySQL stocke les données dans le volume `kineteco`

### Arrêt et redémarrage

**Commande** :
```bash
$ docker-compose down
```

**Résultat** :
- ✓ Arrête et supprime le conteneur MySQL
- ❌ Garde le volume `kineteco` (les données restent !)

**Commande** :
```bash
$ docker-compose up
```

**Résultat** :
- ✓ Crée un nouveau conteneur MySQL
- ✓ Récupère les données du volume `kineteco`
- ✓ Les données SONT RESTITUÉES !

### Test : vérifier les données

```bash
# Avant d'arrêter : créer une table
$ docker-compose exec database mysql -uroot -p ldcsites_wp
mysql> CREATE TABLE test (id INT PRIMARY KEY);
mysql> exit;

# Arrêter les conteneurs
$ docker-compose down

# Les données restent dans le volume kineteco
$ docker volume ls
# VOLUME NAME
# kineteco  ← Toujours là !

# Redémarrer
$ docker-compose up

# Vérifier les données
$ docker-compose exec database mysql -uroot -p ldcsites_wp
mysql> SHOW TABLES;
# +-----------------------+
# | Tables_in_ldcsites_wp |
# +-----------------------+
# | test                  |  ← TABLE EST LÀ !
# +-----------------------+
```

**Conclusion** : Les volumes nommés = données **persistantes** ! 🎉

---

## Syntaxes disponibles

### Syntaxe courte (simple)

```yaml
volumes:
  - kinetico_data:/var/lib/mysql
```

### Syntaxe longue (détaillée, Compose 3.2+)

```yaml
volumes:
  - type: volume
    source: kinetico_data
    target: /var/lib/mysql
```

La syntaxe courte suffit generally.

---

# Exposition des ports

## Qu'est-ce qu'un port ?

Un **port** est une "porte" numérique pour communiquer avec un conteneur.

## Le problème : isolement du conteneur

Les conteneurs Docker sont **isolés** par défaut :
- ❌ Vous ne pouvez pas y accéder de l'extérieur
- ❌ Les données RESTENT dans le conteneur

**Solution** : **Exposer explicitement** les ports pour communiquer.

## La solution : Port mapping (mapping de ports)

Un **port mapping** relie :
- Un port sur **votre ordinateur** (hôte)
- À un port à l'intérieur du **conteneur**

**Visualisation** :
```
Votre ordinateur (localhost)
    |
    Port 8080 
    |
    ↓ (mapping)
    |
Conteneur
    |
    Port 80 (interne)
```

**Accès** : `http://localhost:8080`

## Numéros de port importants

Il existe **plus de 65,000 ports** TCP disponibles.

**Ports courants** :
- **80** : Web (HTTP)
- **443** : Web sécurisé (HTTPS)
- **3306** : MySQL (base de données)
- **5432** : PostgreSQL (base de données)

**Règle essentielle** : ❌ Vous **NE POUVEZ PAS** utiliser 2 fois le même port sur votre ordinateur !

## Avantage de Docker Compose

Docker Compose **documente tous les ports** dans le fichier de configuration :
- Facile de voir quels ports sont exposés
- Pas besoin de mémoriser les mappages

## Exemple : KinetEco

Le problème réel de KinetEco : 2 services web veulent le port 80

**Le vrai fichier docker-compose.yaml de kineteco** :

```yaml
services:
  # Service 1 : Vitrine en ligne
  storefront:
    build: storefront/.
    ports:
      - "80:80"   # Port HTTP
      - "443:443" # Port HTTPS/monitoring
  
  # Service 2 : Planificateur
  scheduler:
    build: scheduler/.
    ports:
      - "81:80"   # Port 81 de l'hôte → Port 80 du conteneur
```

**Visualisation** :

```
Votre ordinateur (localhost)
  |
  ├─ Port 80   → Conteneur storefront:80 (vitrine web)
  ├─ Port 81   → Conteneur scheduler:80  (planificateur web)
  └─ Port 443  → Conteneur storefront:443 (monitoring)
```

**Accès depuis le navigateur** :
- Vitrine : `http://localhost:80` ou `http://localhost`
- Planificateur : `http://localhost:81`
- Monitoring : `https://localhost:443` (port HTTPS)

## Syntaxe du mapping

```
PORT_HÔTE:PORT_CONTENEUR
```

**Exemples** :
- `80:80` = Port 80 de l'hôte vers port 80 du conteneur
- `81:80` = Port 81 de l'hôte vers port 80 du conteneur
- `8080:3000` = Port 8080 de l'hôte vers port 3000 du conteneur

## Conseil : utilisez les guillemets

```yaml
ports:
  - "80:80"   # ✓ Avec guillemets (recommandé)
```

Les guillemets évitent les problèmes avec les numéros spéciaux.

## Cas spécial : plusieurs ports par service

Un service peut avoir besoin de **plusieurs ports** pour différentes fonctions :

```yaml
storefront:
  ports:
    - "80:80"     # Port web principal
    - "443:443"   # Port de monitoring (collecte les métriques)
```

Cela permet à un agent de monitoring sur l'hôte de surveiller le conteneur.

---

# Dépendances entre services

## Le problème : ordre de démarrage

Beaucoup d'applications ont des **dépendances de service** :
- Un service **NE PEUT PAS** fonctionner sans un autre
- Exemple KinetEco : la vitrine **a besoin** de la base de données

**Problème** :
- Si la base de données n'est pas prête, la vitrine plante
- Les services doivent démarrer dans le bon ordre

## La solution : `depends_on`

Utilisez `depends_on` pour spécifier les dépendances :

```yaml
services:
  # La vitrine dépend de la base de données
  storefront:
    depends_on:
      - database  # Démarrer 'database' en premier
  
  database:
    image: mysql:latest
```

### Exemple réel : KinetEco

Voici le vrai fichier docker-compose.yaml de kineteco :

```yaml
version: "3.9"

services:
  storefront:
    build: storefront/.
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - database  # ← Storefront attend database
  
  scheduler:
    build: scheduler/.
    ports:
      - "81:80"
    depends_on:
      - database  # ← Scheduler attend database
  
  database:
    image: "mysql:${TAG:-latest}"
    env_file:
      - ./mysql/env_vars
    volumes:
      - ./mysql:/docker-entrypoint-initdb.d:ro
      - kineteco:/var/lib/mysql
    # ← Database n'a pas de dépendances

volumes:
  kineteco:
```

**Logique** :
- `storefront` et `scheduler` dépendent de `database`
- Docker démarre `database` en premier
- Puis les 2 autres services simultanément

## Comportement automatique

### Au démarrage (`docker-compose up`)
1. ✓ Démarrer `database` en premier
2. ✓ Puis démarrer `storefront` (après database)

### À l'arrêt (`docker-compose down`)
1. ✓ Arrêter `storefront` en premier (la dépendante)
2. ✓ Puis arrêter `database`

**Avantage** : Plus besoin de démarrer manuellement dans le bon ordre !

## Chaînes de dépendances

Le système gère les **cha_nes** :

```yaml
services:
  app:
    depends_on:
      - cache
  
  cache:
    depends_on:
      - database
  
  database:
    image: mysql:latest
```

**Ordre** : database → cache → app

## Démarrer un seul service

```bash
# Démarrer 'storefront' ET ses dépendances automatiquement
$ docker-compose up storefront
```

Cela démarre aussi `database` car `storefront` en dépend.

## ⚠️ Limitation importante

`depends_on` garantit **SEULEMENT** que le service :
- ✓ A **démarré**

`depends_on` **NE garantit PAS** que le service :
- ❌ Est **prêt** à recevoir les requêtes
- ❌ Est en **bonne santé**
- ❌ A fini s'**initialisation**

**Problème concret** : La base de données peut être **en train de démarrer** quand la vitrine commence et essaie de se connecter → ERREUR !

## Pourquoi cette limitation ?

Dans les **systèmes distribués** :
- Le vrai monde est compliqué
- Aucun service n'a **100% de disponibilité**
- Les services peuvent **s'arrêter à TOUT MOMENT**
- Votre application **DOIT gérer les erreurs**

**Exemple** : Si la base de données tombe après le démarrage de la vitrine ? La vitrine doit afficher une erreur utile, pas un plantage !

## Solution (pour les cas spéciaux)

Si vous **absolument** devez attendre qu'une dépendance soit saine :
- Utilisez des **outils tiers** (comme `wait-for-it.sh`)
- Cela encapsule le démarrage pour vérifier la santé

```bash
$ ./wait-for-it.sh database:3306 -- ./start.sh
```

**Mais** : ❌ Ce n'est **PAS recommandé** en production car cela crée un **couplage étroit** entre les services.

---

# Profils de service

## Qu'est-ce qu'un profil de service ?

Un profil est une **catégorie** pour grouper des services.

**Utilité** : Lancer **seulement certains services** au lieu de tous.

## Cas d'utilisation : grandes organisations

Imaginez une grande entreprise avec **plusieurs équipes** :

### Équipe 1 : Vitrine
- Travaille sur : vitrine + ses services
- Ne veut **PAS** du planificateur (consomme des ressources)

### Équipe 2 : Planificateur
- Travaille sur : planificateur + ses services
- Ne veut **PAS** de la vitrine

### Mais un seul fichier !

Les 2 équipes veulent :
- 1 seul fichier `docker-compose.yaml`
- Raison 1 : Pour les tests d'intégration (tout ensemble)
- Raison 2 : Pour comprendre le système global

**Solution** : Profils !

## Comment créer des profils

Affectez chaque service à un profil (catégorie) :

```yaml
services:
  # Service de la vitrine
  storefront:
    profiles:
      - storefront-services  # Va dans ce profil
  
  # Service du planificateur
  scheduler:
    profiles:
      - scheduler-services   # Va dans ce profil
  
  # Service partagé par les 2 équipes
  database:
    image: mysql:latest
    # PAS de 'profiles' → inclus DANS TOUS les profils
```

## Services partagés

La base de données est utilisée par **les 2 services** :

```yaml
database:
  image: mysql:latest
  # Sans profil = lancé automatiquement avec TOUS les profils
```

**Alternative** : Ajouter la base aux 2 profils :
```yaml
database:
  profiles:
    - storefront-services
    - scheduler-services
```

## Comportement par défaut

Quand vous lancez `docker-compose up` **sans** profil :

```bash
$ docker-compose up
```

**Résultat** : Seulement les **services SANS profil** s'exécutent
- ✓ Database (pas de profil)
- ❌ Storefront (profil non activé)
- ❌ Scheduler (profil non activé)

## Activer un profil spécifique

```bash
# Lance la vitrine + la base de données (profil par défaut)
$ docker-compose --profile storefront-services up

# Lance le planificateur + la base de données
$ docker-compose --profile scheduler-services up

# Lance les 2 + la base (2 profils)
$ docker-compose --profile storefront-services --profile scheduler-services up
```

## Profil avec toutes les commandes

Le `--profile` fonctionne avec **toutes les commandes** :

```bash
$ docker-compose --profile storefront-services down
$ docker-compose --profile storefront-services restart
$ docker-compose --profile storefront-services logs
```

## Résumé des profils

Avec les profils, vous pouvez :
- ✓ Lancer **seulement** la vitrine (économise CPU/RAM)
- ✓ Lancer **seulement** le planificateur
- ✓ Lancer **les 2 ensemble**
- ✓ Utiliser **1 seul fichier** de configuration

---

# Cas pratique : Lancer le projet réel KinetEco

## Préparation

Assurez-vous d'avoir les bons outils :
- ✓ Docker installé
- ✓ Docker Compose installé
- ✓ Le dossier `kineteco/` avec tous les fichiers

## Étape 1 : Aller dans le dossier du projet

```bash
$ cd kineteco/
$ ls -la

# Résultat :
# docker-compose.yaml
# .env
# storefront/
# scheduler/
# mysql/
```

## Étape 2 : Vérifier le fichier .env

```bash
$ cat .env

# Résultat :
# TAG=latest
```

Cela définit la version de MySQL à utiliser.

## Étape 3 : Vérifier les variables MySQL

```bash
$ cat mysql/env_vars

# Résultat :
# MYSQL_ROOT_PASSWORD=]p0.3617SR
# MYSQL_DATABASE=ldcsites_wp
# MYSQL_USER=ldcsites_wp
# MYSQL_PASSWORD=]p0.3617S
```

## Étape 4 : Lancer les services

```bash
# Lancer tous les services (storefront + scheduler + database)
$ docker-compose up

# Résultat :
# Creating kineteco_database_1  ... done
# Creating kineteco_storefront_1 ... done
# Creating kineteco_scheduler_1  ... done
# Attaching to kineteco_database_1, kineteco_storefront_1, kineteco_scheduler_1
# ...
# kineteco_database_1 exited with code 0 (MySQL démarré)
# kineteco_storefront_1 | nginx: (master process) (...)
# kineteco_scheduler_1 | nginx: (master process) (...)
```

**Résultat** : Les 3 services tournent ! 🎉

## Étape 5 : Accéder aux applications

Ouvrez votre navigateur :

1. **Vitrine** (storefront) : http://localhost:80
2. **Planificateur** (scheduler) : http://localhost:81
3. **Monitoring** : https://localhost:443

## Étape 6 : Arrêter les services

```bash
# Arrêter en gardant les données
$ docker-compose stop

# Résultat :
# Stopping kineteco_storefront_1 ... done
# Stopping kineteco_scheduler_1  ... done
# Stopping kineteco_database_1   ... done
```

## Étape 7 : Redémarrer

```bash
# Les données MySQL sont restituées !
$ docker-compose start

# Vérifier que tout redémarre
$ docker-compose ps

# Résultat :
# NAME                      STATE
# kineteco_database_1       running
# kineteco_storefront_1     running
# kineteco_scheduler_1      running
```

## Étape 8 : Nettoyer complètement

```bash
# Arrêter ET supprimer les conteneurs
$ docker-compose down

# Garder les données MySQL 🎉
$ docker volume ls
# VOLUME NAME
# kineteco  ← Les données restent !
```

## Étape 9 : Supprimer TOUT (données incluses)

```bash
# ATTENTION : Cela supprime les données MySQL !
$ docker-compose down --volumes

# Les données sont perdues !
$ docker volume ls
# (rien)
```

## Dépannage

### Problème : Port déjà utilisé

```
Error: bind: address already in use
```

**Solution** : Un autre service utilise le port 80 ou 81

```bash
# Trouver quel processus utilise le port 80
$ netstat -ano | findstr :80

# Ou utiliser un autre port
$ docker-compose -f docker-compose.yaml exec -e STOREFRONT_PORT=8080 up
```

### Problème : Variable TAG non définie

```
Error: invalid reference format: "mysql:"
```

**Solution** : Le fichier `.env` manque ou TAG n'est pas défini

```bash
# Vérifier que .env existe
$ ls -la .env

# Si manquant, créer :
$ echo "TAG=latest" > .env
```

### Problème : Les données ne persistent pas

```
La table créée hier a disparu !
```

**Solution** : Vous avez utilisé `docker-compose down --volumes` qui supprime les volumes !

```bash
# Récupérer le volume
$ docker volume ls
$ docker volume inspect kineteco

# Les données sont perdues si vous avez utilisé --volumes
# Prochaine fois : utiliser `down` sans --volumes
```

---

# Plusieurs fichiers de composition

## Quand utiliser plusieurs fichiers ?

Utilisez plusieurs fichiers Docker Compose quand :
- Les configurations **NE S'EXÉCUTENT JAMAIS ENSEMBLE**
- Les comportements sont **COMPLÈTEMENT DIFFÉRENTS**
- Les fichiers sont dans des **environnements différents**

## Bon cas d'utilisation : environnements différents

**Exemple** : Développement local vs Production

```
docker-compose.yaml         # Configuration de base
docker-compose.local.yaml   # Remplacements pour dev local
docker-compose.staging.yaml # Remplacements pour test
docker-compose.prod.yaml    # Remplacements pour production
```

**Logique** : Vous ne lancez **jamais** local ET production ensemble !

### Pourquoi 3 fichiers ?

- **Local** : Services sans authentication, logs détaillés
- **Staging** : Configuration similaire à production, mais test data
- **Prod** : Vraie production avec vraies données

## Mauvais cas d'utilisation : parties du même système

**NE PAS faire** :

```
docker-compose.storefront.yaml  # ❌ Mauvais
docker-compose.scheduler.yaml   # ❌ Mauvais
```

**Pourquoi** ? Car les développeurs doivent souvent lancer **tout le système** ensemble !

**Solution** : Utilisez plutôt les **profils** pour cette situation.

## Comment marche la fusion

Docker Compose lit **automatiquement** 2 fichiers :
1. `docker-compose.yaml` - Configuration de base
2. `docker-compose.override.yaml` - Remplacements personnalisés

### Comment Docker Compose fusionne les fichiers

**Pour les listes** (comme `depends_on`, `ports`) :
- **Combine** les valeurs des 2 fichiers

```yaml
# docker-compose.yaml
ports:
  - "80:80"

# docker-compose.override.yaml  
ports:
  - "3000:3000"

# Résultat : ports 80 ET 3000 ! ✓
```

**Pour les valeurs uniques** (comme `build`, `image`) :
- **Priorité** au fichier d'override (remplace la base)

```yaml
# docker-compose.yaml
image: mysql:5.7

# docker-compose.override.yaml
image: mysql:8.0

# Résultat : mysql:8.0 ✓ (override gagne)
```

## Conseil important

Les chemins dans le fichier d'override doivent être **relatifs** au fichier principal.

```yaml
# ✓ Correct
build: ./services/storefront

# ❌ Incorrect : trop absolu
build: /home/user/project/services/storefront
```

## Avantages de cette approche

- ✓ Partagez **une configuration de base** entre projets
- ✓ Chaque projet personnalise avec ses **propres overrides**
- ✓ Facile à maintenir et à updater

## Noms personnalisés pour les overrides

Vous pouvez avoir **plusieurs fichiers d'override** :
- `docker-compose.local.yaml`
- `docker-compose.staging.yaml`
- `docker-compose.prod.yaml`

**Mais** : Seul `docker-compose.override.yaml` est **automatiquement** activé.

Pour les autres, utilisez le drapeau `-f`.

## Utiliser plusieurs fichiers manuellement

```bash
# Compose : base + overrides locaux
$ docker-compose -f docker-compose.yaml -f docker-compose.local.yaml up

# Compose : base + overrides production
$ docker-compose -f docker-compose.yaml -f docker-compose.prod.yaml up

# Compose : base + 2 overrides
$ docker-compose -f docker-compose.yaml -f docker-compose.local.yaml -f docker-compose.env.yaml up
```

**Expliqué** :
- `-f` = "utilise ce fichier"
- Compose fusionne les fichiers dans l'ordre

---

# Variables d'environnement avancées

## Alternative aux fichiers d'override

Au lieu d'avoir plusieurs fichiers d'override, utilisez des **variables d'environnement** pour adapter le comportement.

**Avantage** : 1 seul fichier `docker-compose.yaml` pour TOUS les environnements !

## Comment ça marche ?

Les variables d'environnement viennent de 3 sources :
1. **Votre shell** (ligne de commande)
2. **Fichiers `.env`** (fichiers de configuration)
3. **Le fichier composé** (utilise les variables)

Les variables peuvent être **propagées** :
- Du shell → au fichier composé
- Du fichier composé → aux conteneurs

## Cas d'utilisation courants

Les variables d'environnement sont souvent pour :
- **Tags d'images** : `mysql:${MYSQL_VERSION}`
- **Ports** : `${PORT}:3306`
- **Noms d'hôtes** : `${SERVICE_NAME}`
- En réalité : **n'importe quoi** dans le fichier !

## Syntaxe : utiliser une variable

Utilisez `$VARIABLE` ou `${VARIABLE}` :

```yaml
services:
  database:
    image: mysql:${MYSQL_VERSION}  # Utilise la variable MYSQL_VERSION
```

Avant de lancer :

```bash
$ export MYSQL_VERSION=8.0
$ docker-compose up
```

**Résultat** : L'image utilisée sera `mysql:8.0`

## Accolades vs sans accolades

```yaml
# Sans accolades (simple)
image: mysql:$MYSQL_VERSION

# Avec accolades (recommandé, plus lisible)
image: mysql:${MYSQL_VERSION}
```

Les accolades `{}` sont **optionnelles** mais **recommandées** pour la clarté.

## Valeurs par défaut en ligne

Si la variable n'est pas définie, utilisez une valeur par défaut :

```yaml
# Si MYSQL_VERSION n'existe pas, utilise 5.7
image: mysql:${MYSQL_VERSION:-5.7}
```

**Expliqué** :
- `${MYSQL_VERSION:-5.7}` = "utilise MYSQL_VERSION, sinon 5.7"

## Exiger une variable (erreur si manquante)

```yaml
# Erreur si MYSQL_VERSION n'est pas défini
image: mysql:${MYSQL_VERSION:?VERSION requis}
```

Utile pour les configurations critiques.

## Option 1 : Utiliser le shell

```bash
# Définir la variable dans le shell
$ export MYSQL_VERSION=8.0
$ export PORT=3306

# Lancer Compose
$ docker-compose up
```

## Option 2 : Fichier `.env` automatique

Créez un fichier `.env` **à la racine** (même dossier que `docker-compose.yaml`) :

```bash
# .env
MYSQL_VERSION=8.0
PORT=3306
LOG_LEVEL=debug
SERVICE_NAME=storefront
```

Docker Compose le lit **automatiquement** ! 🎉

### Contenu recommandé d'un `.env`

```bash
# Database
MYSQL_VERSION=8.0
MYSQL_ROOT_PASSWORD=secret123

# Ports
STOREFRONT_PORT=80
SCHEDULER_PORT=81
DATABASE_PORT=3306

# Environnement
LOG_LEVEL=info
DEBUG=false
```

## Option 3 : Fichier `.env` personnalisé

```bash
# Utilise un fichier .env différent
$ docker-compose --env-file .env.prod up
```

Utile pour avoir plusieurs configurations `.env`.

## Ordre de priorité (lequel gagne ?)

1. **Variables du shell** ← Plus haute priorité
2. Fichier `.env` personnalisé (`--env-file`)
3. Fichier `.env` standard (automatique)
4. Valeurs par défaut en ligne
5. Pas de valeur ! Erreur

**Exemple** :

```bash
# Fichier .env
MYSQL_VERSION=5.7

# Shell (override le .env)
$ export MYSQL_VERSION=latest
$ docker-compose up

# Résultat : mysql:latest ✓ (shell gagne)
```

## Messages d'erreur personnalisés

Quand une variable est obligatoire, affichez un message sympa :

```yaml
database:
  image: mysql:${MYSQL_VERSION:?Erreur : MYSQL_VERSION n'est pas défini}
```

Si `MYSQL_VERSION` manque, le message d'erreur s'affiche.

## Exemple complet : plusieurs environnements

### Fichier docker-compose.yaml

```yaml
version: '3.8'

services:
  database:
    image: mysql:${MYSQL_VERSION:-5.7}
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-default}
    ports:
      - "${DATABASE_PORT:-3306}:3306"
  
  storefront:
    build: .
    ports:
      - "${STOREFRONT_PORT:-80}:80"
```

### Fichier .env (développement)

```bash
MYSQL_VERSION=5.7
MYSQL_ROOT_PASSWORD=dev_password
STOREFRONT_PORT=8080
DATABASE_PORT=3307
```

### Lancer en développement

```bash
$ docker-compose up  # Utilise .env automatiquement
```

### Lancer en production

```bash
$ export MYSQL_VERSION=latest
$ export MYSQL_ROOT_PASSWORD=prod_secret123
$ export STOREFRONT_PORT=80
$ docker-compose up
```

---

## Résumé : Récapitulatif de KinetEco

Félicitations ! Vous avez appris **tous les concepts Docker Compose** ayant du sens réel à travers le projet **KinetEco**.

### Concepts appliqués dans KinetEco

| Concept | Utilisation dans KinetEco | Fichier |
|---------|--------------------------|---------|
| **Services** | 3 services : storefront, scheduler, database | docker-compose.yaml |
| **Images** | `build` pour storefront/scheduler, `image` pour MySQL | docker-compose.yaml |
| **Ports** | 80/443 pour storefront, 81 pour scheduler | docker-compose.yaml |
| **Dépendances** | storefront et scheduler dépendent de database | docker-compose.yaml |
| **Volumes nommés** | `kineteco:` sauvegarde les données MySQL | docker-compose.yaml + volumes |
| **Volumes chemins** | `./mysql:/docker-entrypoint-initdb.d:ro` charge les scripts SQL | docker-compose.yaml |
| **Variables env** | `.env` pour `TAG=latest` | .env |
| **Fichier env_vars** | Variables MySQL privées | mysql/env_vars |
| **Version** | `3.9` — version récente de Compose | docker-compose.yaml |

### Le projet KinetEco complet

```
kineteco/
├── docker-compose.yaml    ← Configuration complète (3.9)
├── .env                   ← Variables : TAG=latest
├── storefront/
│   ├── Dockerfile         ← FROM nginx:alpine
│   ├── index.html         ← Page web
│   └── kineteco_consumer_files/
├── scheduler/
│   ├── Dockerfile         ← FROM nginx:alpine
│   ├── index.html         ← Page web
│   └── kineteco_scheduler_files/
└── mysql/
    ├── env_vars           ← Secrets MySQL (MYSQL_ROOT_PASSWORD, etc.)
    └── db.sql             ← Script d'initialisation
```

### Commande pour lancer KinetEco

```bash
$ cd kineteco/
$ docker-compose up

# Résultat :
# - Construit storefront:nginx + scheduler:nginx
# - Télécharge mysql:latest (depuis TAG=.env)
# - Lance tous les 3 services en même temps
# - MySQL initialise la base avec db.sql
# - Sauvegarde les données dans le volume 'kineteco'

# Accès :
# - Vitrine    : http://localhost:80
# - Planifcateur : http://localhost:81
# - Monitoring : https://localhost:443
```

### Arrêter et redémarrer (données persistantes)

```bash
$ docker-compose stop    # Pause les services
$ docker-compose start   # Redémarre avec les données intactes ✓
$ docker-compose down    # Arrête en gardant le volume kineteco ✓
$ docker-compose up      # Récupère les données ! ✓
$ docker-compose down --volumes  # ⚠️ Supprime les données !
```



