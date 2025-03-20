### Part III : Terraform

## 2. Copy paste

# Constater le déploiement

on commence par faire variables.tf

```
variable "prefix" {
  description = "saudade"
  default     = "numerouno"
}

variable "location" {
  description = "missouri"
  default     = "West Europe"
}

```

puis le gros code bien long de main.tf :

```
provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-rg"
  location = var.location
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "${var.prefix}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "titouan"

  network_interface_ids = []

  admin_ssh_key {
    username   = "titouan"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```
on init tout ca :

```
PS C:\Users\pslnd\terraform-quickstart> terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/azurerm...
- Installing hashicorp/azurerm v4.23.0...
- Installed hashicorp/azurerm v4.23.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```
et on apply

```
terraform apply

```
on verfie ue ca c'est deploye avec :

```
PS C:\Users\pslnd\terraform-quickstart> az vm list -o table
Name          ResourceGroup    Location       Zones
------------  ---------------  -------------  -------
numerouno-vm  NUMEROUNO-RG     westeurope
black_vm      OYSTER           francecentral
red_vm        OYSTER           francecentral

```
On a bien numerouno dans l'ouest !
et on oublie pas de destroy a la fin.

## 3. Do it yourself

# Créer un plan Terraform avec les contraintes suivantes

comme pour la part2 on fait le variables.tf

```
variable "prefix" {
  description = "opium"
  default     = "jacques"
}

variable "location" {
  description = "belgique"
  default     = "West Europe"
}

```

puis le main.tf

```
provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-rg"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "node1" {
  name                = "${var.prefix}-nic-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_interface" "node2" {
  name                = "${var.prefix}-nic-node2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "titouan"

  network_interface_ids = [azurerm_network_interface.node1.id]

  admin_ssh_key {
    username   = "titouan"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

resource "azurerm_linux_virtual_machine" "node2" {
  name                = "${var.prefix}-node2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "titouan"

  network_interface_ids = [azurerm_network_interface.node2.id]

  admin_ssh_key {
    username   = "titouan"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```
puis on init et on apply tout ca 
un petit list -o table pour voir tout ca plus clairement 

```
PS C:\Users\pslnd\terraform-custom> az vm list -o table
Name           ResourceGroup    Location       Zones
-------------  ---------------  -------------  -------
jacques-node1  JACQUES-RG       westeurope
jacques-node2  JACQUES-RG       westeurope
black_vm       OYSTER           francecentral
red_vm         OYSTER           francecentral

```
et on effectue le ping pour voir que tout marche

```
titouan@jacques-node1:~$ ping 10.0.2.4
PING 10.0.2.4 (10.0.2.4) 56(84) bytes of data.
64 bytes from 10.0.2.4: icmp_seq=1 ttl=64 time=2.19 ms
64 bytes from 10.0.2.4: icmp_seq=2 ttl=64 time=1.23 ms
64 bytes from 10.0.2.4: icmp_seq=3 ttl=64 time=1.36 ms
^C
--- 10.0.2.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.228/1.594/2.193/0.426 ms

```

## 4. cloud-iniiiiiiiiiiiiit

# Intégrer la gestion de cloud-init + Proof !


le main.tf ici : 

```
provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "cloud-init-rg"
  location = "West Europe"
}

resource "azurerm_virtual_network" "main" {
  name                = "cloud-init-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "cloud-init-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "main" {
  name                = "cloud-init-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_public_ip" "pip" {
  name                = "cloud-init-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_linux_virtual_machine" "main" {
  name                = "cloud-init-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "azureadmin"

  network_interface_ids = [azurerm_network_interface.main.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  admin_ssh_key {
    username   = "azureadmin"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  custom_data = filebase64("cloud-init.txt")
}

```

et le cloud-inix.txt juste la :

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
et le petit ssh qui va avec : 

```
offset@cloud-init-vm:~$ whoami
offset

```

## B. Go further

# Moar cloud-init and Terraform configuration + Proof !

le main.tf :

```
provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "main" {
  name                 = "${var.prefix}-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_security_group" "main" {
  name                = "${var.prefix}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "Allow-HTTP-WikiJS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_linux_virtual_machine" "main" {
  name                = "${var.prefix}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B2s"
  network_interface_ids = [
    azurerm_network_interface.main.id
  ]

  admin_username = "Kongolese-sous-bbl"

  disable_password_authentication = true

  admin_ssh_key {
    username   = "Kongolese-sous-bbl"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  custom_data = filebase64("cloud-init.txt")
}

```

le cloud-init.txt :

```
#cloud-config
users:
  - default
  - name: titouan
    sudo: true
    shell: /bin/bash
    groups: sudo, docker
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDE9CjbiqpsWI8EeAAe9dKpcMXyRnBJtmznB3P8zAzAbNpyOqrnOZOJBHXpn6yeNekqvmt/MR2xJl/yKbHhy61YmDPMsnM3DurmmjLhTrVV0gF2oqCjBAdXHHEy5J5FL/93ge088mjl1B9WIbUHYSsWTWiK1UbufDEUWt2n5SNiXqXDMWgFpxm8eQvVhCUue3aPQTJLcdnqe61grP/GIt7ary2rPg6acHNWWN/C0tG+iU9cELTWwvQYlUThDkHcvTXKNngLt3/Kw8m6sVYYbhe4rum8bwZdet9dIiTysibyJqAJrcaCySmEFOyNPhgyzsTADXCXL7LudH64GtnsdQVEBIgXtnmvCsmCJjJVqQyVBnAuI6gIXlu8O7dW5qxSlOBz6zpxibGjzKLE6a1WxnNU6t64FEKWAB0DOj+fHKd45M1V6wI9H2WpZVVtY4l4Ursa/FKhd+qD0ld33RrWDpArw0V7LAAaa8E64ZUYsWMC1I6pqCitgBQb2OFaCNKThnDq1AGpLFW6yePCYU1sxkDZ7pvi6F1aQrNuAKcSjS4tG6euKqGEddX7bgegZYxoClFR1f/ZPdjoGtn+XgGOO0Zcc2Lw38cLF7ATwZEep1gEKqUjUz6wYFKRuYs9RB7xi+AtxP4px2qzO0KD8R/LcDyulS6uSQkx0Pj6yJbz/PD0Nw== pslnd@Ordi_Titouan

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - docker-ce
  - docker-ce-cli
  - curl
  - ca-certificates
  - gnupg-agent
  - software-properties-common

runcmd:
  - mkdir /opt/wikijs

write_files:
  - path: /opt/wikijs/docker-compose.yml
    content: |
      version: '3'
      services:
        db:
          image: postgres:15-alpine
          environment:
            POSTGRES_DB: wiki
            POSTGRES_PASSWORD: wikijsrocks
            POSTGRES_USER: wikijs
          logging:
            driver: none
          restart: unless-stopped
          volumes:
            - db-data:/var/lib/postgresql/data
        wiki:
          image: ghcr.io/requarks/wiki:2
          depends_on:
            - db
          environment:
            DB_TYPE: postgres
            DB_HOST: db
            DB_PORT: 5432
            DB_USER: wikijs
            DB_PASS: wikijsrocks
            DB_NAME: wiki
          restart: unless-stopped
          ports:
            - "10101:3000"
      volumes:
        db-data:

    owner: 'titouan:titouan'
    permissions: '0777'

runcmd: 
  - cd /opt/wikijs && docker compose up -d

```
et le docker-compose.yml :

```
version: '3'
services:
  wikijs:
    image: requarks/wiki:latest
    ports:
      - "10101:3000"
    volumes:
      - /etc/wiki:/data

```
