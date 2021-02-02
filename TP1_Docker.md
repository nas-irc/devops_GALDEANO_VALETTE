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

2. Ajouter les lignes suivantes au Dockerfile :
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
```java
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

Puis récupérer le fichier générer et le dézipper.

3. Se placer dans le dossier `simple-api/src/main/java/fr/takima/training/simpleapi` et rajouter un fichier java contenant les lignes suivantes :
```java
package fr.takima.training.simpleapi.controller;

import org.springframework.web.bind.annotation.*;
import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {
	private static final String template = "Hello, %s!" ;
	private final AtomicLong counter = new AtomicLong();
	@GetMapping ( "/" )
	public Greeting greeting( @RequestParam (value = "name" , defaultValue = "World" ) String name) {
		return new Greeting( counter .incrementAndGet(), String. format ( template , name));
	}
	class Greeting {
		private final long id ;
		private final String content ;
		public Greeting( long id, String content) {
			this . id = id;
			this . content = content;
		}
		public long getId() {
			return id ;
		}
		public String getContent() {
			return content ;
		}
	}
}
```

4. Ajouter un Dockerfile à la racine du projet contenant les lignes suivantes : 
```
# Build
FROM maven:3.6.3-jdk-11 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM openjdk:11-jre
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
ENTRYPOINT java -jar myapp.jar
```

5. Exécuter la commande `docker build . -t vvalette/myapp` pour créer notre image.

6. Exécuter la commande `docker run --name myapp --rm vvalette/myapp` pour lancer notre container.

Remarque : Un `multistage build` est composé de plusieurs stage, souvent deux, le BUILD et le RUN. Il faut utiliser un `multistage build` car celui-ci nous permet de build (un fichier java par exemple) et de le run par la suite. Ce qui permet d'optimiser notre Dockerfile et donc notre image car celle-ci sera moins lourde et n'embarquera pas toutes les données nécessaire au BUILD mais seulement celle nécessaire au RUN. Avec les builds en plusieurs étpes, on utilise plusieurs instructions FROM dans notre Dockerfile. Chaque instruction FROM peut utiliser une base différente et chacune d'entre elles commence une nouvelle étape de build. On peut copier de manière sélective des artefacts d’une étape à une autre comme pour le COPY par exemple, laissant derrière vous tout ce que l'on ne veut pas dans l’image finale.

### Backend API

1. Créer un network qui va contenir nos deux container : le container BACKEND et le container BDD. Pour cela exécuter la commande `docker network create -d bridge my_app_network`. On a donc un network nommé "my_app_network" qui est créé.

Remarque : On peut lister les networks existant avec la commande `docker network ls`. De plus on peut obtenir des informations sur un network avec la commande `docker network inspect my_app_network`

2. Télécharger et dézipper le fichier https://github.com/takima-training/sample-application-students/releases/download/simple-api/simple-api.zip

3. Lancer le container de la BDD avec l'image créée dans la partie précédente (Database) avec la commande `docker run --name mybdd --net=my_app_network --rm -v /my/own/datadir:/var/lib/postgresql/data vvalette/mypostgres`. L'option `--net=my_app_network` permet d'ajouter notre container à notre network créé précédemment.


4. Modifier le fichier `simple-api/src/main/resources/application.yml` avec les informations de connexions à la BDD
```
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          lob:
            non_contextual_creation: true
    generate-ddl: false
    open-in-view: true
  datasource:
    url: "jdbc:postgresql://172.18.0.2:5432/db"
    username: val
    password: val123
    driver-class-name: org.postgresql.Driver
```
Remarque : On exécute la commande `docker network inspect my_app_network` afin de connaître l'adresse ip du container mybdd. Ici l'IP est 172.18.0.2.

4. Créer l'image de l'application backend avec la commande `dockebuild . -t vvalette/mybackendapi`

5. On lance le container BACKEND dans le même network que le container BDD à l'aide de la commande suivante : `dockn --name mybackend --net=my_app_network --rm -p 8080:8080 vvalette/mybackendapi`. 

6. On peut alors accéder au données de notre BDD grâce à l'api. Par exemple, "http://localhost:8080/departments/IRC/students", nous renvoi le json suivant : 
```json
[
 {"id":1,"name":"IRC"},
 {"id":2,"name":"ETI"},
 {"id":3,"name":"CGP"}
]
```

---

## Http server
### Basics
### Configuration
### Reverse proxy

---

## Link application
### Docker-compose
### Publish
