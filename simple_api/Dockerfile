FROM python:3.8-buster AS base
LABEL maintainer="N0ll0r"
RUN apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
COPY student_age.py  /
COPY requirements.txt /
RUN pip3 install -r /requirements.txt
VOLUME /data

FROM base AS prod
EXPOSE 5000
CMD [ "python3", "/student_age.py" ]
