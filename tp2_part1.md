### Part I : Programmatic approach

## I. Premiers pas

# Créez une VM depuis le Azure CLI


```

az vm create -g oyster -n green_vm --image Ubuntu2204 --admin-username titouan --ssh-key-values ~ /.ssh/id_rsa.pub

```

ce qui nous donne :

```

{
  "fqdns": "",
  "id": "/subscriptions/b6da137f-67ed-49d9-a67c-577c79fb85c3/resourceGroups/oyster/providers/Microsoft.Compute/virtualMachines/green_vm",
  "location": "francecentral",
  "macAddress": "60-45-BD-6D-21-88",
  "powerState": "VM running",
  "privateIpAddress": "10.2.0.5",
  "publicIpAddress": "4.233.67.196",
  "resourceGroup": "oyster",
  "zones": ""
}

```

# Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.

on s'est connecté en ssh et on voit que tou fonctionne grace a cloud init

```

titouan@greenvm:~$ cloud-init status
status: done

```

## II. Un ptit LAN

# Créez deux VMs depuis le Azure CLI

on commence par creer le vnet pour pouvoir mettre les deux vm dedans

```

az>> az network vnet create -g oyster -n VNet --subnet-name Subnet
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"e962ab70-c943-4083-9fde-b5897a0c9014\"",
    "id": "/subscriptions/b6da137f-67ed-49d9-a67c-577c79fb85c3/resourceGroups/oyster/providers/Microsoft.Network/virtualNetworks/VNet",
    "location": "francecentral",
    "name": "VNet",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "oyster",
    "resourceGuid": "6512142a-69ec-40c5-8f80-4aba6522d0e8",
    "subnets": [
      {
        "addressPrefix": "10.0.0.0/24",
        "delegations": [],
        "etag": "W/\"e962ab70-c943-4083-9fde-b5897a0c9014\"",
        "id": "/subscriptions/b6da137f-67ed-49d9-a67c-577c79fb85c3/resourceGroups/oyster/providers/Microsoft.Network/virtualNetworks/VNet/subnets/Subnet",
        "name": "Subnet",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "oyster",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}

```
on creer la premier vm :

```
create -g oyster -n red_vm --image Ubuntu2204 --admin-username titouan --ssh-key-values ~/.ssh/id_rsa.pub --vnet-name VNet --subnet Subnet --nsg-rule NONE
Selecting "northeurope" may reduce your costs. The region you've selected may cost more for the same services. You can disable this message in the future with the command "az config set core.display_region_identified=false". Learn more at https://go.microsoft.com/fwlink/?linkid=222571

{
  "fqdns": "",
  "id": "/subscriptions/b6da137f-67ed-49d9-a67c-577c79fb85c3/resourceGroups/oyster/providers/Microsoft.Compute/virtualMachines/red_vm",
  "location": "francecentral",
  "macAddress": "00-0D-3A-3C-21-FE",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "4.178.185.183",
  "resourceGroup": "oyster",
  "zones": ""
}

```
et la deuxieme :

```
create -g oyster -n blue_vm --image Ubuntu2204 --admin-username titouan --ssh-key-values ~/.ssh/id_rsa.pub --vnet-name VNet --subnet Subnet --nsg-rule NONE
Selecting "northeurope" may reduce your costs. The region you've selected may cost more for the same services. You can disable this message in the future with the command "az config set core.display_region_identified=false". Learn more at https://go.microsoft.com/fwlink/?linkid=222571

{
  "fqdns": "",
  "id": "/subscriptions/b6da137f-67ed-49d9-a67c-577c79fb85c3/resourceGroups/oyster/providers/Microsoft.Compute/virtualMachines/blue_vm",
  "location": "francecentral",
  "macAddress": "00-0D-3A-95-72-A1",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "4.212.244.239",
  "resourceGroup": "oyster",
  "zones": ""
}

```
Maintenant il reste a ping via l'ip privee et voila

```

titouan@redvm:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=2.99 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=2.19 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.893 ms
^C
--- 10.0.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.893/2.024/2.987/0.863 ms

```