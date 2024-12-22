# CST Process Infrastructure Provisioning on Azure

The tests are planned to be run in a cluster by the Azure DevOps pipeline. Unlike normal pipeline agents, the agent must not be a default agent as the cluster is private and there is no public endpoints for the default agent to connect to the cluster and the discussed architecture was to use **Hub and Spoke** architecture on this Infrastructures.

## Azure Subscriptions
### Shared Subscription (HUB)
- **Name**: `tg-cs-shared-001`
- Hosts infrastructure shared across the spokes.

### Customer Subscriptions (SPOKES)
1. **Bnymellon Subscription**: `tg-cs-bnym-001`
2. **Cagib Subscription**: `tg-cs-cagib-001`

## Hub Infrastructure
The hub hosts resources shared across multiple spokes, provisioned in the **Shared Subscription**.

### Manually Provisioned Resources in the Hub
1. **Resource Group**: For managing hub resources.
2. **Virtual Network (VNet)**: For hosting subnet configurations.
3. **Virtual Machine Scale Set (VMSS)**: Configured for the Azure DevOps pipeline agent pool.

## Spoke Infrastructure
The spokes host customer-specific resources, provisioned in individual customer subscriptions using Terraform.

### Terraform Modules
The WSO2 [terraform-modules](https://github.com/wso2/azure-terraform-modules/tree/main/modules/azurerm/AKS-Generic) are used for automation. These modules simplify the deployment process and ensure consistency across customer subscriptions.

### Planned Infrastructure in the Spokes
1. **Private AKS (Azure Kubernetes Service) Cluster**:
   - Enables secure, private access.
2. **Node Pool Subnet**:
   - Dedicated subnet for cluster node pools.
3. **Load Balancer Subnet**:
   - Supports the load balancer for cluster traffic management.
4. **Network Security Groups (NSG)**:
   - Created for both node pool and load balancer subnets.
5. **NSG Rules**:
   - Configured to control traffic flow.
6. **Route Tables**:
   - Established for network routing.
7. **Role Assignments**:
   - Configured for load balancer and node pools to manage IP assignments.

### Subnet Associations
- NSGs and route tables are associated with the respective subnets to enforce security and routing configurations.

### AKS Cluster Deployment
The **AKS-Generic Module** is utilized to create the private AKS cluster. This ensures:
- No public traffic interference.
- Private access for all associated resources.

## Architectural Diagram
The architecture follows the Hub and Spoke model for secure, private access to resources. 

![Infrastructure Architecture](https://github.com/user-attachments/assets/0b8cb1ff-848c-4763-9079-be2d8d5a57eb)

As in the picture, The hub and spoke architecture is created.
### Private Configuration
- A **Private Fully Qualified Domain Name (FQDN)** is created for the cluster.
- A **Private DNS Zone** is configured to support internal name resolution.
### Vnet Peering
The two Virtual networks in hub and spoke are 




## Summary
This deployment approach ensures:
- Complete isolation of resources in the private cluster.
- Centralized management via the hub.
- Simplified deployment using Terraform modules.
- Secure access using private DNS and NSGs.

For more information or assistance, refer to the [terraform-modules documentation](https://github.com/wso2/azure-terraform-modules/tree/main/modules/azurerm/AKS-Generic).




Hub and Spoke architecture

VMSS creation - For the agent Pool

Agent provisioning script extension installing

Required dependency script

Steps for the debugging VM
1. Install dependencies
2. Az login for authenticate the device 
3. Then add the cluster to the kubeconfig and run the get credentials command
4. Run kubectl get namespace for verification
5. Install the helm chart for the ingress controller
Do not use default ingress controller or any other ingress controllers
Use internal ingress controller as all the resources are private
Use the configuration given in the values.yaml when installing the ingress controller helm chart
6. Initially this was designed to deploy WSO2 APIM 3.2.0 version only.
But this setup can be changed and configured for products with the helm chart correctly configured.
6.1 - WSO2 - APIM 3.2.0 Setup

Helm charts were taken from the public repository - APIM 3.2.0 helm charts

Configuration done to deploy the APIM 3.2.0 helm charts.

Refer this blog - Deploy WSO2 - APIM 3.2.0
















First - Install the script
2. Run the az login -in the azure dev ops pipeline
3. Get credentials of the cluster
4. Link the Vnet to the Private DNS zone
5.Check the connection is working or not


6. Then run helm install nginx with values.yaml
7. Then install the APIM 3.2.0 with serviceChange.yaml
8. Wait and test the connection to the APIM gateway
9.


