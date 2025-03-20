### Part II : cloud-init

## 2. Gooooo

# Tester cloud-init

```
az vm create -g oyster -n green_vm --image Ubuntu2204 --admin-username titouan --ssh-key-values ~/.ssh/id_rsa.pub --custom-data "C:/Users/pslnd/OneDrive/Bureau/cloud/cloud-init.txt"

```

# Vérifier que cloud-init a bien fonctionné

mon cloud-init.txt etant ca :

```
#cloud-config
users:
  - default
  - name: toto
    sudo: false
    shell: /bin/bash
    ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDE9CjbiqpsWI8EeAAe9dKpcMXyRnBJtmznB3P8zAzAbNpyOqrnOZOJBHXpn6yeNekqvmt/MR2xJl/yKbHhy61YmDPMsnM3DurmmjLhTrVV0gF2oqCjBAdXHHEy5J5FL/93ge088mjl1B9WIbUHYSsWTWiK1UbufDEUWt2n5SNiXqXDMWgFpxm8eQvVhCUue3aPQTJLcdnqe61grP/GIt7ary2rPg6acHNWWN/C0tG+iU9cELTWwvQYlUThDkHcvTXKNngLt3/Kw8m6sVYYbhe4rum8bwZdet9dIiTysibyJqAJrcaCySmEFOyNPhgyzsTADXCXL7LudH64GtnsdQVEBIgXtnmvCsmCJjJVqQyVBnAuI6gIXlu8O7dW5qxSlOBz6zpxibGjzKLE6a1WxnNU6t64FEKWAB0DOj+fHKd45M1V6wI9H2WpZVVtY4l4Ursa/FKhd+qD0ld33RrWDpArw0V7LAAaa8E64ZUYsWMC1I6pqCitgBQb2OFaCNKThnDq1AGpLFW6yePCYU1sxkDZ7pvi6F1aQrNuAKcSjS4tG6euKqGEddX7bgegZYxoClFR1f/ZPdjoGtn+XgGOO0Zcc2Lw38cLF7ATwZEep1gEKqUjUz6wYFKRuYs9RB7xi+AtxP4px2qzO0KD8R/LcDyulS6uSQkx0Pj6yJbz/PD0Nw== pslnd@Ordi_Titouan

```
On voit bien que j'ai pu me connecter directement avec le nom que j'ai mit dans le .txt

```
toto@greenvm:~$ whoami
toto

```

## 3. Write your own

# Utilisez cloud-init pour préconfigurer la VM :

pour creer la vm on utilise la meme commande az vm create car ce qui change c'est le cloud-init.txt que voici :

```
#cloud-config

users:
  - name: kalash
    password: criminel
    chpasswd: { expire: False }
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDE9CjbiqpsWI8EeAAe9dKpcMXyRnBJtmznB3P8zAzAbNpyOqrnOZOJBHXpn6yeNekqvmt/MR2xJl/yKbHhy61YmDPMsnM3DurmmjLhTrVV0gF2oqCjBAdXHHEy5J5FL/93ge088mjl1B9WIbUHYSsWTWiK1UbufDEUWt2n5SNiXqXDMWgFpxm8eQvVhCUue3aPQTJLcdnqe61grP/GIt7ary2rPg6acHNWWN/C0tG+iU9cELTWwvQYlUThDkHcvTXKNngLt3/Kw8m6sVYYbhe4rum8bwZdet9dIiTysibyJqAJrcaCySmEFOyNPhgyzsTADXCXL7LudH64GtnsdQVEBIgXtnmvCsmCJjJVqQyVBnAuI6gIXlu8O7dW5qxSlOBz6zpxibGjzKLE6a1WxnNU6t64FEKWAB0DOj+fHKd45M1V6wI9H2WpZVVtY4l4Ursa/FKhd+qD0ld33RrWDpArw0V7LAAaa8E64ZUYsWMC1I6pqCitgBQb2OFaCNKThnDq1AGpLFW6yePCYU1sxkDZ7pvi6F1aQrNuAKcSjS4tG6euKqGEddX7bgegZYxoClFR1f/ZPdjoGtn+XgGOO0Zcc2Lw38cLF7ATwZEep1gEKqUjUz6wYFKRuYs9RB7xi+AtxP4px2qzO0KD8R/LcDyulS6uSQkx0Pj6yJbz/PD0Nw== pslnd@Ordi_Titouan
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: docker
    shell: /bin/bash

packages:
  - docker.io

runcmd:
  - sudo systemctl enable docker
  - sudo systemctl start docker
  - sudo docker pull alpine:latest


```
ce qui nous donne quand on se connecte en ssh :

```

kalash@blackvm:~$ sudo docker --version
sudo docker images
Docker version 26.1.3, build 26.1.3-0ubuntu1~22.04.1
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    aded1e1a5b37   4 weeks ago   7.83MB

```