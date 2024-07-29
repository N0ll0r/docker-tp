# Réalisation du mini Projet conçu par Eazytraining

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
 
# On pratique le multistage afin de réduire l'image même si cela n'a que peu d'effet et c'est aussi pour mettre en pratique cette notion abordé dans le cours de Dirane
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
