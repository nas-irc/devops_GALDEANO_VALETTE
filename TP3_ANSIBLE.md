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

Remarque : Le playbook permet de réaliser différentes tâches. Ici, on réalise uniquement un ping dans notre tâche `Test connection`

### Advanced playbook

1. On créer maintenant un playbook un peu plus complexe qui va nous permettre d'installer docker : 
```yml
- hosts : all
  gather_facts : false
  become : yes
tasks :
   # Install
  - name : Install yum-utils
    yum :
     name : yum-utils
     state : latest

  - name : Install device-mapper-persistent-data
    yum :
     name : device-mapper-persistent-data
     state : latest

  - name : Install lvm2
    yum :
     name : lvm2
     state : latest

  - name : Add Docker stable repository
    yum_repository :
     name : docker-ce
     description : Docker CE Stable - $basearch
     baseurl : https://download.docker.com/linux/centos/7/$basearch/stable
     state : present
     enabled : yes
     gpgcheck : yes
     gpgkey : https://download.docker.com/linux/centos/gpg

  - name : Install Docker
    yum :
     name : docker-ce
     state : present

  - name : Install EPEL
    yum :
     name : epel-release
     state : present

  - name : Install pip
    yum :
     name : python-pip
     state : present

  - name : Install Docker module for python
    pip :
     name : docker

  - name : Make sure Docker is running
    service : name=docker state=started
    tags : docker 
```

Ainsi, ce playbook réalise 9 tâches afin d'intaller Docker. Chaque tâche se voit attribuer un nom puis des actions à réaliser.

### Using roles

L'utilisation des rôles permet de regrouper ensemble des tâches qui permettent de réaliser une partie importante de l'application à déployer. En effet, si toutes les tâches seraient regroupées dans le playbooks principal on ne s'y retrouverait plus.. Ainsi, on créer un rôles `docker`, `database`, `backend`, `frontend` et `network`.

1. Créer les différents rôles avec la commande `ansible-galaxy init roles/docker`.

On obtient donc un dosser ansible/roles/{nom_du_rôle}. Chaque dossier de rôle contient plusieurs dossiers et notamment un dossier task. Dans celui-ci on retrouve un fichier YAML (`main.yml`) qui nous permet de renseigner nos différente tâches. Pour le rôles docker on peut alors déplacer les tâches du playbook dans ce fichier et indiquer dans le playbook qu'il faut exécuter ce rôles docker : 

- Fichier playbook.yml : 
```yml
- hosts : all
  gather_facts : false
  become : yes

  roles :
   - docker
```

- Fichier roles/docker/tasks/mail.yml :
```yml
- name : Install yum-utils
  yum :
   name : yum-utils
   state : latest

- name : Install device-mapper-persistent-data
  yum :
   name : device-mapper-persistent-data
   state : latest

- name : Install lvm2
  yum :
   name : lvm2
   state : latest

- name : Add Docker stable repository
  yum_repository :
   name : docker-ce
   description : Docker CE Stable - $basearch
   baseurl : https://download.docker.com/linux/centos/7/$basearch/stable
   state : present
   enabled : yes
   gpgcheck : yes
   gpgkey : https://download.docker.com/linux/centos/gpg

- name : Install Docker
  yum :
   name : docker-ce
   state : present

- name : Install EPEL
  yum :
   name : epel-release
   state : present

- name : Install pip
  yum :
   name : python-pip
   state : present

- name : Install Docker module for python
  pip :
   name : docker

- name : Make sure Docker is running
  service : name=docker state=started
  tags : docker
   
```

3. Réaliser la même chose pour chaque roles. On peut prendre l'exemple du backend (ne pas oublier de rajouter les rôles dans le playbook):
```yml
- name: Run backend
  docker_container: 
   name: backend
   image: vvalette/backend
   networks: 
    - name: my_app_network
   env: 
     SPRING_DATASOURCE_URL: "jdbc:postgresql://database:5432/SchoolOrganisation"
```

Ici, on vient créer un container nommé `backend` avec l'image créer précédemment et hébergée sur notre docker hub (`vallette/backend`). Puis, on relie notre container au network docker créer dans le roles network. Enfin, on rajoute une variabale d'environnement qui contient le lien pour joindre notre database qui sera contenu dans un autre container et créer elle aussi grâce à un rôle. 

Pour les autres rôles, on peut visualiser la totalité des fichiers de configuraton sur le git suivant : https://github.com/vvalette/sample-application-students (dans le dossier ansible).

---

## Deploy your app
---

## Front
---

## Continuous deployment

