# Azure Bastion

## Instructions

```bash
# ---
# Main Vars
# ---
project="security";                                 echo $project
env="prod";                                         echo $env
app_rg="rg-$project-$env";                          echo $app_rg
l="eastus";                                         echo $l
tags="env=$env project=$project";                   echo $tags

# ---
# NETWORK TOPOLOGY
# ---
vnet_n="vnet-security-$env";                        echo $vnet_n

# ---
# Bastion
# ---
snet_n_bas="AzureBastionSubnet";                    echo $snet_n_bas
snet_addr_bas="10.10.10.0/26";                      echo $snet_addr_bas
nsg_n_bastion="nsg-$project-$env-bastion";          echo $nsg_n_bastion
bastion_n="bastion-$project-$env";                  echo $bastion_n
bastion_pip="pip-bastion-$project-$env";            echo $bastion_pip
bastion_sku="Standard"                              echo $bastion_sku
bastion_pip_sku="Standard"                          echo $bastion_pip_sku
```

```bash
# Bastion
# Bastion NSG
az network nsg create \
--resource-group $app_rg \
--name $nsg_n_bastion \
--location $l \
--tags $tags
# Bastion NSG Rules
# Inbound/Ingress
# AllowHttpsInBound
az network nsg rule create \
--name AllowHttpsInBound \
--resource-group $app_rg \
--nsg-name $nsg_n_bastion \
--priority 120 \
--destination-port-ranges 443 \
--protocol TCP \
--source-address-prefixes Internet \
--destination-address-prefixes "*" \
--access Allow
# AllowGatewayManagerInbound
az network nsg rule create \
--name AllowGatewayManagerInbound \
--direction Inbound \
--resource-group $app_rg \
--nsg-name $nsg_n_bastion \
--priority 130 \
--destination-port-ranges 443 \
--protocol TCP \
--source-address-prefixes GatewayManager \
--destination-address-prefixes "*" \
--access Allow
# AllowAzureLoadBalancerInbound
az network nsg rule create \
--name AllowAzureLoadBalancerInbound \
--direction Inbound \
--resource-group $app_rg \
--nsg-name $nsg_n_bastion \
--priority 140 \
--destination-port-ranges 443 \
--protocol TCP \
--source-address-prefixes AzureLoadBalancer \
--destination-address-prefixes "*" \
--access Allow
# AllowBastionHostCommunication
az network nsg rule create \
--name AllowBastionHostCommunication \
--direction Inbound \
--resource-group $app_rg \
--nsg-name $nsg_n_bastion \
--priority 150 \
--destination-port-ranges 8080 5701 \
--protocol "*" \
--source-address-prefixes VirtualNetwork \
--destination-address-prefixes VirtualNetwork \
--access Allow
# OutBound/Egress
# AllowSshRdpOutbound
az network nsg rule create \
--priority 100 \
--name AllowSshRdpOutbound \
--destination-port-ranges 22 3389 \
--protocol "*" \
--source-address-prefixes "*" \
--destination-address-prefixes VirtualNetwork \
--access Allow \
--nsg-name $nsg_n_bastion \
--resource-group $app_rg \
--direction Outbound
# AllowAzureCloudOutbound
az network nsg rule create \
--priority 110 \
--name AllowAzureCloudOutbound \
--destination-port-ranges 443 \
--protocol TCP \
--source-address-prefixes "*" \
--destination-address-prefixes AzureCloud \
--access Allow \
--nsg-name $nsg_n_bastion \
--resource-group $app_rg \
--direction Outbound
# AllowBastion:Communication
az network nsg rule create \
--priority 120 \
--name AllowBastion:Communication \
--destination-port-ranges 8080 5701 \
--protocol "*" \
--source-address-prefixes VirtualNetwork \
--destination-address-prefixes VirtualNetwork \
--access Allow \
--nsg-name $nsg_n_bastion \
--resource-group $app_rg \
--direction Outbound
# AllowGetSessionInformation
az network nsg rule create \
--priority 130 \
--name AllowGetSessionInformation \
--destination-port-ranges 80 \
--protocol "*" \
--source-address-prefixes "*" \
--destination-address-prefixes Internet \
--access Allow \
--nsg-name $nsg_n_bastion \
--resource-group $app_rg \
--direction Outbound
# Bastion Subnet
az network vnet subnet create \
--resource-group $app_rg \
--vnet-name $vnet_n \
--name $snet_n_bas \
--address-prefixes $snet_addr_bas \
--network-security-group $nsg_n_bastion

# Bastion Public IP
az network public-ip create \
--resource-group $app_rg \
--name $bastion_pip \
--sku $bastion_pip_sku \
--zone 1 2 3 \
--location $l \
--tags $tags

# Bastion (it takes a while go get some coffee)
az network bastion create \
--name $bastion_n \
--public-ip-address $bastion_pip \
--resource-group $app_rg \
--vnet-name $vnet_n \
--location $l \
--sku $bastion_sku \
--tags $tags
```
