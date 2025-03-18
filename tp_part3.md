## Part III : docker-compose

# 2. WikiJS

Installez un WikiJS en utilisant Docker

on va directement commencer par le docker-compose.yml car c'est le plus important

```
version: "3.8"

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wikijs
      MYSQL_USER: wikijs
      MYSQL_PASSWORD: wikijs_pass
    volumes:
      - mariadb_data:/var/lib/mysql
    ports:
      - "3306:3306"

  wikijs:
    image: requarks/wiki:latest
    container_name: wikijs
    restart: always
    depends_on:
      - mariadb
    environment:
      DB_TYPE: mariadb
      DB_HOST: mariadb
      DB_PORT: 3306
      DB_USER: wikijs
      DB_PASS: wikijs_pass
      DB_NAME: wikijs
    ports:
      - "3000:3000"
    volumes:
      - wiki_data:/wiki/data

volumes:
  mariadb_data:
  wiki_data:


```
ensuite on up tout ca

```
azureuser@cloudtp:~$ docker-compose up -d --build

```
Enfin quand on va sur http://localhost:3000 on configure wikijs et voila c'est opérationnel.

# 3. Make your own meow

on commence avec le dockerfile


```
FROM python:3.9

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]


```

ensuite on ajoute requirements.txt, app.py et le html dans flask_app/ sans oublier le docker-compose.yml qui est le suivant 

```
version: "3.8"

services:
  redis:
    image: redis:latest
    container_name: redis
    restart: always
    ports:
      - "6379:6379"

  flask_app:
    build: .
    container_name: flask_app
    restart: always
    depends_on:
      - redis
    ports:
      - "8888:8888"


```

reste plus qu'a up tout ca 

```
azureuser@cloudtp:~$ docker-compose up -d --build

```