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
Après cela, j'ai donc construit l'image et je l'ai testée avec la commande curl mentionnée dans les instructions.

![Commande Curl après lancement du conteneur avec l'image précédemment construite](screenshots/Capture d'écran 2024-07-29 185329.png)
