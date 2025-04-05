### I. Premiers pas Ansible

## 1. Mise en place

# A. Setup Azure

le main.tf

```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }

  required_version = ">= 1.0"
}

provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "cloud-ansible-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "cloud-ansible-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_security_group" "ssh" {
  name                = "${var.prefix}-ssh-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_public_ip" "pip" {
  count               = 2
  name                = "${var.prefix}-vm${count.index + 1}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "nic" {
  count               = 2
  name                = "${var.prefix}-vm${count.index + 1}-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip[count.index].id
  }
}

resource "azurerm_network_interface_security_group_association" "nsg_association" {
  count                     = 2
  network_interface_id      = azurerm_network_interface.nic[count.index].id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "vm" {
  count                 = 2
  name                  = "${var.prefix}-vm${count.index + 1}"
  resource_group_name   = azurerm_resource_group.main.name
  location              = azurerm_resource_group.main.location
  size                  = "Standard_B1s"
  admin_username        = "saboteur"
  network_interface_ids = [azurerm_network_interface.nic[count.index].id]

  admin_ssh_key {
    username   = "saboteur"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  custom_data = filebase64("cloud-init.txt")
}

```

le variables.tf parce que ca va ensemble :

```
variable "prefix" {
  description = "gazo"
  default     = "malagangx"
}

variable "location" {
  description = "france"
  default     = "West Europe"
}

```

et le cloud-init

```

#cloud-config
users:
  - name: saboteur
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDE9CjbiqpsWI8EeAAe9dKpcMXyRnBJtmznB3P8zAzAbNpyOqrnOZOJBHXpn6yeNekqvmt/MR2xJl/yKbHhy61YmDPMsnM3DurmmjLhTrVV0gF2oqCjBAdXHHEy5J5FL/93ge088mjl1B9WIbUHYSsWTWiK1UbufDEUWt2n5SNiXqXDMWgFpxm8eQvVhCUue3aPQTJLcdnqe61grP/GIt7ary2rPg6acHNWWN/C0tG+iU9cELTWwvQYlUThDkHcvTXKNngLt3/Kw8m6sVYYbhe4rum8bwZdet9dIiTysibyJqAJrcaCySmEFOyNPhgyzsTADXCXL7LudH64GtnsdQVEBIgXtnmvCsmCJjJVqQyVBnAuI6gIXlu8O7dW5qxSlOBz6zpxibGjzKLE6a1WxnNU6t64FEKWAB0DOj+fHKd45M1V6wI9H2WpZVVtY4l4Ursa/FKhd+qD0ld33RrWDpArw0V7LAAaa8E64ZUYsWMC1I6pqCitgBQb2OFaCNKThnDq1AGpLFW6yePCYU1sxkDZ7pvi6F1aQrNuAKcSjS4tG6euKqGEddX7bgegZYxoClFR1f/ZPdjoGtn+XgGOO0Zcc2Lw38cLF7ATwZEep1gEKqUjUz6wYFKRuYs9RB7xi+AtxP4px2qzO0KD8R/LcDyulS6uSQkx0Pj6yJbz/PD0Nw== pslnd@Ordi_Titouan

packages:
  - python3
  - python3-pip
  - ansible
runcmd:
  - sudo apt update -y
  - sudo apt upgrade -y

``` 

### 3. Création de nouveaux playbooks

## A. NGINX

# Pour le compte-rendu

fichier nginx.yml

```

- name: Déploiement de NGINX avec HTTPS
  hosts: web
  become: true
  tasks:

    - name: Installer NGINX
      ansible.builtin.yum:
        name: nginx
        state: present

    - name: Générer un certificat SSL auto-signé
      community.crypto.openssl_certificate:
        path: /etc/pki/tls/certs/nginx-selfsigned.crt
        privatekey_path: /etc/pki/tls/private/nginx-selfsigned.key
        provider: selfsigned

    - name: Créer la racine web
      ansible.builtin.file:
        path: /var/www/tp3_site/
        state: directory
        mode: '0755'

    - name: Créer un fichier index.html
      ansible.builtin.copy:
        dest: /var/www/tp3_site/index.html
        content: "<h1>Salut c'est le S !</h1>"
        mode: '0644'

    - name: Déployer la configuration NGINX pour HTTPS
      ansible.builtin.copy:
        dest: /etc/nginx/conf.d/tp3_site.conf
        content: |
          server {
              listen 443 ssl;
              server_name _;
              ssl_certificate /etc/pki/tls/certs/nginx-selfsigned.crt;
              ssl_certificate_key /etc/pki/tls/private/nginx-selfsigned.key;
              root /var/www/tp3_site/;
              index index.html;
          }
        mode: '0644'
      notify: Restart NGINX

    - name: Ouvrir le port 443 dans le firewall
      ansible.builtin.firewalld:
        port: 443/tcp
        permanent: true
        state: enabled
      notify: Reload Firewall

  handlers:
    - name: Restart NGINX
      ansible.builtin.systemd:
        name: nginx
        state: restarted
        enabled: true

    - name: Reload Firewall
      ansible.builtin.systemd:
        name: firewalld
        state: reloaded

```

fichier hosts.ini

```
[web]
10.3.1.11

```

## B. MariaDB ou MySQL

# Pour le compte-rendu

fichier mariadb.yml


```

- name: Déploiement de MariaDB / MySQL
  hosts: db
  become: true
  tasks:

    - name: Installer MariaDB
      ansible.builtin.yum:
        name: mariadb-server
        state: present
      when: ansible_os_family == "RedHat"

    - name: Installer MySQL
      ansible.builtin.apt:
        name: mysql-server
        state: present
      when: ansible_os_family == "Debian"

    - name: Démarrer et activer le service
      ansible.builtin.systemd:
        name: mariadb
        state: started
        enabled: true

    - name: Créer un utilisateur SQL et une base de données
      community.mysql.mysql_user:
        name: userdb
        password: password123
        priv: 'mydb.*:ALL'
        host: '%'
        state: present

    - name: Ouvrir le port 3306 dans le firewall
      ansible.builtin.firewalld:
        port: 3306/tcp
        permanent: true
        state: enabled
      notify: Reload Firewall

  handlers:
    - name: Reload Firewall
      ansible.builtin.systemd:
        name: firewalld
        state: reloaded

```

modif de hosts.ini

```

[db]
10.3.1.12

```
puis ca curl hein








