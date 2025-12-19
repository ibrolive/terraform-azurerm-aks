# Terraform Azure AKS Module Example

This repository demonstrates a production-ready Azure Kubernetes Service (AKS) cluster deployment using the official Azure Terraform AKS module with advanced security and networking configurations.

## Features

This example includes:

- **Secure AKS Cluster**: Private cluster with Azure Policy enabled
- **Disk Encryption**: Customer-managed encryption keys via Azure Key Vault
- **Auto-scaling**: Node pool auto-scaling with configurable min/max nodes
- **Network Configuration**: Custom VNet integration with Azure CNI
- **Application Gateway Ingress**: Built-in ingress controller
- **Monitoring**: Log Analytics workspace integration
- **Maintenance Windows**: Configured automatic maintenance schedules
- **High Availability**: Multi-zone deployment across availability zones
- **OS Configuration**: Custom Linux OS settings and sysctl parameters
- **RBAC**: Azure AD integration with tenant-based authentication

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) >= 1.3
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) - Kubernetes command-line tool
- An Azure subscription
- Azure service principal with contributor access (client_id and client_secret)

## Architecture

The module deploys:

1. **Resource Group** (optional - can use existing)
2. **Virtual Network** with dedicated subnet (10.52.0.0/16)
3. **Azure Key Vault** with disk encryption set
4. **AKS Cluster** with:
   - Kubernetes version 1.30 (automatic patch upgrades)
   - 1-2 node auto-scaling configuration
   - Virtual Machine Scale Sets
   - Private cluster endpoint
   - Application Gateway ingress controller
   - Azure Policy add-on
   - Log Analytics monitoring

## Usage

### Basic Example

```hcl
module "aks" {
  source = "git::https://github.com/ibrolive/terraform-azurerm-aks.git"

  client_id            = var.client_id
  client_secret        = var.client_secret
  location             = "eastus"
  resource_group_name  = "my-aks-rg"
  create_resource_group = false
}
```

### Deploy This Example

1. Clone the repository:
   ```bash
   git clone https://github.com/ibrolive/terraform-azurerm-aks.git
   cd terraform-azurerm-aks
   ```

2. Create a `terraform.tfvars` file:
   ```hcl
   client_id     = "your-service-principal-client-id"
   client_secret = "your-service-principal-secret"
   location      = "eastus"
   ```

3. Initialize and deploy:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

4. Connect to your cluster:
   ```bash
   az aks get-credentials --resource-group <rg-name> --name <cluster-name>
   kubectl get nodes
   ```

## Variables

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| `client_id` | Azure service principal client ID | `string` | n/a | yes |
| `client_secret` | Azure service principal client secret | `string` | n/a | yes |
| `create_resource_group` | Create a new resource group or use existing | `bool` | `true` | no |
| `location` | Azure region for resources | `string` | `"eastus"` | no |
| `resource_group_name` | Name of the resource group | `string` | `null` (auto-generated) | no |
| `key_vault_firewall_bypass_ip_cidr` | IP CIDR to allow access to Key Vault | `string` | `null` (uses current IP) | no |
| `managed_identity_principal_id` | Principal ID for managed identity | `string` | `null` | no |

## Outputs

| Name | Description | Sensitive |
|------|-------------|-----------|
| `test_aks_id` | The AKS cluster resource ID | no |
| `test_host` | The Kubernetes API server endpoint | yes |
| `test_kube_raw` | Raw kubeconfig for cluster access | yes |
| `test_cluster_portal_fqdn` | The Azure portal FQDN for the cluster | no |
| `test_admin_client_certificate` | Admin client certificate | yes |
| `test_admin_client_key` | Admin client key | yes |
| `test_username` | Kubernetes cluster username | yes |
| `test_password` | Kubernetes cluster password | yes |

## Configuration Details

### Network Configuration

- **VNet CIDR**: 10.52.0.0/16
- **AKS Subnet**: 10.52.0.0/24
- **App Gateway Subnet**: 10.52.1.0/24
- **Service CIDR**: 10.0.0.0/16
- **DNS Service IP**: 10.0.0.10

### Node Pool Configuration

- **Node Type**: VirtualMachineScaleSets
- **Auto-scaling**: Enabled (1-2 nodes)
- **Max Pods per Node**: 100
- **OS Disk Size**: 60 GB
- **Availability Zones**: 1, 2
- **Host Encryption**: Enabled

### Maintenance Windows

- **Cluster Upgrade**: Sunday 22:00-23:00
- **Node OS Updates**: Daily 07:00 UTC+1 (16-hour window)
- **Automatic Channel**: Patch upgrades enabled

### Security Features

- **Private Cluster**: Enabled
- **Local Accounts**: Disabled (Azure AD only)
- **Azure Policy**: Enabled
- **Disk Encryption**: Customer-managed keys
- **Confidential Computing**: SGX Quote Helper enabled
- **Network Policy**: Azure Network Policy

## Clean Up

To destroy all resources:

```bash
terraform destroy
```

## Important Notes

1. **Key Vault Firewall**: The Key Vault is configured with network restrictions. Your current public IP is automatically allowed. If deploying from a different location, set `key_vault_firewall_bypass_ip_cidr`.

2. **Private Cluster**: The cluster API endpoint is private. You'll need network connectivity (VPN, ExpressRoute, or jump box) to manage the cluster.

3. **Service Principal**: Ensure your service principal has sufficient permissions to create resources in the target subscription.

4. **Cost Considerations**: This configuration uses Standard SKU load balancers, Application Gateway, and premium Key Vault, which may incur significant costs.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the terms specified in the LICENSE file.

## References

- [Azure Terraform AKS Module](https://github.com/Azure/terraform-azurerm-aks)
- [AKS Documentation](https://docs.microsoft.com/en-us/azure/aks/)
- [Terraform AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

## Author

Ibrahim Dauda - [@ibrolive](https://github.com/ibrolive)
