**GALDEANO**  
**VALETTE**
# TP 2 - CI/CD

## Sample App
### Let's Fork
Pour forker l'appli :
1. aller sur le lien suivant : https://github.com/takima-training/sample-application-students
2. Appuyer sur le bouton fork en haut à gauche.
3. Enfin, cloner le repo créé à l'aide de la commande `git clone https://github.com/nas-irc/sample-application-students.git`

### Let's Run
Le fichier docker-compose.yml nous permet de tout installer à l'aide de la ligne de commande suivante :
`docker-compose up -d`
qui nous permet d'obtenir les 3 conteneurs suivants :
```
CONTAINER ID        IMAGE                                  ...       PORTS                NAMES
29df4729c282        sample-application-students_backend    ...       8080/tcp             backend
ea460c8bb345        sample-application-students_frontend   ...       0.0.0.0:80->80/tcp   frontend
5a564b68878e        postgres:12.0-alpine                   ...       5432/tcp             database


```
Et enfin de lancer l'appli en accédant à l'adresse http://localhost/ sur le navigateur.

---

## Setup Travis CI
### Register to Travis
Pour accéder à travis CI avec le compte github, il faut aller sur https://travis-ci.com/signin et sélectionner github.

Il faut ensuite autoriser Travis CI à accéder au repo forké précedemment. Celui-ci apparaîtra ensuite sur le volet à gauche "my repositories"



### First step into the CI world
Pour commencer, il faut créer un fichier `.travis.yaml` à la racine du projet. Ce fichier permetrea de faire fonctionner Travis  CI (nous le verrons par la suite).

Une fois le fichier créer, il faut exécuter les commandes suivantes : 
```
$git add .travis.yaml
$git commit -m "init travis CI conf"
$git push origin master
```

[Remember the differences between each technology. ?? ]
### Build and test your app

### First step into the CD world

---

## Setup Quality Gate
### What is quality about ?
### Register to sonarCloud
### Goign further

