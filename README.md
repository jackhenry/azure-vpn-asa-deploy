# Azure VPN and Cisco ASA IKEV2 Deployment

This repo contains an Ansible playbook for deploying an Azure S2S VPN between a Cisco ASA and an Azure Virtual Network.

These playbooks may come in handy when creating CI/CD pipelines for your network environment.

Azure Key Vault is used to provide necessary secrets during the deployment (Cisco credentials, IPsec shared secret, etc.)

## Initial Setup

### Azure Key Vault

Create an Azure Key Vault and add the following secrets:

| Secret | Description |
|--------|-------------|
|asa-user|Username of ASA user|
|asa-pass|Password of ASA user|
|asa-enable-pass|Enable password of ASA device|
|vpn-shared-secret|The IPsec VPN shared secret to be used|

### Ansible Setup

These playbooks depend on the Azure Ansible collection.

Documentation for setup can be found [here.](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/docs/?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW)

### Update `vars.yml`

After cloning the repository, locate and update the `vars.yml` file:

```
git clone https://github.com/jackhenry/azure-vpn-asa-deploy.git
cd azure-vpn-asa-deploy.git
cat vars.yml
```

```yml
resource_group: VPN-RG
location: westus2
keyvaulturl: "keyvault-address-here" # Azure Key Vault Address
sharedkey: "a secret" # IKEV2 Shared Secret
vnet:
  name: VNet1 # Name of Azure Virtual Network
  address_net: 172.177.0.0/16 # VNet Address Space
  subnet_net: 172.177.0.0/24 # First VNet Subnet
  subnet_name: "Resource Subnet" # First Vnet Subnet Name
  gw_subnet_net: 172.177.10.0/24 # Required GatewaySubnet Network
  gw_name: VnetGw1 # VPN Gateway NAme
  public_ip_name: GwPip # Public IP Name
local:
  gw_name: "ASAGW" # Local Gateway Name
  public_ip_addr: 1.1.1.1 # Public IP Address of Local Device
  prefix: 192.168.10.0/24 # Local Network on Device
  net: "192.168.10.0 mask 255.255.255.0"
  gw_peer_addr: 192.168.20.2 # BGP Peering Address
  gw_peer_addr_net: "192.168.20.0 mask 255.255.255.252"
  vpn_route_ip: 192.168.20.3
  asn: 64555 # Local Device ASN
  vpn_conn_name: AzureVPN # VPN Tunnel Interface Name
```

### ASA Setup

The ASA device must be accessible from the Ansible Control Node.

You must create an inventory file and include the ASA device as a host before running the playbook.

For example, if your ASA device has the IP address of `192.168.20.1`, create the following inventory file:

```
[asadevices]
192.168.20.1
```

## Running the Playbooks

### VPN Creation

```
ansible-playbook create-vpn.yml -i path/to/inventory --extra-vars "@vars.yml"
```

### VPN Removal

```
ansible-playbook rm-vpn.yml -i path/to/inventory --extra-vars "@vars.yml"
```
