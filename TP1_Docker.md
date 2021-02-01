**GALDEANO**  
**VALETTE**
# TP 1 - Docker

## Database
### Basics

1. Créer un répertoir "tp_docker" par exemple se placer à l'intérieur avec la commande `cd tp_docker`. Puis, créer un fichier Dockerfile (avec la commande `touch Dockerfile`).

2. Ouvrir celui-ci avec nano Dockerfile, et y écrire : 
``` 
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db POSTGRES_USER=val POSTGRES_PASSWORD=val123
```

3. Exécuter la commande `sudo docker build . -t vvalette/mypostgres` (vvalette étant notre ussername dockercloud (docker ID) et mypostgres le nom de notre image). Notre image est alors créée sous le nom vvalette/mypostgre.

4. Dans un premier terminal, exécuter la commande `sudo docker run -p 5432:5432 --name mypostgres vvalette/mypostgres` afin de créer notre docker à partir de l'image précédemment créée

5. Pour accéder à notre BDD postgres on peut utiliser adminer. Pour cela, dans un second terminal, exéctuer la commande `sudo docker run --link mypostgres:db -p080:8080 adminer`.

6. Il ne manque plus qu'à se connecter à l'url : 127.0.0.1:8080 pour accèder à adminer, puis rentrer nos données de connexion à notre BDD. On peut donc enfin accèder à celle-ci.

### Init database
### Persist data

## Backend API
### Basics
### Multistage build
### Backend API

## Http server
### Basics
### Configuration
### Reverse proxy


## Link application
### Docker-compose
### Publish
