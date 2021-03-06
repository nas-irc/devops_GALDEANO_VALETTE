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
Pour commencer, il faut créer un fichier `.travis.yaml` à la racine du projet. Ce fichier permettra de faire fonctionner Travis  CI (s'il n'y en a pas, travis ignore les actions github) en lui décrivant les étapes à effectuer. La plupart des serrvices CI utilisent un fichier de ce type (sauf jenkins par exemple avec leurs fichiers Groovy).


Une fois le fichier créer, il faut exécuter les commandes suivantes : 
```
$git add .travis.yaml
$git commit -m "init travis CI conf"
$git push origin master
```

### Build and test your app

Maintenant; il faut ajouter au fichier la conf suivante : 
```yml
git:
 depth: 5
jobs:
 include:
  - stage: "Build and Test Java"
    language: java
    jdk: oraclejdk11
    before_script:
    - cd sample-application-backend	
    script :
    - mvn clean verify
  - stage: "Build and Test Nodejs"
    language: node.js
    node_js: "12.20"
    before_script:
    - cd sample-application-frontend
    script :
    - npm install 
    - npm build
    - npm run test
cache:
 directories:
  - "$HOME/.m2/repository"
  - "$HOME/.npm"
```
On distingue dans cette configuration 2 étapes de build & test :

- La première pour le backend utilise la commande `mvn clean verify` dans le dossier backend (car on la met dans le script). qui va build le code et executer les tests. 

Nous avons besoin d'une base de données PostgreSQL pour exécuter les tests d'intégrations qui ont étés écrit. Nous pouvons le faire en utilisant les conteneurs de test Spring Boot. Pour les utiliser, nous devons insérer la dépendance 'embedded-postgresql' suivante :
```xml
<dependency>
 <groupId>com.playtika.testcontainers</groupId>
 <artifactId>embedded-postgresql</artifactId>
 <scope>test</scope>
</dependency>
```
Qui va permettre de faire "pop-up" un conteneur de test "base de données" selon le fichier `src/test/resources/bootstrap.yml` suivant : 
```yml
embedded:
  postgresql:
    dockerImage: "postgres:12.0"

logging:
  level:
    org.testcontainers.shaded.org.zeroturnaround.exec.ProcessExecutor: OFF
```

- La deuxième va fonctionner mais ne lancera rien pour le moment car aucun test n'a encore été créé coté front. On pourrait développer des tests unitaires avec jest ou encore moka.

Pour lancer ces 2 étapes de "build & test" sur Travis CI, il faut effectuer un commit quelconque (modif de la vue par exemple) et aller vérifier.

> Note : Lorsque'on écrit rien dans la partie script et que le langage est "java" alors si le projet a un fichier pom.xml dans la racine du référentiel mais pas de build.gradle, Travis CI construit le projet avec Maven 3:
`test mvn -B`. Aussi si le projet inclut également le script wrapper mvnw dans la racine du référentiel, Travis CI l'utilise à la place: `./mvnw test -B`. Enfin, la commande par défaut ne génère pas de JavaDoc (-Dmaven.javadoc.skip = true).


> Note 2 : Lorsque'on écrit rien dans la partie script et que le langage est "node.js". Alors le script de build par défaut pour les projets utilisant nodejs est: `test npm`. Dans le cas où aucun fichier package.json n'est présent dans le dossier racine, le script de construction par défaut est: `make test`.



### First step into the CD world
1. Créer un compte sur dockerhub : https://hub.docker.com/. Puis créer un répository (liée à notre compte github si possible)

2. Créer une branche `dev` du projet avec les commandes suivantes : 
```
git pull             => récupérer les dernières modifications du master
git checkout -b dev  => créer la branche et se placer dessus en local
git push origin dev  => pusher la branche sur github
git status           => vérifier qu'on est bien sur le branche dev
```

3. Modifier le fichier .travis.yml :
```yml
git :
 depth : 5

stages :
 - "Build and Test"
 - "Package"
 
jobs :
 include :
 - stage : "Build and Test"
   language : java
   jdk : oraclejdk11
   before_script :
    - cd sample-application-backend
   script :
    - mvn clean verify
 - stage : "Build and Test"
   language : node.js
   node_js : "12.20"
   before_script :
    - cd sample-application-frontend
   script :
    - npm install 
    - npm build
 - stage : "Package"
   before_script :
    - cd sample-application-backend
   script :
    - docker build -t vvalette/backend .
    - docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
    - docker push vvalette/backend
 - stage : "Package"
   before_script :
    - cd sample-application-frontend
   script :
    - docker build -t vvalette/frontend .
    - docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
    - docker push vvalette/frontend

cache :
 directories :
  - "$HOME/.m2/repository"
  - "$HOME/.npm"

services :
 - docker
```
2 étapes ajoutées : Package Backend et Package FrontEnd. Ces 2 étapes effectuent un `docker build` puis un `dockerpush` des 2 images.

4. Pour l'étape de connexion, il faut ajouter des variables d'environnement sur travis : More options > Settings > Environment Variables. Choisir le branche dev et ajouter les variables DOCKER_USERNAME et DOCKER_PASSWORD.

Ainsi, à chaque push sur la branche `dev`, le code va être (dans l'ordre) build, test, et deploy afin d'avoir une version fonctionelle de l'image docker en ligne et de pouvoir la réutiliser. 

---

## Setup Quality Gate
### What is quality about ?

La 'Quality' est là pour vous assurer que votre code sera maintenable et déterminer chaque non sécurisé
bloquer. Cela vous aide à produire des fonctionnalités mieux et testées, et cela évitera également d'avoir d
code 'sale' poussé dans votre branche principale.
Pour cela, nous allons utiliser SonarCloud, une solution cloud qui fait des analyses et des rapports de votre code. C'est un outil utile que tout le monde devrait utiliser pour apprendre les best-practices java.

### Register to sonarCloud

1. Créer cotre compte sur https://sonarcloud.io/ et le lier à votre compte github pour pouvoir analyser vos repos.

2. Ajouter votre clé SonnarCLoud à vos variable d'environnements (SONARCLOUD_TOKEN) de Travis.

3. Modifier le .travis.yml :
```yml
git :
 depth : 5

stages :
 - "Build and Test"
 - "Package"
 
jobs :
 include :
 - stage : "Build and Test"
   language : java
   jdk : oraclejdk11
   before_script :
    - cd sample-application-backend
   script :
    - mvn clean verify sonar:sonar -Dsonar.projectKey=vvalette_sample-application-students
 - stage : "Build and Test"
   language : node.js
   node_js : "12.20"
   before_script :
    - cd sample-application-frontend
   script :
    - npm install 
    - npm build
 - stage : "Package"
   before_script :
    - cd sample-application-backend
   script :
    - docker build -t vvalette/backend .
    - docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
    - docker push vvalette/backend
 - stage : "Package"
   before_script :
    - cd sample-application-frontend
   script :
    - docker build -t vvalette/frontend .
    - docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
    - docker push vvalette/frontend

cache :
 directories :
  - "$HOME/.m2/repository"
  - "$HOME/.npm"

services :
 - docker

addons :
 sonarcloud :
  organization : "vvalette"
  token : "$SONARCLOUD_TOKEN"
```
On remarque qu'on a ajouté l'add-on sonarcloud ainsi que le "sonarScanner au moment du `mvn clean verify`.

4.Push le code !

5. Aller sur https://sonarcloud.io/dashboard?id=vvalette_sample-application-students&branch=dev&resolved=false pour visualiser le résultat une fois que la pipeline Travis est terminée.

### Goign further

On rajoute une condition dans nos stages afin de déployer nos images Docker uniquement si on est sur la branch dev :
```yml
stages :
 - "Build and Test"
 - name: "Package"
   if: branch = dev
```
