Azure cloud is used to provision the required infrastructure for the CST process.



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


