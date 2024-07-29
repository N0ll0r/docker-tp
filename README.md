# Mini Projet Conçu par Eazytraining

Les instructions se trouvent dans le fichier nommé "INSTRUCTIONS.md".

## 1ère étape : Construire et Tester

Dans cette première étape, il fallait créer une image Docker d'une application Python. J'ai donc rempli le fichier Dockerfile avec les instructions suivantes :

### Dockerfile

```dockerfile
FROM python:3.8-buster AS base
LABEL maintainer="N0ll0r"
RUN apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
COPY student_age.py  /
COPY requirements.txt /
RUN pip3 install -r /requirements.txt
VOLUME /data
EXPOSE 5000

FROM base AS prod
CMD [ "python3", "/student_age.py" ]
