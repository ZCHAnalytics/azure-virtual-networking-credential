# virtual-networking

Microsoft Guided Project - Configure secure access to workloads with Azure virtual networking services

In this guided project, the company has a web application hosted on Azure. It needs to ensure that these workloads are being accessed securely and therefore requires the following:

1. Network isolation and segmentation for the web application - Create and configure virtual networks: 
- Create a virtual network.
- Configure subnets.
- Configure virtual network peering.

2. Control over the network traffic to and from the web application.- Create and configure network security groups (NSGs)	
Create an NSG.
Associate an NSG to a subnet or a network interface.
Create NSG rules.
Create and use Application Security Groups (ASGs) in NSG rules.

3. Protecting the web application from malicious traffic and blocking unauthorized access - Create and configure Azure Firewall	
Create an Azure Firewall.
Create and configure a public IP address.
Create and configure a firewall policy.

4. Routing traffic to the firewall - Configure network routing	
Create and configure a route table.
Link a route to a subnet.

5. Recording and resolving domain names internally. - Create DNS zones and configure DNS settings	
Create and configure a private DNS zone.
Create and configure DNS records.
Configure DNS settings on a virtual network.

# 1: Network isolation and segmentation for the web application 
Scenario: First, I create an Azure virtual networks with subnets provided by the  company's IT team. Once the virtual network is created, I configure virtual network peering to allows the virtual networks to communicate with each other securely and privately.

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/85a42fa0-d3e9-442a-91c0-42047c8812c4)

Create vritual netwrok. First I create a resource group in azure using Azure cloud shell, powershell: az group create --location uksouth --resource-group RG1
Then to help me job my memore=y, I use az network vnet create --help 

# 2: Scenario
Company requires control of the network traffic to and from the web application. To further enhance the security of the web application, network security groups (NSG) and application security groups (ASG) can be configured. NSG is a security layer that filters network traffic to and from Azure resources, while ASG allows grouping of resources to be managed collectively. 
These security groups provide fine-grained control over the network traffic to and from the web application components.

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/030aa5b0-27aa-4e19-b13d-05fb12c215cb)

Create Application Secuirty group, then Network Security Group associated to backend subnet. Then Use AM template to create VM in Azure Cloud Shell:    $RGName = "RG1"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateUri https://raw.githubusercontent.com/MicrosoftLearning/Configure-secure-access-to-workloads-with-Azure-virtual-networking-services/main/Instructions/Labs/azuredeploy.json
   Output: Parameters              : 
                          Name                              Type                       Value     
                          ================================  =========================  ==========
                          virtualMachines_VM1_name          String                     "VM1"     
                          virtualMachines_VM2_name          String                     "VM2"     
                          publicIPAddresses_VM1_ip_name     String                     "VM1-ip"  
                          publicIPAddresses_VM2_ip_name     String                     "VM2-ip"  
                          virtualNetworks_app_vnet_name     String                     "app-vnet"
                          networkInterfaces_VM1_nic_name    String                     "VM1-nic" 
                          networkInterfaces_VM2_nic_name    String                     "VM2-nic" 
                          
associate ASG to network interface of the VM2 - app-backend-asg

# 3: Firewall 
Scenario
Your organization is looking to protect the web application from malicious traffic and block unauthorized access. In addition to NSG and ASG, a firewall can be configured to add an extra layer of security to the web application.
Azure Firewall policy is a top-level resource that contains security and operational settings for Azure Firewall. It allows you to define a rule hierarchy and enforce compliance. 

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/b8c4a7d7-b51e-42f3-af4b-f8e90712c56f)
Create a subnet 10.1.63.0/24 with a fw-policy and fwpip 
than update FW policy by adding: 
application rule collection to allow access from 10.1.0.0./23 to FQDN azure Pipelines.
then Network Rule collection  to allows DNS from 10.1.0.0./23 UDP 53 to Cloudfare DNS resolution service. 

# 4 - Operationalise 
Scenario: I need to route network traffic to the firewall subnet so it can filter and inspect the traffic. Route tables provide control over the routing of network traffic to and from the web application. 
Network Traffic is subject to the firewall rules when network traffic is routed to the firewall as the subnet default gateway.

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/3f6c9224-b11f-424d-bb9a-257603b64857)

Create a route table, associate route table to app-vnet frontend and backend subnets. and create a route that directs outbound to 0000 to firewallp private ip address.

# 5 Record and resolve domain names internally
Scenario
Company requires workloads to record and resolve domain names internally in virtual networks. Virtual machines can use domain name instead of IPs for internal communication, the domain names will be resolved with a private DNS zone through a virtual network link.
![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/baa11e00-ecbf-4231-a6e0-d8a6e70769d4)

create Private DNS zone, create a virtual network link, then create a DNS record set. 



