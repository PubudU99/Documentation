Azure cloud is used to provision the required infrastructure for the CST process.

The tests are planned to be run in a cluster by the Azure DecOps pipeline. Unlike normal pipeline agents, the agent must not be a default agent as the cluster is private and there is no public endpoints for the default agent to connect to the cluster and the discussed architecture was to use Hub and Spoke architecture on this Infrastructures.

There are two subscription in azure.

Shared Subscription - ```tg-cs-shared-001``` -> HUB
Bnymellon subscription - ```tg-cs-bnym-001``` -> SPOKES
Cagib subscription - ```tg-cs-cagib-001``` -> SPOKES

Infrastructures in the hub will be provisioned in the Shared subscription and Infrastructure for spokes will be provisioned in each of the customer's subscription.

Resources that is provisioned in the Hub(Manually)
1. Resource group for managing the resources
2. Virtual Network for deploy the VMSS instances in specified subnet.
3. Virtual Machine Scale Set for the Azure DevOps pipeline agent pool


Infrastructure is provisioned by using terraform

I have used terraform modules build by the wso2.
[terraform-modules](https://github.com/wso2/azure-terraform-modules/tree/main/modules/azurerm/AKS-Generic)

Planned Infrastructure that is provisioned by the Terraform.
1. Private AKS(Azure Kubernetes Service) cluster.
2. Node pool subnet
3. Load balancer subnet
4. Network Security Groups(NSG) for both the subnets
5. NSG rules for both subnets
6. Route table

Then role assignments are done for the load balancer and the node pool for assigning IPs from the load balancer subnet.

Associations have been created in order to assign the NSGs and route rables to the subnets.

I have used the AKS-Generic module to create the cluster.

Here is the Architectural diagram for the Infrastructure that is provisioned.
[Image link][docs/images/azure_portal/archi.png]

As we dont want public traffic to be interfere or any public access to the cluster, The AKS cluster is a private cluster. 
All the created resources are private. As the settings have been configured to have seperate Resource group for the Cluster resources, private fqdn has been created and a private DNS zone has been configured.



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


