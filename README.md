# API Réservation Terrain Badminton

Date de création: 29 décembre 2023 22:21
Author: Mattéo ROSSI

*Ceci est une api permettant de réserver des terrains par créneau horaire !*

## Tables des Matières

- [Lancer le Projet](#lancer-le-projet)
  - [Prérequis](#prérequis)
  - [Lancer le projet avec Docker-Compose](#lancer-le-projet-avec-docker-compose)
- [Tester](#tester)
  - [API](#api)
  - [Base de Données](#base-de-données)
  - [Adminer](#adminer)
- [Base de Données](#base-de-données-1)
- [Conception](#conception)
  - [Dictionnaire des données](#dictionnaire-des-données)
  - [Modèle Conceptuel de Données](#modèle-conceptuel-de-données)
- [Remarques](#remarques)
  - [Difficultés](#difficultés)
- [Références](#références)

## Lancer le Projet

### Prérequis

Avant de lancer le projet, vous devez vous assurer de posséder :

- NodeJS [[lien](https://nodejs.org/en)]
- Docker et Docker-Compose [[lien du téléchargement](https://www.docker.com/get-started/)]
- Cloner le dépot et se placer à la racine du projet

```jsx
git clone https://github.com/TheVisitor-coding/API_Badminton.git
```

### **Lancer le projet avec Docker-Compose**

Dupliquer le fichier `.env.dist`

```
cp .env.dist .env
```

> Vous pouvez modifier les variables d'environnement si vous le souhaitez (des valeurs par défaut sont fournies)
> 

Installer les dépendances de l'application node et générer la doc swagger

```
pushd api
npm install
npm run swagger-autogen
popd
```

Démarrer le projet

```jsx
docker-compose up -d
```

## Tester

### API

Se rendre à l’url `[localhost:3030](http://localhost:3030)` et vérifier son bon lancement

### Base de Données

Pour vérifier le bon fonctionnement de la base de données, ouvrir un terminal de commande puis taper : 

```jsx
docker exec -it badminton-db bash //Permet d'accéder au container de la bdd
mysql -uroot -proot -Dmydb || mysql -uuser -ppassword -Dmydb
```

### Adminer

### Client graphique Adminer pour la base de données MySQL

Le starterpack vient avec [Adminer](https://www.adminer.org/), un gestionnaire de base de données à interface graphique, simple et puissant.

Se rendre sur l'URL [http://localhost:8080](http://localhost:5003/) (par défaut) et se connecter avec les credentials *root* (login *root* et mot de passe *root* par défaut), ou ceux de l'utilisateur (`user` et `password` par défaut) ainsi que `‘db`’ pour le serveur.

## Base de Données

La base de données vient avec deux utilisateurs par défaut :

- `root` (administrateur), mot de passe : `root`
- `user` (utilisateur lambda), mot de passe : `password`

Pour accéder à la base de données :

- *Depuis* un autre conteneur (Node.js, Adminer) : `host` est `db`, le nom du service sur le réseau Docker
- *Depuis* la machine hôte (une application node, PHP exécutée sur votre machine, etc.) : `host` est `localhost` ou `127.0.0.1`. **Préférer utiliser l'adresse IP `127.0.0.1` plutôt que son alias `localhost`** pour faire référence à votre machine (interface réseau qui) afin éviter des potentiels conflits de configuration avec le fichier [socket](https://www.jetbrains.com/help/datagrip/how-to-connect-to-mysql-with-unix-sockets.html) (interface de connexion sous forme de fichier sur les systèmes UNIX) du serveur MySQL installé sur votre machine hôte (si c'est le cas).

## Conception

### Dictionnaire des données

| Ressource | URL | Méthode HTTP | Paramètres d’url et Variations | Commentaires |
| --- | --- | --- | --- | --- |
| terrains | /terrains | GET | isFlooded = 0 ou 1 (afficher uniquement les terrains non innondés) | Aucune contrainte |
| réservations | /terrains/{id}/reservations | GET, POST | {id} → représente l’id du terrain | N’est autorisé à y accéder uniquement les personnes authentifiés.  |
| Terrain Innondé | /terrains/{id}/flooded | GET, POST | {id} → représente l’id du terrain | N’est autorisé à modifier l’état d’un terrain uniquement les personnes administrateur |
| Authentification | /login | GET, POST |  | Quand une personne s’authentifie elle se voit attribuer un token JWT temporaire |

### Modèle Conceptuel de Données

| adherent |
| --- |
| id_adherent (PrimaryKey) |
| pseudo |
| isAdmin |

| field |
| --- |
| id_field (PrimaryKey) |
| field_name |
| is_flooded |

| reservation |
| --- |
| id_reservation (PrimaryKey) |
| id_field (ForeignKey) |
| id_adherent (ForeignKey) |
| begin_date |
| end_date |

adherent → reservation (id_adherent)

field → reservation (id_field)

## Remarques

### Difficultés

Mes principales difficultés dans la production de cette API ont été dans le fonctionnement de l’authentification. J’ai eu quelques soucis à implémenter le token JWT et à récupérer l’id contenu dans le token. Mais j’ai finalement pu réussir à le rendre fonctionnel et j’en suis très content.

Une deuxième difficulté à aussi été la compréhension de comment utiliser une requête *post.* Ma grande difficulté était de comprendre comment faire appel à une requête post.

## Références

Token JWT  ⇒ [https://www.ibm.com/docs/fr/cics-ts/6.1?topic=cics-json-web-token-jwt](https://www.ibm.com/docs/fr/cics-ts/6.1?topic=cics-json-web-token-jwt)

Création d’un user mysql en terminal de commande ⇒ [https://kinsta.com/fr/base-de-connaissances/creer-gerer-utilisateur-mysql/#:~:text=CREATE USER 'exampleuser'%40',mot de passe MySQL ultérieurement](https://kinsta.com/fr/base-de-connaissances/creer-gerer-utilisateur-mysql/#:~:text=CREATE%20USER%20%27exampleuser%27%40%27,mot%20de%20passe%20MySQL%20ult%C3%A9rieurement).
