# OpenShift Origin Deployment Template

This template deploys OpenShift Origin with basic username / password for authentication to OpenShift. This uses CentOS and includes the following resources:

|Resource           |Properties                                                                                                                          |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
|Virtual Network   		|**Address prefix:** 10.0.0.0/8<br />**Master subnet:** 10.1.0.0/16<br />**Node subnet:** 10.2.0.0/16                      |
|Master Load Balancer	|2 probes and 2 rules for TCP 8443 and TCP 9090 <br/> NAT rules for SSH on Ports 2200-220X                                           |
|Infra Load Balancer	|3 probes and 3 rules for TCP 80, TCP 443 and TCP 9090 									                                             |
|Public IP Addresses	|OpenShift Master public IP attached to Master Load Balancer<br />OpenShift Router public IP attached to Infra Load Balancer            |
|Storage Accounts <br />Unmanaged Disks  	|1 Storage Account for Master VMs <br />1 Storage Account for Infra VMs<br />2 Storage Accounts for Node VMs<br />2 Storage Accounts for Diagnostics Logs <br />1 Storage Account for Private Docker Registry<br />1 Storage Account for Persistent Volumes  |
|Storage Accounts <br />Managed Disks      |2 Storage Accounts for Diagnostics Logs <br />1 Storage Account for Private Docker Registry |
|Network Security Groups|1 Network Security Group Master VMs<br />1 Network Security Group for Infra VMs<br />1 Network Security Group for Node VMs  |
|Availability Sets      |1 Availability Set for Master VMs<br />1 Availability Set for Infra VMs<br />1 Availability Set for Node VMs  |
|Virtual Machines   	|3 or 5 Masters. First Master is used to run Ansible Playbook to install OpenShift<br />2 or 3 Infra nodes<br />User-defined number of Nodes (1 to 30)<br />All VMs include a single attached data disk for Docker thin pool logical volume|

If you have a Red Hat subscription and would like to deploy an OpenShift Container Platform (formerly OpenShift Enterprise) cluster, please visit: https://github.com/Microsoft/openshift-container-platform

# General prerequisites

* Administrator rights
* Open ports: 80, 443, 8443, 2200

#### Other thing

* Docker on local OS: https://www.docker.com/community-edition#/download
* Azure CLI 2.0: https://azure.github.io/projects/clis/
* Optional: Visual Studio Code https://code.visualstudio.com + extensions:
    * YAML https://marketplace.visualstudio.com/items?itemName=adamvoss.yaml
    * Docker https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker
    * Python https://marketplace.visualstudio.com/items?itemName=ms-python.python
    * C# https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp
    * Azure Resource Manager Tools https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools
    * Visual Studio Team Services https://marketplace.visualstudio.com/items?itemName=ms-vsts.team
    * Azure Account https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account
    * Azure CLI Tools https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli
    * NuGet Package Manager https://marketplace.visualstudio.com/items?itemName=jmrog.vscode-nuget-package-manager

## Create a new Azure Active Directory tenant

https://account.azure.com/organization

## Add Azure Pass

https://www.microsoftazurepass.com

# Quick deployment in few steps

### Cloud Shell

* ssh-keygen -t rsa

### Azure CLI

Import private ssh key to  KeyVault
* az group create -n kluczessh -l eastus
* az keyvault create -n linuxpolska**ID** -g kluczessh -l eastus --enabled-for-template-deployment true
* az keyvault secret set --vault-name linuxpolska**ID** -n uzytkownik --file ~/.ssh/id_rsa

Add "service principal" and assign to ocplinuxpolska groupp 

* az group create --name ocplinuxpolska --location eastus
* az account list --output table
* az ad sp create-for-rbac -n deployment -p 'warsztatyos' --role contributor --scopes /subscriptions/**twoje-unikalne-subscription-id**/resourceGroups/ocplinuxpolska


# Deploy Template

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Flinuxpolska%2FwarsztatyAzureOpenShift%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
<a href="https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2linuxpolska%2FwarsztatyAzureOpenShift%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/AzureGov.png"/></a>

On new tab : <br/>
* Select existing **ocplinuxpolska** resource group <br/>
* Run below command in **cloud shell**. Next copy content of (public key) command  to text editor and remove CRLF(new line signs). It is very important!!! <br/>
cat ~/.ssh/id_rsa.pub <br/>

* Verify and fill in  below fields:
	* openshift password: warsztatyos<br/>
	* Rhsm username or org Id: askinstructor@linuxpolska.pl<br/>
	* Rhsm passowrd Or Activation Key: askInstructor:)<br/>
	* Rhsm Pool Id: askInstructor:) <br/>
	* Ssh Public Key: **ssh-rsa AAAA…..BB myaccount@secret.local** <- paste here your public key without new lines  <br/>
	* Key Vault Resource Group: kluczessh <br/>
	* Key Vault Name: linuxpolska**ID**<br/>
	* Key Vault Secret: uzytkownik<br/>
	* Aad Client Id: deployment<br/>
	* Aad Client Secret: warztatyos <br/> 

