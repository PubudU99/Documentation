## Azure DevOps for the CST process

Ballerina service call the azure pipelines for triggering the CI-CD pipelines for the Customer Specific Tests process.

Azure dev Ops is used for implementing the pipelines for CI-CD parts of the CST process. CI-pipeline is used to build and push the images of the requested products by the Ballerina service.

pipelines are written in yaml files and main two pipelines are ```CI-Pipeline``` and ```CD-Pipeline```

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
  5. SharedConnection - (Azure Resource Manager connection) This connection is used for the connection between two subscriptions namely ```tg-cs-bnym-001``` and ```
