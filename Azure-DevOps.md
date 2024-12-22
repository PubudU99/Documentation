## Azure DevOps for the CST process

Ballerina service call the azure pipelines for triggering the CI-CD pipelines for the Customer Specific Tests process.

Azure dev Ops is used for implementing the pipelines for CI-CD parts of the CST process. CI-pipeline is used to build and push the images of the requested products by the Ballerina service.

pipelines are written in yaml files and main two pipelines are ```CI-Pipeline``` and ```CD-Pipeline```

Agent Pools used
  1. Default Azure pipeline agent - Default pipeline agents given by Azure.
  2. Shred VMSS pipeline agent - This is a self hosted pipeline agent created using a VMSS deployed in the shared connection.
Service Hooks used
  1.  CI pipeline status update webhook
  2.  CD pipeline trigger webhook
  3.  CD pipeline status update webhook

Service connections used
  1. aws_connection - For downloading the vanilla products from the aws s3 bucket
  2. wso2_cs - (Github Service Connection) For checkout files for each customer in the CST available customer repositories.
  3. Infrastructure Provisioning Service Connection - (Azure Resource Manager connection) For creating and destroying resources
        - This service connection has owner permission as the Infrastructure has to be provisioned and user roles should be assigned to some resources.[1]
  4. Push Image to ACR - (Docker Registry Type connection) This Service connection is used to build and push images to the Azure container registry.
  5. SharedConnection - (Azure Resource Manager connection) This connection is used for the connection between two subscriptions namely ```tg-cs-bnym-001``` and ```tg-cs-global-shared-001```

Secure Files for pipelines
  1. values.yaml file - For product deployment - (Still this is for only wso2am-3.2.0 version)
  2. terraform.conf.secret.tfvars - For infrastructure provisioning

Teams for the project CST_Project
  1. CST_Project Team - This is used to notify about the pipeline completion or failures in pipelines.

# Pipelines

## Continuous Integration Pipeline

### Pipeline Agent 
- **Default Microsoft pipeline agents**
  
### Parameters
- **product**: The product name (e.g., wso2am, wso2is).
- **version**: The product version (e.g., 3.2.0).
- **customer_update_level**: The specific update level for the customer.
- **update_type**: The typeThe product name (e.g., wso2am, wso2is) of update (regular update, hotfix update).
- **hotfix_pack**: The URL of the hotfix pack.

## Variables
- **one**: Initial value.
- **imageRepository**: The image repository name (e.g., wso2am).
- **containerRegistry**: The container registry URL (e.g., <registry-name>.azurecr.io).
- **tag**: The build tag (e.g., $(Build.BuildId)).

## Stages

### Stage 1 - Tag Update the product

#### Job: Tag_update
- **Step: Clone Repository and Get Latest Tag**
  - Clone the appropriate GitHub repository based on the product.
  - Retrieve the latest tag for the specified version.
  - Set the latest tag as a variable.

### Stage 2 - Build and Push the Image

#### Job: Build_Image
- **Step: Download Vanilla Product**
  - Use AWS CLI to download the vanilla product zip from an S3 bucket.
- **Step: Extract Vanilla Product**
  - Extract the downloaded zip file.
  - Download and extract the update tool.
- **Step: Update Vanilla Product**
  - **Regular Update**
    - If no specific customer update level is provided, update to the latest test level.
    - If a specific customer update level is provided, update to that level.
  - **Hotfix Update**
    - Download and apply the hotfix pack.
- **Step: Get Dockerfile and Modify**
  - Zip the updated product.
  - Retrieve and modify theThe product name (e.g., wso2am, wso2is) Dockerfile from GitHub.
- **Step: Build and Push Image**
  - Log in to the container registry.
  - Build and push the Docker image to the Azure Container Registry (ACR).
 
## Continuous Deployment Pipeline

### Parameters
- **customer**
  - Type: `string`
  - The customer name (e.g., bnymellonprod, cagibsub)
- **helm_overide_value_string**
  - Type: `string`
  - Customer product name (e.g., wso2am-3.2.0,wso2is-5.0.0)

### Variables
Terraform backend
- **terraformstoragerg**- Resource group name where the state file of the terraform configurations are saved.
- **terraformstorageaccount**: `Storage account name where the state file of the terraform configurations are saved.
- **backendContainerName**: Container name where the state file of the terraform configurations are saved.
- **backendKey**: State file name.

Customer spoke resource details (private cluster)
- **clusterName**: AKS cluster name for specific customer deployment (e.g., ```'aks-${{ parameters.customer }}-cst-prod-westeurope-001```).
- **clusterResourceGroup**: AKS cluster resource group name for specific customer deployment (e.g., ```'rg-${{ parameters.customer }}-cst-prod-westeurope'```).
- **azureContainerRegistry**: Azure container registry name where the pushed images by **CI-Pipeline** are stored at.
- **clusterControlPlaneResourceGroup**: AKS cluster resource group name for the control plane of the cluster(e.g., ```'rg-${{ parameters.customer }}-cst-prod-westeurope-001'```)

Common hub resource details
- **sharedSubscription**: Common subscription for shared resources.
- **sharedResourceGroup**: Resource group name of the shared resources (e.g., `'rg-bnymellon-cst-prod-westeurope-shared-001'`).
- **sharedVnet**: Shared Vnet name for the private shared resources (e.g., `'vnet-bnymellon-cst-hub-prod-eastus-shared-001'`).

Ingress controller configurations
- **lbSubnetIP**: LB subnet's IP for the ingress controller (e.g., 10.0.3.9).
- **ingressControllerNamespace**: Namespace name of the Ingress controller 

### Resources
Github Repositories where the Infrastructure Provisioning and product deployment files are stored at.
- **Repositories**
  - **cst-bnymellonprod** 
  - **cst-cagipsub**
  
### Pool
This is the Agent pool where the CD - pipeline tasks are run. This is a self hosted Agent pool which is created using a Virtual Machine Scale Set(VMSS)
which deployed in the shared subscription.

- **name**: `shared-vmss-pool`

### Stages

#### Stage 1 - Infrastructure_Provisioning_Stage

##### Job: Terraform init -> Terraform apply
- **Step :- Checkout Repository**
  - Checkout the repository `cst-${{ parameters.customer }}`.
- **Step :- Display Customer Details**
  - Echo customer details and helm override value string.
- **Step :- Download Terraform Secret File**
  - Download the `terraform.conf.secret.tfvars` file.
- **Step :- Move Secret Files**
  - Move the secret file to the Terraform directory.
- **Step :- Navigate to Infrastructure Directory**
  - Navigate to the Infrastructure directory.
- **Step :- Install Terraform**
  - Install Terraform version 1.9.6.
- **Step :- Run Terraform Commands**
  - Set the Azure subscription.
- **Step:- Terraform init**
  - Initialize Terraform with backend configuration.
- **Step :- Terraform plan**
  - Plan the Terraform deployment.
- **Step :- Terraform apply**
  - Apply the Terraform plan with auto-approve.

#### Stage 2: Deploy_Products_in_the_provisioned_environment

##### Job: Attach ACR -> Link private DNS zone -> login to cluster -> Deploy ingress controller and Product
- **Step :- Checkout Repository**
  - Checkout the repository `cst-${{ parameters.customer }}`.
- **Step :- Download Helm Values File**
  - Download the `values.yaml` file.
- **Step :- Move Secret Files**
  - Move the `values.yaml` file to the relevant directory.
- **Step :- Attach ACR and Link Vnet**
  - Attach the ACR to the cluster and link the Vnet to the private DNS zone.
- **Step :- Login to AKS**
  - Login to the AKS cluster using `kubelogin`.
- **Step :- Install Ingress Controller**
  - Install the ingress controller using Helm.
- **Step :- Wait for Ingress Controller**
  - Wait for the ingress controller to be in a running state.
- **Step :- Check External IP**
  - Check whether the external IP is assigned.
- **Step :- Install APIM**
  - Install APIM using Helm.
- **Step :- Wait for APIM installation**
  - Wait for the APIM to be in a running state.

#### Stage 3: Run_tests_in_the_deployed_product

##### Job: Hostname resolution -> Run tests -> Uninstall the helm charts
- **Step :- Checkout Repository**
  - Checkout the repository `cst-${{ parameters.customer }}`.
- **Step :- Download Terraform Secret File**
  - Download the `terraform.conf.secret.tfvars` file.
- **Step :- Move Secret Files**
  - Move the secret file to the Terraform directory.
- **Step :- Navigate to Infrastructure Directory**
  - Navigate to the Infrastructure directory.
- **Step :- Hostname Resolution**
  - Run the DNS mapping script.
- **Step :- Check Hostnames**
  - Check if hostnames have been resolved correctly.
- **Step :- Install Java and Maven**
  - Install Java 11 and Maven.
- **Step :- Run Tests**
  - Run tests on the product using Maven.

#### Stage 4: Destroy_provisioned_infrastructure

##### Job: Terraform Init -> Terraform Destroy
- **Step :- Checkout Repository**
  - Checkout the repository `cst-${{ parameters.customer }}`.
- **Step :- Download Terraform Secret File**
  - Download the `terraform.conf.secret.tfvars` file.
- **Step :- Move Secret Files**
  - Move the secret file to the Terraform directory.
- **Step :- Navigate to Infrastructure Directory**
  - Navigate to the Infrastructure directory.
- **Step :- Uninstall Helm Charts**
  - Uninstall the ingress controller and APIM Helm charts.
- **Step :- Wait for Uninstallations**
  - Wait until uninstallations are completed.
- **Step :- Install Terraform**
  - Install Terraform version 1.9.6.
- **Step :- Terraform init**
  - Initialize Terraform with backend configuration.
- **Step :- Terraform destroy**
  - Destroy the Terraform-managed infrastructure with auto-approve.


