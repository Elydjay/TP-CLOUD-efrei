## 1/ Install 

# Installer Docker votre machine Azure

on utilise la doc officielle, on start docker et on utilise la commande qui nous est donné et efin un petit docker ps pour prouver que tout fonctionne.

```
azureuser@cloudtp:~$ sudo systemctl start docker

azureuser@cloudtp:~$ sudo usermod -aG docker $(whoami)

azureuser@cloudtp:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```

## 3/ Lancement de conteneurs

# Utiliser la commande docker run
# Rendre le service dispo sur internet

```
azureuser@cloudtp:~$ docker run --name web -d -p 9999:80 nginx
d3aac920a032 

azureuser@cloudtp:~$ sudo ufw allow 9999/tcp
Rule added
Rule added (v6)

```
Entre temps on va sur Azure pour ajouter une règle de sécurité pour autoriser le port 9999

```
azureuser@cloudtp:~$ curl http://20.199.44.16:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

# Custom un peu le lancement du conteneur

D'abord dans nginx.conf on le fais écouter sur l port 7777 et on met le chemin vers la racine web

```

server {
    listen 7777;

    root /var/www/cloudtp;
}

```
Ensuite on fais un petit html dans index.html

```

<html>
<head><title>Salut cest Orelsan</title></head>
<body><h1>Bientot la collab NGINX x SPOTIFY</h1></body>
</html>

```
Et enfin on run tout ca

```

azureuser@cloudtp:~$ docker run --name meow \
  -d \
  -p 7777:7777 \
  -v /home/nginx.conf:/etc/nginx/conf.d/custom-nginx.conf \
  -v /home/cloudtp:/var/www/cloudtp \
  --memory=512m \
  nginx

```
Et avec curl on voit que tout marche

```

root@cloudtp:~# curl http://20.199.44.16:7777
<html>
<head><title>Salut cest Orelsan</title></head>
<body><h1>Bientot la collab NGINX x SPOTIFY</h1></body>
</html>

```