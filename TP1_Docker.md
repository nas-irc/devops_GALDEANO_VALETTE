**GALDEANO**  
**VALETTE**
# TP 1 - Docker

Prérequis : Exécuter la commande `sudo usermod -aG docker {nom_utilisateur}` afin de ne pas avoir à utiliser sudo pour chaque commande docker.

## Database
### Basics

1. Créer un répertoir "tp_docker/BDD" par exemple se placer à l'intérieur avec la commande `cd tp_docker/BDD`. Puis, créer un fichier Dockerfile (avec la commande `touch Dockerfile`).

2. Ouvrir celui-ci avec nano Dockerfile, et y écrire : 
~~~ 
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db POSTGRES_USER=val POSTGRES_PASSWORD=val123
~~~

3. Exécuter la commande `docker build . -t vvalette/mypostgres` (vvalette étant notre ussername dockercloud (docker ID) et mypostgres le nom de notre image). Notre image est alors créée sous le nom vvalette/mypostgres. (Attention à ne pas oublier le point dans la commande !)

4. Dans un premier terminal, exécuter la commande `docker run -p 5432:5432 --name mypostgres vvalette/mypostgres` afin de créer notre container à partir de l'image précédemment créée. L'option `-p 5432:5432 (hôte:invité)` permet d'associer le port hôte au port invité. Cependant, il est déconseillé de le faire car il est plus sécurisé de ne pas avoir accès à la base de données directement.

5. Pour accéder à notre BDD postgres on peut utiliser adminer. Pour cela, dans un second terminal, exéctuer la commande `docker run --link mypostgres:db -p 8080:8080 adminer`. On aurai aussi pu utiliser l'option `-d` lors du lancement du container mypostgres. Celle-ci permet de lancer le container en mode détaché (arrière plan). Ainsi `--link mypostgres:db` permet de linker adminer à notre container "mypostgres" créé précedemment. Cependant, l'option link est utilisée uniquement dans ce cas la avec adminer pour du DEBUG. En PROD, il faudrait mettre les deux container dans le même network (docker network).

6. Il ne manque plus qu'à se connecter à l'url : 127.0.0.1:8080 pour accèder à adminer, puis rentrer nos données de connexion à notre BDD. On peut donc enfin accèder à celle-ci.

Remarques : 
- Utiliser -e pour fournir les variables d'environnements afin d'éviter de les écrire en dur. Il faut donc exécuter la commande `docker run -p 5432:5432 --name mypostgres -e POSTGRES_DB=db POSTGRES_USER=val POSTGRES_PASSWORD=val123 vvalette/mypostgres` sans oublier de régénérer l'image sans la ligne `ENV POSTGRES_DB=db POSTGRES_USER=val POSTGRES_PASSWORD=val123`
- Si votre serveur postgresql ne veut pas se lancer car le port est déjà utilisé: lancez  `systemctl stop postgresql`


### Init database

1. Créer deux scripts sql (01-CreateScheme.sql et 02-InsertData.sql) et les placer dans le même dossier que le fichier Dockerfile.

2. Ajouter les lignes suivantes au Dokerfile :
```
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/
```
Ces lignes permettent de copier les scripts dans le dossier `docker-entrypoint-initdb.d` et seront exécuter par ordre alphabétique au lancement du container.

3. Lancer le docker avec la commande habituelle. On observe donc que la base de donnée est initialiser à la création du container.

### Persist data

- Exécuter la commande `docker run -p 5432:5432 --name mypostgres -v /my/own/datadir:/var/lib/postgresql/data vvalette/mypostgres` afin de créer un volume et de stoker les données de la base de données même si le container est supprimé. C'est l'option `-v /my/own/datadir:/var/lib/postgresql/data` qui permet cela.

Remarques :
- Supprimer un container : `docker fm -f {nom_container ou ID)`
- Lister les container : `docker ps -a`

---

## Backend API
### Basics

1. Installer un jdk pour compiler et exécuter du java : `apt install default-jdk`

2. Créer un fichier Main.java contenant les lignes suivantes :
```
public class Main {
public static void main (String[] args) {
System. out .println( "Hello World!" ) ;
}
}
```

3. Le compiler avec la commande `javac Main.java`

4. Exécuter l'exécutable avec la commande `java Main`

### Multistage build

1. Créer un Dockerfile contenant les lignes suivantes : 
```
FROM openjdk:11 AS BUILD_IMAGE
# Copy resource 
COPY Main.java /usr/src/
# Build Main.java
RUN javac /usr/src/Main.java

FROM openjdk:11-jre
# Copy resource from previous stage
COPY --from=BUILD_IMAGE /usr/src/Main.class .
# Run Main
ENTRYPOINT java Main
```
Remarque : 
- On peut aussi mettre `--from=0` et ne pas mettre `AS BUILD_IMAGE` afin de prendre le premier stage. Cela revient au même.
- RUN : exécuter comme build donc on a pas l'affichage, il faut utiliser `ENTRYPOINT java Main`

2. On lance le container avec la commande `docker run --name myapi  -- rm vvalette/myapi`. L'option `--rm` permet de supprimer automatiquement le container une fois qu'il est terminé.

3. Créer une application Springboot sur : https://start.spring.io/ en utilisant la configuration suivante :
```
- Project: Maven
- Language: Java 11
- Spring Boot: 2.2.4 : Cette version n'existe pas on a donc utilisé la version 2.4.2
- Packaging: Jar
- Dependencies: Spring Web
```
### Backend API

---

## Http server
### Basics
### Configuration
### Reverse proxy

---

## Link application
### Docker-compose
### Publish
