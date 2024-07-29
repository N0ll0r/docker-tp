Mini projet conçu par Eazytraining dont les instructions se trouvent dans le fichier nommé "INSTRUCTIONS.md".

--------- 1ère étape : Construire et tester --------

Dans cette première étape, il fallait créer une image docker d'une application python. 
J'ai donc rempli le fichier Dockerfile avec les isntructions suivantes : 
--------- Dockerfile ---------
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
----------------------------------
 Après cela j'ai donc construit l'image et je l'ai tester avec la commande curl mentionné dans les instructions.

 
 
