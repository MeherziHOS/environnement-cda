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
