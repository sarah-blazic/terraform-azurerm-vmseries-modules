# Palo Alto Networks Transit VNet Dedicated Example

This folder shows an example of Terraform code that helps to deploy a [Transit VNet design model](https://www.paloaltonetworks.com/resources/guides/azure-transit-vnet-deployment-guide) with dedicated VM-Series firewalls on Microsoft Azure.

## Usage

Create a `terraform.tfvars` file and copy the content of `example.tfvars` into it, adjust if needed.

```sh
terraform init
terraform apply
terraform ouput -json password
```

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >=0.13, <0.16 |
| <a name="requirement_azurerm"></a> [azurerm](#requirement\_azurerm) | =2.58 |
| <a name="requirement_random"></a> [random](#requirement\_random) | ~>3.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_azurerm"></a> [azurerm](#provider\_azurerm) | =2.58 |
| <a name="provider_random"></a> [random](#provider\_random) | ~>3.0 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_bootstrap"></a> [bootstrap](#module\_bootstrap) | ../../modules/bootstrap |  |
| <a name="module_inbound_lb"></a> [inbound\_lb](#module\_inbound\_lb) | ../../modules/loadbalancer |  |
| <a name="module_inbound_vmseries"></a> [inbound\_vmseries](#module\_inbound\_vmseries) | ../../modules/vmseries |  |
| <a name="module_outbound_bootstrap"></a> [outbound\_bootstrap](#module\_outbound\_bootstrap) | ../../modules/bootstrap |  |
| <a name="module_outbound_lb"></a> [outbound\_lb](#module\_outbound\_lb) | ../../modules/loadbalancer |  |
| <a name="module_outbound_vmseries"></a> [outbound\_vmseries](#module\_outbound\_vmseries) | ../../modules/vmseries |  |
| <a name="module_vnet"></a> [vnet](#module\_vnet) | ../../modules/vnet |  |

## Resources

| Name | Type |
|------|------|
| [azurerm_network_security_rule.mgmt](https://registry.terraform.io/providers/hashicorp/azurerm/2.58/docs/resources/network_security_rule) | resource |
| [azurerm_public_ip.public](https://registry.terraform.io/providers/hashicorp/azurerm/2.58/docs/resources/public_ip) | resource |
| [azurerm_resource_group.this](https://registry.terraform.io/providers/hashicorp/azurerm/2.58/docs/resources/resource_group) | resource |
| [random_password.this](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_address_space"></a> [address\_space](#input\_address\_space) | The address space used by the virtual network. You can supply more than one address space. | `list(string)` | n/a | yes |
| <a name="input_allow_inbound_data_ips"></a> [allow\_inbound\_data\_ips](#input\_allow\_inbound\_data\_ips) | List of IP CIDR ranges (like `["23.23.23.23"]`) that are allowed to access public data interfaces of VM-Series.<br>If the list is empty, the contents of `allow_inbound_mgmt_ips` are substituted instead. | `list(string)` | `[]` | no |
| <a name="input_allow_inbound_mgmt_ips"></a> [allow\_inbound\_mgmt\_ips](#input\_allow\_inbound\_mgmt\_ips) | List of IP CIDR ranges (like `["23.23.23.23"]`) that are allowed to access management interfaces of VM-Series.<br>If you use Panorama, include its address in the list (as well as the secondary Panorama's). | `list(string)` | `[]` | no |
| <a name="input_common_vmseries_sku"></a> [common\_vmseries\_sku](#input\_common\_vmseries\_sku) | VM-series SKU - list available with `az vm image list -o table --all --publisher paloaltonetworks` | `string` | `"bundle2"` | no |
| <a name="input_frontend_ips"></a> [frontend\_ips](#input\_frontend\_ips) | A map of objects describing frontend IP configurations and rules for the inbound load balancer. See the [loadbalancer documentation](./modules/loadbalancer/README.md) for details. | `any` | n/a | yes |
| <a name="input_inbound_files"></a> [inbound\_files](#input\_inbound\_files) | Map of all files to copy to `inbound_storage_share_name`. The keys are local paths, the values are remote paths. Always use slash `/` as directory separator (unix-like), not the backslash `\`. For example `{"dir/my.txt" = "config/init-cfg.txt"}` | `map(string)` | `{}` | no |
| <a name="input_inbound_lb_name"></a> [inbound\_lb\_name](#input\_inbound\_lb\_name) | Name of the inbound load balancer (the public-facing one). | `string` | `"lb_inbound"` | no |
| <a name="input_inbound_storage_share_name"></a> [inbound\_storage\_share\_name](#input\_inbound\_storage\_share\_name) | Name of storage share to be created that holds `files` for bootstrapping inbound VM-Series. | `string` | n/a | yes |
| <a name="input_inbound_vmseries"></a> [inbound\_vmseries](#input\_inbound\_vmseries) | Map of virtual machines to create to run VM-Series dedicated for traffic inbound from the Internet. Keys are the individual names, values<br>are the objects containing the attributes unique to that individual virtual machine:<br><br>- `avzone`: the Azure Availability Zone identifier ("1", "2", "3"). Default is "1" in order to avoid non-HA deployments.<br>- `trust_private_ip`: the static private IP to assign to the trust-side data interface (nic2). If unspecified, uses a dynamic IP.<br><br>The hostname of each of the VM-Series will consist of a `name_prefix` concatenated with its map key.<br><br>Basic:<pre>{<br>  "fw00" = { avzone = 1 }<br>  "fw01" = { avzone = 2 }<br>}</pre>Full example:<pre>{<br>  "fw00" = {<br>    trust_private_ip = "192.168.0.10"<br>    avzone           = "1"<br>  }<br>  "fw01" = { <br>    trust_private_ip = "192.168.0.11"<br>    avzone           = "2"<br>  }<br>}</pre> | `any` | n/a | yes |
| <a name="input_inbound_vmseries_tags"></a> [inbound\_vmseries\_tags](#input\_inbound\_vmseries\_tags) | A map of tags to be associated with the inbound virtual machines, their interfaces and public IP addresses. | `map` | `{}` | no |
| <a name="input_inbound_vmseries_version"></a> [inbound\_vmseries\_version](#input\_inbound\_vmseries\_version) | Inbound VM-series PAN-OS version - list available with `az vm image list -o table --all --publisher paloaltonetworks` | `string` | `"9.1.3"` | no |
| <a name="input_inbound_vmseries_vm_size"></a> [inbound\_vmseries\_vm\_size](#input\_inbound\_vmseries\_vm\_size) | Azure VM size (type) to be created. Consult the *VM-Series Deployment Guide* as only a few selected sizes are supported. | `string` | `"Standard_D3_v2"` | no |
| <a name="input_location"></a> [location](#input\_location) | The Azure region to use. | `string` | `"East US 2"` | no |
| <a name="input_name_prefix"></a> [name\_prefix](#input\_name\_prefix) | A prefix for all the names of the created Azure objects. It can end with a dash `-` character, if your naming convention prefers such separator. | `string` | `"pantf"` | no |
| <a name="input_network_security_groups"></a> [network\_security\_groups](#input\_network\_security\_groups) | Definition of Network Security Groups to create. Refer to the `vnet` module documentation for more information. | `any` | n/a | yes |
| <a name="input_olb_private_ip"></a> [olb\_private\_ip](#input\_olb\_private\_ip) | The private IP address to assign to the outbound load balancer. This IP **must** fall in the `private_subnet` network. | `string` | `"10.110.0.21"` | no |
| <a name="input_outbound_files"></a> [outbound\_files](#input\_outbound\_files) | Map of all files to copy to `outbound_storage_share_name`. The keys are local paths, the values are remote paths. Always use slash `/` as directory separator (unix-like), not the backslash `\`. For example `{"dir/my.txt" = "config/init-cfg.txt"}` | `map(string)` | `{}` | no |
| <a name="input_outbound_lb_name"></a> [outbound\_lb\_name](#input\_outbound\_lb\_name) | Name of the outbound load balancer. | `string` | `"lb_outbound"` | no |
| <a name="input_outbound_storage_share_name"></a> [outbound\_storage\_share\_name](#input\_outbound\_storage\_share\_name) | Name of storage share to be created that holds `files` for bootstrapping outbound VM-Series. | `string` | n/a | yes |
| <a name="input_outbound_vmseries"></a> [outbound\_vmseries](#input\_outbound\_vmseries) | Map of virtual machines to create to run VM-Series dedicated for traffic outbound to the Internet. Format is the same as for `inbound_vmseries`. | `any` | n/a | yes |
| <a name="input_outbound_vmseries_tags"></a> [outbound\_vmseries\_tags](#input\_outbound\_vmseries\_tags) | A map of tags to be associated with the outbound virtual machines, their interfaces and public IP addresses. | `map` | `{}` | no |
| <a name="input_outbound_vmseries_version"></a> [outbound\_vmseries\_version](#input\_outbound\_vmseries\_version) | Outbound VM-series PAN-OS version - list available with `az vm image list -o table --all --publisher paloaltonetworks` | `string` | `"9.1.3"` | no |
| <a name="input_outbound_vmseries_vm_size"></a> [outbound\_vmseries\_vm\_size](#input\_outbound\_vmseries\_vm\_size) | Azure VM size (type) to be created. Consult the *VM-Series Deployment Guide* as only a few selected sizes are supported. | `string` | `"Standard_D3_v2"` | no |
| <a name="input_password"></a> [password](#input\_password) | Initial administrative password to use for all systems. Set to null for an auto-generated password. | `string` | `null` | no |
| <a name="input_resource_group_name"></a> [resource\_group\_name](#input\_resource\_group\_name) | Name of the Resource Group to create. If not provided, it will be auto-generated. | `string` | `""` | no |
| <a name="input_route_tables"></a> [route\_tables](#input\_route\_tables) | Definition of Route Tables to create. Refer to the `vnet` module documentation for more information. | `any` | n/a | yes |
| <a name="input_storage_account_name"></a> [storage\_account\_name](#input\_storage\_account\_name) | Default name of the storage account to create.<br>The name you choose must be unique across Azure. The name also must be between 3 and 24 characters in length, and may include only numbers and lowercase letters. | `string` | `"pantfstorage"` | no |
| <a name="input_subnets"></a> [subnets](#input\_subnets) | Definition of Subnets to create. Refer to the `vnet` module documentation for more information. | `any` | n/a | yes |
| <a name="input_username"></a> [username](#input\_username) | Initial administrative username to use for all systems. | `string` | `"panadmin"` | no |
| <a name="input_virtual_network_name"></a> [virtual\_network\_name](#input\_virtual\_network\_name) | The name of the VNet to create. | `string` | n/a | yes |
| <a name="input_vnet_tags"></a> [vnet\_tags](#input\_vnet\_tags) | A mapping of tags to assign to the created virtual network and other network-related resources. By default equals to `inbound_vmseries_tags`. | `map(any)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_frontend_ips"></a> [frontend\_ips](#output\_frontend\_ips) | IP Addresses of the inbound load balancer. |
| <a name="output_mgmt_ip_addresses"></a> [mgmt\_ip\_addresses](#output\_mgmt\_ip\_addresses) | IP Addresses for VM-Series management (https or ssh). |
| <a name="output_password"></a> [password](#output\_password) | Initial administrative password to use for VM-Series. |
| <a name="output_username"></a> [username](#output\_username) | Initial administrative username to use for VM-Series. |
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
