# Moving on-premises File Server to Azure File with AD authentication and Custom DNS

Many companies that decide to migrate their File Server to Azure File want to keep their user authentication against File servers to keep their ACL rules. So that this may happen they need to set up Azure files (after migration) to authenticate against their on-premises AD. Usually this hybrid scenario requires to keep using company's DNS that usually resides on-premises as well. Below you will see how to migrate an on-premises File Server to Azure and keep users ACL by authenticatingg on AD and using the existing DNS.

FTA Team leading by Alejandra designed a map that may help to visualize others options to migrate to the Cloud. Take a look at here: <link>

![image](https://user-images.githubusercontent.com/97529152/148959058-7b4c661c-7a5a-4511-a5fa-6a44272df21c.png)

## Step by step in how to migrate to Azure File with AD authentication and Custom DNS

This below is the architecture we are going to work
![image](https://user-images.githubusercontent.com/97529152/148958205-0b9b43c9-4ba2-4ce9-b9dd-0a480e1dd927.png)

### Prerequisites

Before we start, let's take a look at in the prerequisites.

This architecture assumes that you have the following:
-	Azure subscription 
-	RBAC permission to perform the deployment of Azure Files, Virtual Network, Azure Firewall and VPN or Express Route.
-	It will be required elevated permission to work with Azure AD to synchronize ADDS with it.


### Step 1 - Sync ADDS with Azure AD
You will have to syncronize your ADDS with Azure AD through AD Connect. There 3 options to syncronize them, but one of the most used is the "hash password sync". So, let's see how to use it and set this up through this link: 
https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-install-express

### Step 2 - create a VNET
Create a virtual network with at least two subnets, one for the private endpoint we are going to use later and one for the DNS proxy server we are going to use in the solution. See how to create a VNET here:
https://docs.microsoft.com/en-us/azure/virtual-network/quick-create-portal

### Step 3 - connecting on-premises on Azure
In case you still don't have your on-premises environment connected on Azure, you will have to build a VPN or ExpressRoute. VPN is the easiest way to do it. See how to deploy a VPN gateway here:
https://docs.microsoft.com/en-us/azure/vpn-gateway/tutorial-create-gateway-portal

### Step 4 - create the Storage Account
Create a storage account and then create a file share. See how to create a file share on top of a Storage account:
https://docs.microsoft.com/en-us/azure/storage/files/storage-how-to-create-file-share?tabs=azure-portal

### Step 5 - create the private endpoint
After you have your storage account and file server done, you will create a private endpoint to your file share. Private endpoints is a NIC (Network Interface Card) created inside a subnet and attached to an Azure service, in this case, the Azure storage. See how to implement it here Private endpoint for storage here:
https://docs.microsoft.com/en-us/azure/storage/common/storage-private-endpoints

### Step 6 - create the DNS Server on VNET to work as a DNS proxy
This solution assumes that DNS server will be in the on-premises environment and that this existent DNS server is the one used to resolve all ip addresses, including Azure file FQDN. However Azure file server FQDN is by default resolved through Azure DNS (Azure internal infrastructure of DNS) and currently there is a rule that require that DNS queries to internal Azure DNS need to be originated from an Azure VNET, and not be originated from the on-premises DNS. 
That means you will need to have a DNS proxy inside the VNET to send those queries to the internal Azure DNS. This is explained through this diagram below that is part of this Microsoft article: https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-dns#on-premises-workloads-using-a-dns-forwarder

![image](https://user-images.githubusercontent.com/97529152/149030657-266911a4-9f33-435d-942a-bb522a0ccd7e.png)

The example above uses a SQL Server instead of a Storage File server, but the example is exactly the same. 

You will also see a VM called DNS forwarder (10.5.0.254), this could be a Windows Server VM with DNS server role, a Linux Server with DNS server or an Azure Firewall with DNS proxy (PaaS).

### Step 7 - set conditional forwarder on on-premises DNS server
Part of the implementation of private endpoint with a custom DNS is set the custom DNS (on-premises) with a conditional forwarder by pointing out the ip address of the new DNS proxy implemented on the VNET. See it on the diagram above how this configuration must look like. This non-Microsoft link may help you in how to do it:
https://www.readandexecute.com/how-to/server-2016/dns/configure-conditional-forwarders-windows-server-2016/

The DNS Domain must be "file.core.windows.net" and the IP address must be the IP Address of the new DNS server created on the VNET.

### Step 8 - set Azure File to be authenticated on on-premises ADDS
Set up Azure File to be authenticated with on-premises AD will be a 4 parts:

#### Part 1 - Enable ADDS authentication for Azure file (run PS command)
https://docs.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-enable

#### Part 2 - Assign share-level permission (RBAC)
https://docs.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-assign-permissions?tabs=azure-portal

#### Part 3 - Configure directory and file level permission over SMB (Windows ACL)
https://docs.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-configure-permissions

#### Part 4 - mount the file share from a domain-joined VM
https://docs.microsoft.com/en-us/azure/storage/files/storage-files-identity-ad-ds-mount-file-share

## Azure Files main features

### General purpose v1 (HDD based):
- not recommended
- doesn’t support lots of new features

### General purpose v2 (HDD based):
- only SMB
- LRS, ZRS, GRS, GZRS
- 5TiB by default or 100TiB only LRS, ZRS

### FileStorage storage account (SSD based)
- SMB or NFS 
- LRS and ZRS
- up to 100TiB by default (in GPv2)
- not available in all regions

### Azure Files best practices

- only use the File Share in a storage account (don't mix with blob or others)
- pay attention in the storage account IOPS limitation
- ideally map file share 1:1 (one file share per storage account) or
- group file shares that are not so active in the same storage account and highly active separated
- don’t use GPv1

### Azure Files tools to move data from on-premises

 

Private endpoint detailed architecture and step-by-step

•	DNS query must be originated from the VNET to Azure DNS.
•	Three options to DNS forwarder:
o	Win VM with DNS Server
o	Linux VM with DNS Server
o	Azure Firewall










1.	Configure Private endpoint for Storage
•	Test access with nslookup
2.	Configure Azure Firewall with DNS proxy
•	Set DNS proxy with Azure DNS IP (168.63.129.16)
•	Az Firewall > Firewall manager > Az FW Policies > DNS
•	Capture Azure Firewall private IP
3.	At your DNS Server on-premises set Conditional forwarder
•	use Azure Firewall private IP
•	Set conditional forwarder to domain “file.core.windows.net”
4.	Test access again with nslookup
5.	Set storage account networking 


Setting up the Authentication with ADDS (on-premises AD)

1.	create Storage Account and File share
2.	enable ADDS authentication for your Azure File share
•	download AzFilesHybrid module to use with Powershell
•	run Join-AzStorageAccountForAuth
•	** run as Administrator **
3.	configure share-level permissions to the users/groups (through RBAC)
4.	configure directory and file level permission (ACL) over SMB
5.	test it.

 
IMPORTANT:

Azure RBAC share-level permissions work as the high-level gatekeeper that determines whether a user can access the share. While the Windows ACLs operate at a more granular level to determine what operations the user can do at the directory or file level. 



