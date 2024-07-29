# Réalisation du mini Projet conçu par Eazytraining

Les instructions se trouvent dans le fichier nommé "INSTRUCTIONS.md".

## 1ère étape : Construire et Tester

Dans cette première étape, il fallait créer une image Docker d'une application Python. J'ai donc rempli le fichier Dockerfile avec les instructions suivantes :

### Dockerfile

```dockerfile
# On part de l'image de base fournie directement par EazyTraining
FROM python:3.8-buster AS base
LABEL maintainer="N0ll0r" # On nomme celui qui maintient l'image
RUN apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y # On fait les mises à jour des dépôts et on installe les paquets nécessaires
COPY student_age.py  / # On copie le code source à la racine 
COPY requirements.txt / # On copie les prérequis également à la racine
RUN pip3 install -r /requirements.txt # On installe les paquets requis via pip
VOLUME /data # On monte le volume de données qui sera utilisé pour de la persistence
EXPOSE 5000 # On expose le port 5000 pour communiquer avec d'autres conteneurs ou l'extérieur

FROM base AS prod # On pratique le multistage afin de réduire l'image même si cela n'a que peu d'effet et c'est aussi pour mettre en pratique cette notion abordé dans le cours de Dirane
CMD [ "python3", "/student_age.py" ] # Commande qui lance le conteneur
