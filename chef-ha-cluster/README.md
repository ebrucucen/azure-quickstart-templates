# Chef Backend High-Availability Cluster

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fchef-ha-cluster%2Fazuredeploy.json" target="_blank">
<img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fchef-ha-cluster%2Fazuredeploy.json" target="_blank">
<img src="http://armviz.io/visualizebutton.png"/>
</a>

This template deploys a Chef High-Availability Cluster.
`Tags: chef,cluster,ha`

## Deployment steps

This template has artifacts (Configuration Scripts) which are automatically grabbed from github, or can be staged for deployment. Use the below command with the upload flag to deploy this template or provide a storage account and SAS token when using the deploy button above.

This template also uses blob storage to share secrets and configuration templates between nodes in the cluster. You must create a blob storage container for these and provide an SAS token. If you're creating a storage container for artifacts, you can use the same one for secrets storage.

## Ebru's changes: 

1. Deploy.sh file do copy secrets location/sastoken from artifacts directory
2. Template does create 2 vms rather than 3 (10 core limit), each in a different region, vnet
3. There is a vnet peering added to get them talking
4. Changed readme file for ssh connecting to vm's (I can login to FE one, not BE one)
5. The deployment script fails at BE01Setup stage:
```
  "status": "Failed",
  "error": {
    "code": "ResourceDeploymentFailure",
    "message": "The resource operation completed with terminal provisioning state 'Failed'.",
    "details": [
      {
        "code": "VMExtensionProvisioningError",
        "message": "VM has reported a failure when processing extension 'BE0Setup'. Error message: \"Enable failed: processing file downloads failed: failed to download file[0]: failed to download file: unexpected status code: got=404 expected=200\"."
      }
    ]
  ```

## Using the command-line
 ```PowerShell
 .\Deploy-AzureResourceGroup.ps1 -ResourceGroupLocation 'eastus' -ArtifactsStagingDirectory 'chef-ha-cluster' UploadArtifacts
 ```
 Create and put your sshkey into parameters file: 
 ```bash
sshKeyData=$(cat ~/.ssh/azure_rsa.pub)
```
And run 
 ```bash
 deploy.sh -a chef-ha-cluster -l eastus -u
 ```

## Using the "deploy to Azure" button
1. Provision a Standard (LRS) storage account, or use an existing one (must be Standard)
2. Provision a blob storage container underneath storage account.  Note the container URL (ie. https://mystandardstorage.blob.core.windows.net/artifactsfolder )
3. Generate a Shared Acccess Signature (SAS) token with and End date exceeding the life of your cluster.  Note the SAS token.
4.  Click the "deploy to Azure" button at the beginning of this document
5.  Enter in the required fields
  * Artifacts Location:  the container URL from step 2
  * Artifacts Location SAS Token: the SAS token from step 3
  * Chef DNS name: A unique short name (ex: mychefhacluster ) that will be prepended to `.region.cloudapp.azure.com` (ex: `mychefhacluster.westus.cloudapp.azure.com`)
  * SSH Key Data: The contents of your [SSH Public key](https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key) for SSH authentication


## Usage

#### Connect

Connect using ssh
To reach a frontend use port 50000,50001 (FE0,1):
```
ssh -i ~/.ssh/azure_rsa -p 50000 GEN-UNIQUE@GEN-UNIQUE-13.eastus.cloudapp.azure.com
```
To reach a backend do something like
```
ssh -i ~/.ssh/azure_rsa -o ProxyCommand="ssh -W %h:%p -p 50000 -q GEN-UNIQUE@GEN-UNIQUE-13.westus.cloudapp.azure.com" GEN-UNIQUE@be0
```

#### Management

See the chef documentation at [Chef](https://docs.chef.io/)
