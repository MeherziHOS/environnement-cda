# Environnement CDA — Docker

Projet réalisé dans le cadre de ma formation Concepteur Développeur d'Applications (CDA) à l'AFPA.

## 1. Installation

Environnement utilisé :

- Windows 11
- WSL 2 avec Ubuntu
- Docker Desktop
- Docker Engine 29.9.2
- Docker Compose v5.6.1
- Apache HTTP Server 2.4

Commandes de vérification :

```bash
docker --version
docker compose version
docker run hello-world
```

Résultats : Docker fonctionne correctement et le test `hello-world` est réussi.

## 2. Servir ma-page-dev avec Apache

Le portfolio `ma-page-dev` est développé avec Vite.

### Construire le site

Dans `C:\dev\ma-page-dev` :

```bash
npm run build
```

Cette commande génère le dossier `dist`.

### Lancer Apache avec Docker

Depuis un terminal Windows :

```powershell
docker run -d --name ma-page-apache -p 127.0.0.1:8080:80 --mount "type=bind,source=C:\dev\ma-page-dev\dist,target=/usr/local/apache2/htdocs,readonly" httpd:2.4
```

### Consulter le site

- Accueil : http://localhost:8080
- Projets : http://localhost:8080/projets.html

### Vérifier le conteneur

```bash
docker ps
docker exec ma-page-apache ls /usr/local/apache2/htdocs
docker logs ma-page-apache
```

Les pages HTML sont accessibles et les journaux Apache confirment des réponses HTTP 200.

### Mettre à jour le site

1. Modifier les fichiers du projet `ma-page-dev`.
2. Exécuter `npm run build`.
3. Actualiser le navigateur.

Le dossier `dist` étant monté dans le conteneur, les modifications sont disponibles sans reconstruire l'image Docker.

## 3. Sécurité

- Port publié uniquement sur `127.0.0.1:8080`.
- Dossier du site monté en lecture seule.
- Fichier `.env` exclu du dépôt Git.

## 4. Comparaison des images Docker

J'ai comparé les images officielles `httpd:2.4` et `httpd:2.4-alpine` sur Docker Hub.

- `httpd:2.4` : 177 Mo d'espace disque.
- `httpd:2.4-alpine` : 97,4 Mo d'espace disque.

**Justification :**

J'ai choisi `httpd:2.4` car c'est une image officielle Apache adaptée à notre environnement de formation. Même si la variante Alpine est plus légère, la version standard offre un environnement plus classique pour apprendre et effectuer les exercices. Elle permet mieux de servir mon portfolio `ma-page-dev` avec Apache.

## Fiche 2 — Exercice 2 : Documentation MySQL

1 - Laquelle des variables est obligatoire ?

This variable is mandatory and specifies the password that will be set for the MySQL root superuser account.
MYSQL_ROOT_PASSWORD est la variable obligatoire pour définir le mot de passe de l'administrateur root.

2 - Que deviennent les variables si le volume contient déjà une base ?

any pre-existing database will always be left untouched on container startup.
les variables d'initialisation n'ont plus d'effet sur la base existante. Les données déjà présentes sont conservées.

3 - Pourquoi Adminer peut-il refuser la connexion juste après le premier docker run ?

it will not accept incoming connections until such initialization completes.
Lors du premier démarrage, MySQL doit initialiser sa base de données. Pendant cette opération, il n'accepte pas encore les connexions. Adminer doit donc attendre que MySQL soit prêt.

4 - Où l'image cherche-t-elle des scripts à exécuter au premier démarrage, et de quels types ?

it will execute files with extensions .sh, .sql, .sql.gz, .sql.bz2, .sql.xz, and .sql.zst that are found in /docker-entrypoint-initdb.d.
Les scripts sont recherchés dans le dossier /docker-entrypoint-initdb.d. Ils peuvent être des fichiers SQL, des scripts Shell et des fichiers SQL compressés.

## Fiche 2 — Exercice 3 : MySQL et Adminer

### Installation

- MySQL : `mysql:8.4`
- Adminer : `adminer:latest` (version utilisée : 6.2.0)
- Réseau Docker : `cda`
- Volume persistant : `cda-donnees`
- Base de données : `cda`
- Utilisateur : `cda`
- Mots de passe : définis dans `.env`, non publiés sur GitHub

### Accès

- MySQL : `127.0.0.1:3307`
- Adminer : `http://localhost:8081`
- Serveur dans Adminer : `cda-mysql`

### Commandes de démarrage

MySQL :

```cmd
docker run -d --name cda-mysql --network cda -p 127.0.0.1:3307:3306 --env-file .env -v cda-donnees:/var/lib/mysql mysql:8.4
```

Adminer :

```cmd
docker run -d --name cda-adminer --network cda -p 127.0.0.1:8081:8080 -e ADMINER_DEFAULT_SERVER=cda-mysql adminer:latest
```

### Test de persistance

J'ai créé une table `competences` contenant cinq lignes : HTML, CSS, JavaScript, PHP et Docker.

J'ai arrêté et supprimé le conteneur `cda-mysql`, puis je l'ai recréé avec le même volume `cda-donnees`.

Après redémarrage, la requête `SELECT * FROM competences;` a retrouvé les cinq lignes.

**Conclusion :** les données sont conservées dans le volume Docker, même lorsque le conteneur est supprimé.

### Documentation officielle MySQL

**Section :** Environment Variables

**Phrase anglaise :**

> Any pre-existing database will always be left untouched on container startup.

**Traduction :** Toute base de données préexistante reste inchangée au démarrage du conteneur.
