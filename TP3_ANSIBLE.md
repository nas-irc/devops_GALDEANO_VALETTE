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
 
Par défaut, seulement les endpoints 'health' et 'info' sont accessibles. Pour en ajouter, il faut ajouter au fichier `sample-application-backend/src/main/resources/application.yml` la conf suivante : 

```yml
management:
 server:
  add-application-context-header: false
 endpoints:
  web:
   base-path: /api/actuator
   exposure:
    include: health,info,env,metrics,beans,configprops,prometheus
```
Ainsi nous aurons accès aux endpoints suivants : 
- health : The health endpoint provides detailed information about the health of the application.
- info : The info endpoint provides general information about the application.
- env : 	Returns list of properties in current environment
- metrics : It shows several useful metrics information like JVM memory used, system CPU usage, open files, and much more.
- beans : Returns a complete list of all the Spring beans in your application.
- configprops : The configprops endpoint provides information about the application’s @ConfigurationProperties beans.
- prometheus : The prometheus endpoint provides Spring Boot application’s metrics in the format required for scraping by a Prometheus server.


### Inventories

1. A la racine du projet créer un fichier ansible/inventories/setup.yml: 
```yml
all :
 vars :
  ansible_user : centos
  ansible_ssh_private_key_file : inventories/SSH_KEY/valentin.valette
 children :
  prod :
   hosts : 34.243.109.82                     // or hostname //
```
> Note : Pour 'ansible_ssh_private_key_file' il est conseillé de mettre un chemin absolu car l'execution dépends de notre position courante dans l'arborescence.

2. Tester la connexion avec la commande `ansible all -i inventories/setup.yml -m ping`

Remarque : si on un problème de connexion il faut ajouter `host_key_checking = False` en dessous de `[defaults]` dans `/etc/ansible/ansible.cfg`. 
`
### Facts

Les facts Ansible sont des données liées à vos systèmes distants, y compris les systèmes d'exploitation, les adresses IP, les systèmes de fichiers attachés, etc.

Pour requêter le serveur et avoir la distribution il faut utiliser le module `setup` avec un filtre de la manière suivante : 

`$ ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"` qui donne :
```
nasri.galdeano.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```

---

## Playbooks
### First playbook

1. A la racine du projet créer un fichier ansible/playbook.yml :
```yml
- hosts : all
  gather_facts : false
  become : yes
  tasks :
  - name : Test connection
    ping :
```
Celui ci permet de réaliser une multitude de commande ansible en les regroupandes pas tâches. Dans notre exemple, on réaliser un ping qui est contenu dans une tâche "Test connection"

2. Lancer le playbook  avec la commande `ansible-playbook -i inventories/setup.yml playbook.yml`

### Advanced playbook

### Using roles


---

## Deploy your app
---

## Front
---

## Continuous deployment

