# Réalisation du mini projet Docker conçu par Eazytraining

Les instructions se trouvent dans le fichier nommé "INSTRUCTIONS.md".

## 1ère étape : Construire et Tester

Dans cette première étape, il fallait créer une image Docker d'une application Python. J'ai donc rempli le fichier Dockerfile avec les instructions suivantes :

### Dockerfile

```dockerfile
# On part de l'image de base fournie directement par EazyTraining
FROM python:3.8-buster AS base
# On nomme celui qui maintient l'image
LABEL maintainer="N0ll0r"
# On fait les mises à jour des dépôts et on installe les paquets nécessaires
RUN apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
# On copie le code source à la racine
COPY student_age.py  /
# On copie les prérequis également à la racine 
COPY requirements.txt /
# On installe les paquets requis via pip
RUN pip3 install -r /requirements.txt
# On monte le volume de données qui sera utilisé pour de la persistence
VOLUME /data
# On expose le port 5000 pour communiquer avec d'autres conteneurs ou l'extérieur
EXPOSE 5000
 
# On pratique le multistage afin de réduire l'image même si cela n'a que peu d'effets et c'est aussi pour mettre en pratique cette notion abordée dans le cours de Dirane
FROM base AS prod
# Commande qui lance le conteneur
CMD [ "python3", "/student_age.py" ]

```

Après cela, j'ai donc construit l'image et je l'ai testé avec la commande curl mentionnée dans les instructions.


![Commande Curl](https://github.com/N0ll0r/mini-project-docker/blob/main/screenshots/Capture%20d'%C3%A9cran%202024-07-29%20185329.png?raw=true)


## 2nde étape : Infrastructure as code (IAC)

À cette étape, il fallait maintenant déployer l'image API avec docker-compose et ajouter une interface graphique.

Par conséquent, j'ai rempli le docker-compose.yml de la façon suivante avec des tests réguliers afin de trouver la bonne recette. 


```yaml
services:
  php-apache:
    image: php:apache
    container_name: php-apache
# Comme souhaité, le volume est persistent avec la jonction entre ces 2 dossiers
    volumes:
      - ./website/:/var/www/html/
# Comme mentionné dans les instructions, le conteneur php-apache ne s'exéccute que si le conteneur api est démarré
    depends_on:
      - api
    ports:
      - "80:80"
# Un réseau est lié entre ce conteneur et le conteneur api
    networks:
      - student-list
    environment:
#      - URL="http://api:5000/pozos/api/v1.0/get_student_ages"
      - USERNAME=toto
      - PASSWORD=python

  api:
    image: student-list:latest
    container_name: api
#    ports:
#      - "5000:5000"
# Le conteneur api ne communique qu'avec le conteneur php-apache qu'à travers ce réseau, il n'est pas accessible depuis l'extérieur
    networks:
      - student-list
# Création du volume
    volumes:
      - $PWD/simple_api/student_age.json:/data/student_age.json
# Déclaration du réseau interne à api et php-apache
networks:
  student-list:

```


Après de nombreux tests/erreurs, voici la page web qui s'affiche :


![Page_web](https://github.com/N0ll0r/mini-project-docker/blob/main/screenshots/Capture%20d'%C3%A9cran%202024-07-28%20120129.png?raw=true)

## 3ème étape : Docker registry

L'objectif est de mettre en place un registry privé pour y stocker notre image docker précédemment créée. 

Pour cela, j'ai décidé de créer un regsitre docker avec une interface graphique. 
J'ai repris et adapté le docker-compose;yml de ce dépôt : [docker-registry-ui](https://github.com/Joxit/docker-registry-ui)

Ce qui donne : 
```yaml
version: '3.8'
services:
  registry-ui:
    image: joxit/docker-registry-ui:main
    restart: always
    ports:
      - 8080:80
    environment:
      - SINGLE_REGISTRY=true
      - REGISTRY_TITLE=Private registry of Nollor
      - DELETE_IMAGES=true
      - SHOW_CONTENT_DIGEST=true
      - NGINX_PROXY_PASS_URL=http://registry-server:5000
      - SHOW_CATALOG_NB_TAGS=true
      - CATALOG_MIN_BRANCHES=1
      - CATALOG_MAX_BRANCHES=1
      - TAGLIST_PAGE_SIZE=100
      - REGISTRY_SECURED=false
      - CATALOG_ELEMENTS_LIMIT=1000
    container_name: registry-ui


  registry-server:
    image: registry:2.8.2
    restart: always
    environment:
# J'ai précisé l'adresse IP de mon host
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Origin: '[http://192.168.1.166]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Methods: '[HEAD,GET,OPTIONS,DELETE]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Credentials: '[true]'
      REGISTRY_HTTP_HEADERS_Access-Control-Allow-Headers: '[Authorization,Accept,Cache-Control]'
      REGISTRY_HTTP_HEADERS_Access-Control-Expose-Headers: '[Docker-Content-Digest]'
      REGISTRY_STORAGE_DELETE_ENABLED: 'true'
# J'ai aussi précisé le dossier à utiliser pour la persistence des données via : ./registry/data
    volumes:
      - ./registry/data:/var/lib/registry
    container_name: registry-server
# Ici j'ai dû exposer le port 5000 afin de pouvoir push mon image dessus à l'instar de ce qui est proposé dans le cours docker
    ports:
      - "5000:5000"

```

On push l'image après l'avoir taguée  de cette façon : docker tag student-list:latest localhost:5000/student-list

Et on obtient ça quand on se rend sur l'interface graphique :

![image](https://github.com/N0ll0r/mini-project-docker/blob/main/screenshots/Capture%20d'%C3%A9cran%202024-07-28%20213939.png?raw=true)

![image](https://github.com/N0ll0r/mini-project-docker/blob/main/screenshots/Capture%20d'%C3%A9cran%202024-07-28%20213953.png?raw=true)


C'est ici que s'achève mon premier mini-projet Docker et c'était sympa à réaliser.  Cela me pousse à en réaliser d'autres afin de monter en compétences sur cette technologie qui est tout bonnement formidable !
