**GALDEANO**  
**VALETTE**
# TP 3 - ANSIBLE

Pré-requis : 
- installer ansible : 
```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

## Intro

### Setup


Premièrement, il faut vérifier que la dépendance suivante est bien présente en backend (pom.xml) : 
```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Actuator apporte des fonctionnalités prêtes à la production à notre application.

C'est un moyen de surveiller notre application, collecter des métriques, comprendre le trafic ou l'état de notre base de données.

Il utilise des points de terminaison HTTP ou des beans JMX pour nous permettre d'interagir avec lui.
 


### Inventories

1. Créer un dossier `tp_ansible/ansible/inventories` et y ajouter `setup.yml`

Remarque : si on un problème de connexion il faut ajouter 
### Facts


---

## Playbooks
### First playbook

### Advanced playbook

### Using roles


---

## Deploy your app
---

## Front
---

## Continuous deployment

