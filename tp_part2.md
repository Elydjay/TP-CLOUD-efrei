## Part II : Images

#  Construire votre propre image

Comme demander il y aura que le Dockerfile dans le compte rendu donc le voila : 

```
FROM debian:latest

RUN apt update -y && apt install -y apache2

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN apache2ctl configtest

COPY index2.html /var/www/html/index2.html

EXPOSE 80

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

```