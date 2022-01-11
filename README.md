STORAGE SCENARIO MIGRATIONS

SCENARIO D – File Servers with DFS (Distributed File Name)

Step by step (From scratch) to migrate on-premises File Server into Azure Files

1.	Define what Azure AD (Azure Tenant) you will use
a.	A very new one
b.	The same of your O365 subscription
2.	create an Azure subscription
a.	through EA Portal
b.	through another method (Free, MSDN, PAYG)
3.	Set your permissions (RBAC)
a.	You will need AD permission to perform AD domain join and DNS administration
4.	Sync your ADDS with Azure AD by using Azure AD Connect
a.	Define the type of sync (PTA, Hash Password, ADFS)
5.	create a VNET
a.	create at least two subnets, one for the Private Endpoint and one for the Azure Firewall (DNS proxy)
b.	set custom DNS on your Azure VNET
6.	build a VPN GW
a.	create a gateway subnet in your VNET
b.	in case you are working in an existing Azure environment, you may want to peer some VNETs
c.	through a VPN S2S
d.	through an ExpressRoute
7.	create a Storage Account (see best practices for Azure File)
a.	create a file share
8.	create a private endpoint
9.	create the DNS proxy inside the VNET (to talk to Azure DNS)
a.	it could be a VM with Windows Server and DNS role
b.	or it could be an Azure Firewall working as DNS Proxy
10.	Set up Azure File to be authenticated with on-premises AD
a.	Enable ADDS authentication for Azure file (run PS command)
b.	Assign share-level permission (RBAC)
c.	Configure directory and file level permission over SMB (Windows ACL)
11.	In the on-premises side, create a DNS Conditional Forwarder at your DNS server


Azure Files main features

•	General purpose v1 (HDD based):
o	not recommended
o	doesn’t support lots of new features
•	General purpose v2 (HDD based):
o	only SMB
o	LRS, ZRS, GRS, GZRS
o	5TiB by default or 100TiB only LRS, ZRS
•	FileStorage storage account (SSD based)
o	SMB or NFS / 
o	LRS and ZRS
o	up to 100TiB by default (in GPv2)
o	not available in all regions

Azure Files (some) best practices

•	only use the File Share in a storage account (don't mix with blob or others)
•	pay attention in the storage account IOPS limitation
•	ideally map file share 1:1 (one file share per storage account) or
•	group file shares that are not so active in the same storage account and highly active separated
•	don’t use GPv1

Azure Files tools to move data from on-premises

 

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



