FROM ubuntu:latest

RUN apt update && apt install -y apache2

EXPOSE 80

CMD apachectl -k start -X # this command starts apache2 in no detached mode