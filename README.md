<h1> AZ-104 </h1>

###  Notes to help me prepare for the Microsoft AZ-104


**Connect** <br />
When you're working with a local install of Azure PowerShell, you'll need to authenticate before you can execute Azure commands. The `Connect-AzAccount` cmdlet prompts for your Azure credentials, then connects to your Azure subscription. It has many optional parameters, but if all you need is an interactive prompt, you don't need any parameters:

Use the `Get-AzContext` cmdlet to determine which subscription is active.

Get a list of all subscription names in your account with the Get-AzSubscription command.

You can retrieve a list of all Resource Groups in the active subscription.
`Get-AzResourceGroup`
To get a more concise view, you can send the output from the Get-AzResourceGroup to the Format-Table cmdlet using a pipe '|'.
`Get-AzResourceGroup` | `Format-Table`

You can create resource groups by using the `New-AzResourceGroup` cmdlet. You must specify a name and location. The name must be unique within your subscription. The location determines where the metadata for your resource group will be stored (which may be important to you for compliance reasons). 

New-AzResourceGroup -Name <name> -Location <location>

Create an Azure Virtual Machine
Another common task you can do with PowerShell is to create VMs.

Azure PowerShell provides the `New-AzVm` cmdlet to create a virtual machine. The cmdlet has many parameters to let it handle the large number of VM configuration settings. Most of the parameters have reasonable default values, so we only need to specify five things:

- ResourceGroupName: The resource group into which the new VM will be placed.
- Name: The name of the VM in Azure.
- Location: Geographic location where the VM will be provisioned.
- Credential: An object containing the username and password for the VM admin account. We'll use the Get-Credential cmdlet. This cmdlet will prompt for a username and password and package it into a credential object.
- Image: The operating system image to use for the VM, which is typically a Linux distribution or Windows Server.
```powershell
New-AzVm
       -ResourceGroupName <resource group name>
       -Name <machine name>
       -Credential <credentials object>
       -Location <location>
       -Image <image name>
  ```
You can supply these parameters directly to the cmdlet as shown in the previous example. Alternatively, you can use other cmdlets to configure the virtual machine, such as `Set-AzVMOperatingSystem`, `Set-AzVMSourceImage`, `Add-AzVMNetworkInterface`, and `Set-AzVMOSDisk`.

Here's an example that strings the Get-Credential cmdlet together with the `-Credential` parameter:
  | Command | Description |
  | --- | --- |
  | `Remove-AzVM` | Deletes an Azure VM |
  | `Start-AzVM` | 	Start a stopped VM |
  | `Stop-AzVM` | Stop a running VM |
  | `Restart-AzVM` | Restart a VM |
  | `Update-AzVM`	| Updates the configuration for a VM |
  
  You can list the VMs in your subscription using the Get-AzVM -Status command. This command also supports entering a specific VM by including the -Name property. Here, we'll assign it to a PowerShell variable:
  ```powershell
  $vm = Get-AzVM  -Name MyVM -ResourceGroupName ExerciseResources
```

The interesting thing is that now your VM is an object with which you can interact. For example, you can make changes to that object, then push changes back to Azure by using the Update-AzVM command:

```powerShell
$ResourceGroupName = "ExerciseResources"
$vm = Get-AzVM  -Name MyVM -ResourceGroupName $ResourceGroupName
$vm.HardwareProfile.vmSize = "Standard_DS3_v2"

Update-AzVM -ResourceGroupName $ResourceGroupName  -VM $vm
```
The interactive mode in PowerShell is appropriate for one-off tasks. In our example, we'll likely use the same resource group for the lifetime of the project, so creating it interactively is reasonable. Interactive mode is often quicker and easier for this task than writing a script and executing that script exactly once.

You can reach into complex objects through a dot (".") notation. For example, to see the properties in the VMSize object associated with the HardwareProfile section, run the following command:
```powershell
$vm.HardwareProfile

VmSize          VmSizeProperties
------          ----------------
Standard_D2s_v3 
```
To get information on one of the disks, run the following command:
```powershell
$vm.StorageProfile.OsDisk

OsType                  : Linux
EncryptionSettings      : 
Name                    : testvm-eus-01_disk1_3d16b00420464f0b9052cbc39e535925
Vhd                     : 
Image                   : 
Caching                 : ReadWrite
WriteAcceleratorEnabled : 
DiffDiskSettings        : 
CreateOption            : FromImage
DiskSizeGB              : 30
ManagedDisk             : Microsoft.Azure.Management.Compute.Models.ManagedDiskP
                          arameters
DeleteOption            : Detach
```
 to get your public IP address:
 ```powershell
 Get-AzPublicIpAddress -ResourceGroupName [resource group name] -Name "testvm-01"
 ```
 
 ### Delete a VM
 To try out some more commands, let's delete the VM. We'll shut it down first:

```powershell
Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```
Now, let's delete the VM by running the Remove-AzVM cmdlet:

```powershell
Remove-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```

Run this command to list all the resources in your resource group:
```powershell
Get-AzResource -ResourceGroupName $vm.ResourceGroupName | Format-Table
```
Delete the network interface:

```PowerShell
$vm | Remove-AzNetworkInterface ???Force
```
Delete the managed OS disks and storage account:

```PowerShell
Get-AzDisk -ResourceGroupName $vm.ResourceGroupName -DiskName $vm.StorageProfile.OSDisk.Name | Remove-AzDisk -Force
```
Next, delete the virtual network:

```PowerShell
Get-AzVirtualNetwork -ResourceGroupName $vm.ResourceGroupName | Remove-AzVirtualNetwork -Force
```
Delete the network security group:

```PowerShell
Get-AzNetworkSecurityGroup -ResourceGroupName $vm.ResourceGroupName | Remove-AzNetworkSecurityGroup -Force
```
And finally, delete the public IP address:

```PowerShell
Get-AzPublicIpAddress -ResourceGroupName $vm.ResourceGroupName | Remove-AzPublicIpAddress -Force
```
### Variables
In the last unit, you saw that PowerShell supports variables. Use $ to declare a variable and = to assign a value. For example:
```powershell
$loc = "East US"
$iterations = 3
```
### Loops
PowerShell has several loop structures, including `For`, `Do...While`, and `For...Each`. The For loop is the best match for our needs because we'll execute a cmdlet a fixed number of times.

The following example shows the core syntax. The example runs for two iterations and prints the value of i each time. The comparison operators are written `-lt` for "less than", `-le` for "less than or equal", `-eq` for "equal", `-ne` for "not equal", etc.
```powershell 
For ($i = 1; $i -lt 3; $i++)
{
    $i
}
```

Start by capturing the input parameter in a variable. Add the following line to your script.
Prompt for a username and password for the VM's admin account and capture the result in a variable, create a loop that executes three times, and in the loop body, create a name for each VM and store it in a variable, then output it to the console. Last, create a VM using the $vmName variable:
```powershell
param([string]$resourceGroup)

$adminCredential = Get-Credential -Message "Enter a username and password for the VM administrator."

For ($i = 1; $i -le 3; $i++)
{
    $vmName = "ConferenceDemo" + $i
    Write-Host "Creating VM: " $vmName
    New-AzVm -ResourceGroupName $resourceGroup -Name $vmName -Credential $adminCredential -Image Canonical:0001-com-ubuntu-server-focal:20_04-lts:latest
}
```
# What Azure resources can be managed using the Azure CLI?
The Azure CLI lets you control nearly every aspect of every Azure resource. You can work with resource groups, storage, virtual machines, Azure Active Directory (Azure AD), containers, machine learning, and so on.

Commands in the CLI are structured in groups and subgroups. Each group represents a service provided by Azure, and the subgroups divide commands for these services into logical groupings. For example, the storage group contains subgroups including account, blob, and queue.

So, how do you find the particular commands you need? One way is to use az find, the AI robot that uses the Azure documentation to tell you more about commands, the CLI, and more.

Example: find the most popular commands related to the word blob.
	
```bash
az find blob
```       
	
If you already know the name of the command you want, the --help argument for that command will get you more detailed information on the command, and a list of the available subcommands for a command group. So, with our storage example, here's how you can get a list of the subgroups and commands for managing blob storage:
	
```bash
az storage blob --help
```       
### Connect
Since you're working with a local install of the Azure CLI, you'll need to authenticate before you can execute Azure commands by using the Azure CLI login command.

```bash
 az login
```

After a successful sign-in, you'll be connected to your Azure subscription.

### Create
You'll often need to create a new resource group before you create a new Azure service, so we'll use resource groups as an example to show how to create Azure resources from the CLI.

The Azure CLI group create command creates a resource group. You must specify a name and location. The name must be unique within your subscription. The location determines where the metadata for your resource group will be stored. You use strings like "West US", "North Europe", or "West India" to specify the location; alternatively, you can use single-word equivalents, such as westus, northeurope, or westindia. The core syntax is:

```bash
az group create --name <name> --location <location>
```    
	
### Verify
For many Azure resources, the Azure CLI provides a list subcommand to view resource details. For example, the Azure CLI group list command lists your Azure resource groups. This is useful to verify whether the resource group was successfully created:

```bash
az group list
```       
	
To get a more concise view, you can format the output as a simple table:

```bash
az group list --output table       
```  
	
<hr>

 **Single sign-on (SSO) access**
 - Azure AD provides secure single sign-on (SSO) to web apps on the cloud and to on-premises apps. Users can sign in with the same set of credentials to access all their apps.
 
  **Ubiquitous device support**
  
  - Azure AD works with iOS, macOS, Android, and Windows devices, and offers a common experience across the devices. Users can launch apps from a personalized web-based access panel, mobile app, Microsoft 365, or custom company portals by using their existing work credentials.
  
  **Secure remote access**	
  
  - Azure AD enables secure remote access for on-premises web apps. Secure access can include multifactor authentication (MFA), conditional access policies, and group-based access management. Users can access on-premises web apps from everywhere, including from the same portal.
  
  **Cloud extensibility**	
  
  - Azure AD can extend to the cloud to help you manage a consistent set of users, groups, passwords, and devices across environments.
  
  **Sensitive data protection**	
  
  - Azure AD offers unique identity protection capabilities to secure your sensitive data and apps. Admins can monitor for suspicious sign-in activity and potential vulnerabilities in a consolidated view of users and resources in the directory.
  
  **Self-service support**
  
  - Azure AD lets you delegate tasks to company employees that might otherwise be completed by admins with higher access privileges. Providing self-service app access and password management through verification steps can reduce helpdesk calls and enhance security.

# Things to consider when using Azure AD features
Azure AD offers many features and benefits. Consider which features can be used to best support your corporate scenarios.

Consider enabling single sign-on access. Enable SSO access for your users to connect to the cloud or use on-premises apps. Azure AD SSO supports Microsoft 365 and thousands of SaaS apps, such as Salesforce, Workday, DocuSign, ServiceNow, and Box.

Consider UX and device support. Build a consistent user experience that works for all devices and directory access points. You can design custom company portals and personalized web-based access for your employees that lets them connect with their existing work credentials.

Consider benefits of secure remote access. Protect your on-premises web apps by implementing secure remote access with MFA and access policies.

Consider advantages of cloud extensibility. Connect Active Directory and other on-premises directories in the cloud to Azure AD in just a few steps. You can make it easier for your admins to manage the same users, groups, passwords, and devices across all supported environments.

Consider advanced protection for sensitive data. Enhance the security of your sensitive data and apps by using the built-in protection features of Azure AD. Your admins can utilize advanced security reports, notifications, remediation recommendations, and risk-based policies.

Consider reduced costs, self-service options. Take advantage of the Azure AD self-service features to help reduce costs for your organization. Delegate certain tasks like resetting passwords, or creating and managing groups to your non-admin users.

  | Azure AD concept | Description |
  | ---------------- | ----------- |
  | `Identity`	| An identity is an object that can be authenticated. The identity can be a user with a username and password. Identities can also be applications or other servers that require authentication by using secret keys or certificates. Azure AD is the underlying product that provides the identity service.
  | `Account`  | An account is an identity that has data associated with it. To have an account, you must first have a valid identity. You can't have an account without an identity.
  | `Azure AD account`	| An Azure AD account is an identity that's created through Azure AD or another Microsoft cloud service, such as Microsoft 365. Identities are stored in Azure AD and are accessible to your organization's cloud service subscriptions. The Azure AD account is also called a work or school account.
  | `Azure tenant (directory)` | An Azure tenant is a single dedicated and trusted instance of Azure AD. Each tenant (also called a directory) represents a single organization. When your organization signs up for a Microsoft cloud service subscription, a new tenant is automatically created. Because each tenant is a dedicated and trusted instance of Azure AD, you can create multiple tenants or instances.
  | `Azure subscription` | An Azure subscription is used to pay for Azure cloud services. A subscription is linked to a credit card. Each subscription is joined to a single tenant. You can have multiple subscriptions. |
# Compare Active Directory Domain Services to Azure Active Directory
	
Active Directory Domain Services (AD DS) is the traditional deployment of Windows Server-based Active Directory on a physical or virtual server. AD DS is commonly considered to be primarily a directory service, but it's only one component of the Windows Active Directory suite of technologies. The suite also includes Active Directory Certificate Services (AD CS), Active Directory Lightweight Directory Services (AD LS), Active Directory Federation Services (AD FS), and Active Directory Rights Management Services (AD RMS).	
As you plan your identity strategy, consider the following characteristics that distinguish Azure AD from AD DS.

- **Identity solution**: AD DS is primarily a directory service, while Azure AD is a full identity solution. Azure AD is designed for internet-based applications that use HTTP and HTTPS communications. The features and capabilities of Azure AD support target strong identity management.

- **REST API queries**: Azure AD is based on HTTP and HTTPS protocols. Azure AD tenants can't be queried by using LDAP. Azure AD uses the REST API over HTTP and HTTPS.

- **Communication protocols**: Because Azure AD is based on HTTP and HTTPS, it doesn't use Kerberos authentication. Azure AD implements HTTP and HTTPS protocols, such as SAML, WS-Federation, and OpenID Connect for authentication (and OAuth for authorization).

- **Federation services**: Azure AD includes federation services, and many third-party services like Facebook.

- **Flat structure**: Azure AD users and groups are created in a flat structure. There are no organizational units (OUs) or group policy objects (GPOs).

- **Managed service**: Azure AD is a managed service. You manage only users, groups, and policies. If you deploy AD DS with virtual machines by using Azure, you manage many other tasks, including deployment, configuration, virtual machines, patching, and other backend processes.

### Things to consider when using joined devices
Your organization is interested in using joined devices in their management strategy. As you plan for how to implement the feature, review these configuration points:

Consider connection options. Connect your device to Azure AD in one of two ways:

Register your device to Azure AD so you can manage the device identity. Azure AD device registration provides the device with an identity that's used to authenticate the device when a user signs into Azure AD. You can use the identity to enable or disable the device.

Join your device, which is an extension of registering a device. Joining provides the benefits of registering, and also changes the local state of the device. Changing the local state enables your users to sign into a device by using an organizational work or school account instead of a personal account.

Consider combining registration with other solutions. Combine registration with a mobile device management (MDM) solution like Microsoft Intune, to provide other device attributes in Azure AD. You can create conditional access rules that enforce access from devices to meet organization standards for security and compliance.

Consider other implementation scenarios. Although AD Join is intended for organizations that don't have an on-premises Windows Server Active Directory infrastructure, it can be used for other scenarios like branch offices.

- SSPR requires an Azure AD account with Global Administrator privileges to manage SSPR options. This account can always reset their own passwords, no matter what options are configured.

- SSPR uses a security group to limit the users who have SSPR privileges.

- All user accounts in your organization must have a valid license to use SSPR.

- Consider who can reset their passwords. Decide which users in your organization should be enabled to use the feature. In the Azure portal, there are three options for the SSPR feature: None, Selected, and All.

The Selected option is useful for creating specific groups who have SSPR enabled. You can create groups for testing or proof of concept before applying the feature to a larger group. When you're ready to deploy SSPR to all user accounts in your Azure AD tenant, you can change the setting.

- Consider your authentication methods. Determine how many authentication methods are required to reset a password, and select the authentication options for users.

  - Your system must require at least one authentication method to reset a password.

  - A strong SSPR plan offers multiple authentication methods for the user. Options include email notification, text message, or a security code sent to the user's mobile or office phone. You can also offer the user a set of security questions.

  - You can require security questions to be registered for the users in your Azure AD tenant.

  - You can configure how many correctly answered security questions are required for a successful password reset.

  - Consider combining methods for stronger security. Security questions can be less secure than other authentication methods. Some users might know the answers for a particular user's questions, or the questions might be easy to solve. If you support security questions, combine this option with other authentication methods.
	
Describes Azure Active Directory?
   - Azure AD is primarily an identity solution. It's designed for internet-based applications by using HTTP and HTTPS communications.
	
What term defines a dedicated and trusted instance of Azure Active Directory?
   - A tenant is a dedicated and trusted instance of Azure AD. A tenant is automatically created when an organization signs up for a Microsoft cloud service subscription.
	
Your users want to sign-in to devices, apps, and services from anywhere. Users want to sign-in by using an organizational work or school account instead of a personal account. What should you do first?
   - Joining the device provides the features you need.

# Create user accounts

Every user who wants access to Azure resources needs an Azure user account. A user account has all the information required to authenticate the user during the sign-in process. Azure Active Directory (Azure AD) supports three types of user accounts. The types indicate where the user is defined (in the cloud or on-premises), and whether the user is internal or external to your Azure AD organization.

Things to know about user accounts
The following table describes the user accounts supported in Azure AD. As you review these options, consider what types of user accounts suit your organization.
|User Account | Description |
|------------ | ------------|
| Cloud identity | 	A user account with a *cloud identity* is defined only in Azure AD. This type of user account includes administrator accounts and users who are managed as part of your organization. A cloud identity can be for user accounts defined in your Azure AD organization, and also for user accounts defined in an external Azure AD instance. When a cloud identity is removed from the primary directory, the user account is deleted.
| Directory-synchronized identity | User accounts that have a *directory-synchronized identity* are defined in an on-premises Active Directory. A synchronization activity occurs via Azure AD Connect to bring these user accounts in to Azure. The source for these accounts is Windows Server Active Directory.
| Guest user | *Guest user* accounts are defined outside Azure. Examples include user accounts from other cloud providers, and Microsoft accounts like an Xbox LIVE account. The source for guest user accounts is Invited user. Guest user accounts are useful when external vendors or contractors need access to your Azure resources. |
	
### Things to consider when choosing user accounts
- **Consider where users are defined.** Determine where your users are defined. Are all your users defined within your Azure AD organization, or are some users defined in external Azure AD instances? Do you have users who are external to your organization? It's common for businesses to support two or more account types in their infrastructure.

- **Consider support for external contributors.** Allow external contributors to access Azure resources in your organization by supporting the Guest user account type. When the external contributor no longer requires access, you can remove the user account and their access privileges.

- **Consider a combination of user accounts.** Implement the user account types that enable your organization to satisfy their business requirements. Support directory-synchronized identity user accounts for users defined in Windows Server Active Directory. Support cloud identities for users defined in your internal Azure AD structure or for user defined in an external Azure AD instance.
# Manage User Accounts 
A new user account must have a display name and an associated user account name. An example display name is `Aran Sawyer-Miller` and the associated user account name could be `asawmill@contoso.com.`

Information and settings that describe a user are stored in the user account profile.

The profile can have other settings like a user's job title, and their contact email address.

A user with Global administrator or User administrator privileges can preset profile data in user accounts, such as the main phone number for the company.

Non-admin users can set some of their own profile data, but they can't change their display name or account name.

### Things to consider when managing cloud identity accounts
There are several points to consider about managing user accounts. As you review this list, consider how you can add cloud identity user accounts for your organization.

- **Consider user profile data**. Allow users to set their profile information for their accounts, as needed. User profile data, including the user's picture, job, and contact information is optional. You can also supply certain profile settings for each user based on your organization's requirements.

- **Consider restore options for deleted accounts**. Include restore scenarios in your account management plan. Restore operations for a deleted account are available up to 30 days after an account is removed. After 30 days, a deleted user account can't be restored.

- **Consider gathered account data** . Collect sign-in and audit log information for user accounts. Azure AD lets you gather this data to help you analyze and improve your infrastructure.
	
- **Consider naming conventions** . Establish or implement a naming convention for your user accounts. Apply conventions to user account names, display names, and user aliases for consistency across the organization. Conventions for names and aliases can simplify the bulk create process by reducing areas of uniqueness in the CSV file. A convention for user names could begin with the user's last name followed by a period, and end with the user's first name, as in Sawyer-Miller.Aran@contoso.com.

- **Consider using initial passwords.** Implement a convention for the initial password of a newly created user. Design a system to notify new users about their passwords in a secure way. You might generate a random password and email it to the new user or their manager.

- **Consider strategies for minimizing errors.** View and address any errors, by downloading the results file on the Bulk operation results page in the Azure portal. The results file contains the reason for each error. An error might be a user account that's already been created or an account that's duplicated. Generally, it's easier to upload and troubleshoot smaller groups of user accounts.

# Create group accounts

Azure Active Directory (Azure AD) allows your organization to define two different types of group accounts. 
- *Security groups* are used to manage member and computer access to shared resources for a group of users. You can create a security group for a specific security policy and apply the same permissions to all members of a group.
- *Microsoft 365*  groups provide collaboration opportunities. Group members have access to a shared mailbox, calendar, files, SharePoint site, and more.

  - Use security groups to set permissions for all group members at the same time, rather than adding permissions to each member individually.

  - Add Microsoft 365 groups to enable group access for guest users outside your Azure AD organization.

  - Security groups can be implemented only by an Azure AD administrator.
  
  - Normal users and Azure AD admins can both use Microsoft 365 groups.

|Access rights | Description |
|--------------| ------------|
| Assigned | Add specific users as members of a group, where each user can have unique permissions.
| Dynamic user | Use dynamic membership rules to automatically add and remove group members. When member attributes change, Azure reviews the dynamic group rules for the directory. If the member attributes meet the rule requirements, the member is added to the group. If the member attributes no longer meet the rule requirements, the member is removed.
| Dynamic device | (Security groups only) Apply dynamic group rules to automatically add and remove devices in security groups. When device attributes change, Azure reviews the dynamic group rules for the directory. If the device attributes meet the rule requirements, the device is added to the security group. If the device attributes no longer meet the rule requirements, the device is removed.|

### Things to think about administrative units
Consider how a central admin role can use administrative units to support the Engineering department in our scenario:

Create a role that has administrative permissions for only Azure AD users in the Engineering department administrative unit.

Create an administrative unit for the Engineering department.

Populate the administrative unit with only the Engineering department students, staff, and resources.

Add the Engineering department IT team to the role, along with its scope.

### Things to consider when working with administrative units
Think about how you can implement administrative units in your organization. Here are some considerations:

- Consider management tools. Review your options for managing AUs. You can use the Azure portal, PowerShell cmdlets and scripts, or Microsoft Graph.

- Consider role requirements in the Azure portal. Plan your strategy for administrative units according to role privileges. In the Azure portal, only the Global Administrator or Privileged Role Administrator users can manage AUs.

- Consider scope of administrative units. Recognize that the scope of an administrative unit applies only to management permissions. Members and admins of an administrative unit can exercise their default user permissions to browse other users, groups, or resources outside of their administrative unit.
	
What type of user account allows an external organization to access your resources?	
- A guest user account for each member of the external team. 
  -  A guest user account restricts users to just the access they need.
	
What kind of group account can you create so you can apply the same permissions to all group members?	
- Security group
  - You can create a security group for a specific security policy and apply the same permissions to all members of the group.
	
Which Azure AD role enables a user to manage all groups in your Teams tenants, and also assign other admin roles?
- Global administrator
  - The Global Administrator role manages all aspects of Azure AD and Microsoft services that use Azure AD identities. This role can manage groups across tenants and assign other administrator roles.

# Identify Azure regions

Microsoft Azure is made up of datacenters located around the globe. These datacenters are organized and made available to end users by region. A region is a geographical area on the planet containing at least one, but potentially multiple datacenters. The datacenters are in close proximity and networked together with a low-latency network. A few examples of regions are West US, Canada Central, West Europe, Australia East, and Japan West.

### Things to know about regions
Here are some points to consider about regions:

- Azure is generally available in more than 60 regions in 140 countries.

- Azure has more global regions than any other cloud provider.

- Regions provide you with the flexibility and scale needed to bring applications closer to your users.

- Regions preserve data residency and offer comprehensive compliance and resiliency options for customers.

|Characteristic | Description |
|---------------| ------------|
|Physical isolation | Azure prefers at least 300 miles of separation between datacenters in a regional pair. This principle isn't practical or possible in all geographies. Physical datacenter separation reduces the likelihood of natural disasters, civil unrest, power outages, or physical network outages affecting both regions at once.
|Platform-provided replication | Some services like Geo-Redundant Storage provide automatic replication to the paired region.
|Region recovery order | During a broad outage, recovery of one region is prioritized out of every pair. Applications that are deployed across paired regions are guaranteed to have one of the regions recovered with priority.
|Sequential updates | Planned Azure system updates are rolled out to paired regions sequentially (not at the same time). Rolling updates minimizes downtime, reduces bugs, and logical failures in the rare event of a bad update.
|Data residency | Regions reside within the same geography as their enabled set (except for the Brazil South and Singapore regions).|

### Things to consider when using regions and regional pairs

- Consider resource and region deployment. Plan the regions where you want to deploy your resources. For most Azure services, when you deploy a resource in Azure, you choose the region where you want your resource to be deployed.

- Consider service support by region. Research region and service availability. Some services or Azure Virtual Machines features are available only in certain regions, such as specific Virtual Machines sizes or storage types.

- Consider services that don't require regions. Identify services that don't need region support. Some global Azure services that don't require you to select a region. These services include Azure Active Directory, Microsoft Azure Traffic Manager, and Azure DNS.

- Consider exceptions to region pairing. Check the Azure website for current region availability and exceptions. If you plan to support the Brazil South region, note this region is paired with a region outside its geography. The Singapore region also has an exception to standard regional pairing.

- Consider benefits of data residency. Take advantage of the benefits of data residency offered by regional pairs. This feature can help you meet requirements for tax and law enforcement jurisdiction purposes.

### Things to know about subscriptions
- As you think about the subscriptions to implement for your company, consider the following points:

- Every Azure cloud service belongs to a subscription.

- Each subscription can have a different billing and payment configuration.

- Multiple subscriptions can be linked to the same Azure account.

- More than one Azure account can be linked to the same subscription.

- Billing for Azure services is done on a per-subscription basis.

- If your Azure account is the only account associated with a subscription, you're responsible for the billing requirements.

- Programmatic operations for a cloud service might require a subscription ID.

### Things to consider when using subscriptions
- Consider how many subscriptions your organization needs to support the business scenarios. As you plan, think about how you can organize your resources into resource groups.

- Consider the types of Azure accounts required. Determine the types of Azure accounts your users will link with Azure subscriptions. You can use an Azure AD account or a directory that's trusted by Azure AD like a work or school account. If you don't belong to one of these organizations, you can sign up for an Azure account by using your Microsoft Account, which is also trusted by Azure AD.

- Consider multiple subscriptions. Set up different subscriptions and payment options according to your company's departments, projects, regional offices, and so on. A user can have more than one subscription linked to their Azure account, where each subscription pertains to resources, access privileges, limits, and billing for a specific project.

- Consider a dedicated shared services subscription. Plan for how users can share resources allocated in a single subscription. Use a shared services subscription to ensure all common network resources are billed together and isolated from other workloads. Examples of shared services subscriptions include Azure ExpressRoute and Virtual WAN.

- Consider access to resources. Every Azure subscription can be associated with an Azure AD. Users and services authenticate with Azure AD before they access resources.

Things to know about Microsoft Cost Management
Your organization is interested in the benefits of using Microsoft Cost Management to monitor their subscription billing and resource usage. As you plan for your implementation, review the following product characteristics and features:

- Microsoft Cost Management shows organizational cost and usage patterns with advanced analytics. Costs are based on negotiated prices and factor in reservation and Azure Hybrid Benefit discounts. Predictive analytics are also available.

- Reports in Microsoft Cost Management show the usage-based costs consumed by Azure services and third-party Marketplace offerings. Collectively, the reports show your internal and external costs for usage and Azure Marketplace charges. The reports help you understand your spending and resource use, and can help find spending anomalies. Charges, such as reservation purchases, support, and taxes might not be visible in reports.

- The product uses Azure management groups, budgets, and recommendations to show clearly how your expenses are organized and how you might reduce costs.

- You can use the Azure portal or various APIs for export automation to integrate cost data with external systems and processes. Automated billing data export and scheduled reports are also available.

### Things to consider when using Microsoft Cost Management
- Microsoft Cost Management can help you plan for and control your organization costs. Consider how the product features can be implemented to support your business scenarios:

- Consider cost analysis. Take advantage of Microsoft Cost Management cost analysis features to explore and analyze your organizational costs. You can view aggregated costs by organization to understand where costs are accrued, and to identify spending trends. Monitor accumulated costs over time to estimate monthly, quarterly, or even yearly cost trends against a budget.

- Consider budget options. Use Microsoft Cost Management features to establish and maintain budgets. The product helps you plan for and meet financial accountability in your organization. Budgets help prevent cost thresholds or limits from being surpassed. You can utilize analysis data to inform others about their spending to proactively manage costs. The budget features help you see how company spending progresses over time.

  - Consider recommendations. Review the Microsoft Cost Management recommendations to learn how you can optimize and improve efficiency by identifying idle and underutilized resources. Recommendations can reveal less expensive resource options. When you act on the recommendations, you change the way you use your resources to save money. Using recommendations is an easy process:

  1. View cost optimization recommendations to see potential usage inefficiencies.
  2. Act on a recommendation to modify your Azure resource use and implement a more cost-effective option.
  3. Verify the new action to make sure the change has the desired effect.
- Consider exporting cost management data. Microsoft Cost Management helps you work with your billing information. If you use external systems to access or review cost management data, you can easily export the data from Azure.

  - Set a daily scheduled export in comma-separated-value (CSV) format and store the data files in Azure storage.
  - Access your exported data from your external system.


### Things to know about resource tags
As you plan your Azure subscriptions, resources, and services, review these characteristics of Azure resource tags:

- Each resource tag has a name and a value.

- The tag name remains constant for all resources that have the tag applied.

- The tag value can be selected from a defined set of values, or unique for a specific resource instance.

- A resource or resource group can have a maximum of 50 tag name/value pairs.

- Tags applied to a resource group aren't inherited by the resources in the resource group.

### Things to consider when using resource tags
Here are a few things you can do with resource tags:

- Consider searching on tag data. Search for resources in your subscription by querying on the tag name and value.

- Consider finding related resources. Retrieve related resources from other resource groups by searching on the tag name or value.

- Consider grouping billing data. Group resources like virtual machines by cost center and production environment. When you download the resource usage comma-separated values (CSV) file for your services, the tags appear in the `Tags` column.

- Consider creating tags with PowerShell or the Azure CLI. Create many resource tags programatically by using Azure PowerShell or the Azure CLI.
	
|Cost saving | Description|
|------------| -----------|
|Reservations| Save money by paying ahead. You can pay for one year or three years of virtual machine, SQL Database compute capacity, Azure Cosmos DB throughput, or other Azure resources. Pre-paying allows you to get a discount on the resources you use. Reservations can significantly reduce your virtual machine, SQL database compute, Azure Cosmos DB, or other resource costs up to 72% on pay-as-you-go prices. Reservations provide a billing discount and don't affect the runtime state of your resources.
|Azure Hybrid Benefits | Access pricing benefits if you have a license that includes Software Assurance. Azure Hybrid Benefits helps maximize the value of existing on-premises Windows Server or SQL Server license investments when migrating to Azure. There's an Azure Hybrid Benefit Savings Calculator to help you determine your savings.
|Azure Credits | Use the monthly credit benefit to develop, test, and experiment with new solutions on Azure. As a Visual Studio subscriber, you could use Microsoft Azure at no extra charge. With your monthly Azure credit, Azure is your personal sandbox for development and testing.
|Azure regions | Compare pricing across regions. Pricing can vary from one region to another, even in the US. Double check the pricing in various regions to see if you can save by selecting a different region for your subscription.
|Budgets | Apply the budgeting features in Microsoft Cost Management to help plan and drive organizational accountability. With budgets, you can account for the Azure services you consume or subscribe to during a specific period. Monitor spending over time and inform others about their spending to proactively manage costs. Use budgets to compare and track spending as you analyze costs.
|Pricing Calculator | The [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) provides estimates in all areas of Azure, including compute, networking, storage, web, and databases.|
	
The company financial controller wants to be notified whenever the company is half-way to spending the money allocated for cloud services. Which approach supports this request?
	
- Create a budget and a spending threshold.
  - Create a budget and a spending threshold. Billing Alerts help your monitor and manage billing activity for your Azure accounts. Budget thresholds can be evaluated and are reset automatically at the end of a period.
	
The company financial controller wants to identify which billing department each Azure resource belongs to. Which approach enables this requirement?
	
- Apply a tag to each resource that includes the associated billing department. 
  - Tags provide extra information, or metadata, about your resources. The team might create a tag named `BillingDept`, where the value is the name of the billing department. Azure Policy ensures that the proper tags are assigned when resources are provisioned.

Which option preserves data residency, and offers comprehensive compliance and resiliency options? 
	
 - Regions preserve data residency, and offer comprehensive compliance and resiliency options for customers.

# Create management groups

Organizations that use multiple subscriptions need a way to efficiently manage access, policies, and compliance. Azure management groups provide a level of scope and control above your subscriptions. You can use management groups as containers to manage access, policy, and compliance across your subscriptions.

Things to know about management groups
Consider the following characteristics of Azure management groups:

- By default, all new subscriptions are placed under the top-level management group, or root group.

- All subscriptions within a management group automatically inherit the conditions applied to that management group.

- A management group tree can support up to six levels of depth.

- Azure role-based access control authorization for management group operations isn't enabled by default.

### Things to consider when using management groups


- Consider custom hierarchies and groups. Align your Azure subscriptions by using custom hierarchies and grouping that meet your company's organizational structure and business scenarios. You can use management groups to target policies and spending budgets across subscriptions.

- Consider policy inheritance. Control the hierarchical inheritance of access and privileges in policy definitions. All subscriptions within a management group inherit the conditions applied to the management group. You can apply policies to a management group to limit the regions available for creating virtual machines (VMs). The policy can be applied to all management groups, subscriptions, and resources under the initial management group, to ensure VMs are created only in the specified regions.

- Consider compliance rules. Organize your subscriptions into management groups to help meet compliance rules for individual departments and teams.

- Consider cost reporting. Use management groups to do cost reporting by department or for specific business scenarios. You can use management groups to report on budget details across subscriptions.

  1. The ID value can't be changed after it's created because it's used throughout the Azure system to identify the management group. The display name for the management group is optional and can be changed at any time.
  
# Implement Azure policies

Azure Policy is a service in Azure that you can use to create, assign, and manage policies. You can use policies to enforce rules on your resources to meet corporate compliance standards and service level agreements. Azure Policy runs evaluations and scans on your resources to make sure they're compliant.

### Things to know about Azure Policy
The main advantages of Azure Policy are in the areas of enforcement and compliance, scaling, and remediation. Azure Policy is also important for teams that run an environment that requires different forms of governance.

|Advantage | Description|
|--------| ------------------------------|
|Enforce rules and compliance |	Enable built-in policies, or build custom policies for all resource types. Support real-time policy evaluation and enforcement, and periodic or on-demand compliance evaluation.
|Apply policies at scale | Apply policies to a management group with control across your entire organization. Apply multiple policies and aggregate policy states with policy initiative. Define an exclusion scope.
|Perform remediation | Conduct real-time remediation, and remediation on your existing resources.
|Exercise governance | Implement governance tasks for your environment: Support multiple engineering teams (deploying to and operating in the environment) Manage multiple subscriptions Standardize and enforce how cloud resources are configured Manage regulatory compliance, cost control, security, and design consistency|

### Things to consider when using Azure Policy

- Consider deployable resources. Specify the resource types that your organization can deploy by using Azure Policy. You can specify the set of virtual machine SKUs that your organization can deploy.

- Consider location restrictions. Restrict the locations your users can specify when deploying resources. You can choose the geographic locations or regions that are available to your organization.

- Consider rules enforcement. Enforce compliance rules and configuration options to help manage your resources and user options. You can enforce a required tag on resources and define the allowed values.

- Consider inventory audits. Use Azure Policy with Azure Backup service on your VMs and run inventory audits.

# Access built-in policy definitions
You can sort the [list of built-in definitions](https://learn.microsoft.com/en-us/azure/governance/policy/samples/built-in-policies) by category to search for policies that meet your business needs.

- Allowed virtual machine size SKUs: Specify a set of VM size SKUs that your organization can deploy. This policy is located under the Compute category.

- Allowed locations: Restrict the locations users can specify when deploying resources. Use this policy to enforce your geo-compliance requirements. This policy is located under the General category.

- Configure Azure Device Update for IoT Hub accounts to disable public network access: Disable public network access for your Device Update for IoT Hub resources. This policy is located under the Internet of Things category
	
# Use a built-in initiative definition
You can create your own initiative definitions, or use built-in definitions in Azure Policy. You can sort the [list of built-in initiatives](https://learn.microsoft.com/en-us/azure/governance/policy/samples/built-in-initiatives) by category to search for definitions for your organization.

Here are some examples of built-in initiative definitions:

Audit machines with insecure password security settings: Use this initiative to deploy an audit policy to specified resources in your organization. The definition evaluates the resources to check for insecure password security settings. This initiative is located under the Guest Configuration category.

Configure Windows machines to run Azure Monitor Agent and associate them to a Data Collection Rule: Use this initiative to monitor and secure your Windows VMs, Virtual Machine Scale Sets, and Arc machines. The definition deploys the Azure Monitor Agent extension and associates the resources with a specified Data Collection Rule. This initiative is located under the Monitoring category.

ISO 27001:2013: Use this initiative to apply policies for a subset of ISO 27001:2013 controls. This initiative is located under the Regulatory Compliance category.

There are several Azure policies that need to be applied to a new branch office. What's the best approach? 
	
- Create a policy initiative
  - A policy initiative is a set of policy definitions that could be applied to the new branch office.

To satisfy the finance team's request for billing by department, multiple resource groups have been created and the resource tags applied. What's the next step? 
	
- Create an Azure policy 
  -An Azure policy requires that a resource tag is applied before the resource is created.

How can you ensure that only cost-effective virtual machine SKU sizes are deployed?

 - Create a policy in Azure Policy that specifies the allowed SKU sizes
  - There's a built-in Azure policy to specify the allowed virtual machine SKU sizes. After the policy is enabled, it's applied whenever a virtual machine is created or resized.

 Which option can you use to manage governance across multiple Azure subscriptions?
	
- Management groups facilitate the hierarchical ordering of Azure resources into collections, at a level of scope above subscriptions. Distinct governance conditions can be applied to each management group, with Azure Policy and Azure role-based access controls, to manage Azure subscriptions effectively. The resources and subscriptions assigned to a management group automatically inherit the conditions applied to the management group.
	
# Implement role-based access control
	
### Things to know about Azure RBAC

- Allow an application to access all resources in a resource group.

- Allow one user to manage VMs in a subscription, and allow another user to manage virtual networks.

- Allow a database administrator (DBA) group to manage SQL databases in a subscription.

- Allow a user to manage all resources in a resource group, such as VMs, websites, and subnets.
	
| Concept | Description | Examples |
|---------| -------------------|----------|
Security principal | An object that represents something that requests access to resources. | User, group, service principal, managed identity
Role definition | A set of permissions that lists the allowed operations. Azure RBAC comes with built-in role definitions, but you can also create your own custom role definitions. |	Some built-in role definitions: *Reader, Contributor, Owner, User Access Administrator*
Scope |	The boundary for the requested *level* of access, or "how much" access is granted. | Root, management group, subscription, resource group, resource
Assignment | An assignment attaches a role definition to a security principal at a particular scope. Users can grant the access described in a role definition by creating (attaching) an assignment for the role. | Assign the *User Access Administrator* role to an admin group scoped to a management group Assign the *Contributor* role to a user scoped to a subscription|

### Things to consider when using Azure RBAC

- Consider your requestors. Plan your strategy to accommodate for all types of access to your resources. Security principals are created for anything that requests access to your resources. Determine who are the requestors in your organization. Requestors can be internal or external users, groups of users, applications and services, resources, and so on.

- Consider your roles. Examine the types of job responsibilities and work scenarios in your organization. Roles are commonly built around the requirements to fulfill job tasks or complete work goals. Certain users like administrators, corporate controllers, and engineers can require a level of access beyond what most users need. Some roles can be defined to provide the same access for all members of a team or department for specific resources or applications.

- Consider scope of permissions. Think about how you can ensure security by controlling the scope of permissions for role assignments. Outline the types of permissions and levels of scope that you need to support. You can apply different scope levels for a single role to support requestors in different scenarios.

- Consider built-in or custom definitions. Review the built-in role definitions in Azure RBAC. Built-in roles can be used as-is, or adjusted to meet the specific requirements for your organization. You can also create custom role definitions from scratch.
	
# Create a role definition
	
A role definition consists of sets of permissions that are defined in a JSON file. Each permission set has a name, such as Actions or NotActions that describes the purpose of the permissions. Some examples of permission sets include:

- Actions permissions identify what actions are allowed.

- NotActions permissions specify what actions aren't allowed.

- DataActions permissions indicate how data can be changed or used.

- AssignableScopes permissions list the scopes where a role definition can be assigned.

The Actions permissions show the Contributor role has all action privileges. The asterisk `"*"` wildcard means "all." The NotActions permissions narrow the privileges provided by the Actions set, and deny three actions:

- `Authorization/*/Delete`: Not authorized to delete or remove for "all."
- `Authorization/*/Write`: Not authorized to write or change for "all."
- `Authorization/elevateAccess/Action`: Not authorized to increase the level or scope of access privileges.
	
The Contributor role also has two DataActions permissions to specify how data can be affected:

- `"NotDataActions": []`: No specific actions are listed. Therefore, all actions can affect the data.
- `"AssignableScopes": ["/"]`: The role can be assigned for all scopes that affect data.
	
```powershell
Name: Owner
ID: 01010101-2323-4545-6767-987453021523
IsCustom: False
Description: Manage everything, including access to resources
Actions: {*}             # All actions allowed
NotActions: {}           # No actions denied
AssignableScopes: {/}    # Role can be assigned to all scopes
```
Things to know about role definitions

- Azure RBAC provides built-in roles and permissions sets. You can also create custom roles and permissions.

- The Owner built-in role has the highest level of access privilege in Azure.

- The system subtracts NotActions permissions from Actions permissions to determine the effective permissions for a role.

- The AssignableScopes permissions for a role can be management groups, subscriptions, resource groups, or resources.

### Role permissions

Use the Actions and NotActions permissions together to grant and deny the exact privileges for each role. The Actions permissions can provide the breadth of access and the NotActions permissions can narrow the access.

The following table shows how the Actions or NotActions permissions are used in the definitions for three built-in roles: Owner, Contributor, and Reader. The permissions are narrowed from the Owner role to the Contributor and Reader roles to limit access.

|Role name | Description | Actions permissions| NotActions permissions|
|----------|-------------|--------------------|-----------------------|	
|*Owner*|	Allow all actions |	`*` |	n/a
|*Contributor*| Allow all actions, except write or delete role assignment| `*`| `Microsoft.Authorization/*/Delete` `Microsoft.Authorization/*/Write` `Microsoft.Authorization/elevateAccess/Action`
|*Reader*|Allow all read actions|	`/*/read`| n/a
	
### Things to consider when creating roles

- Consider using built-in roles. Review the list of [built-in role definitions](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles) in Azure RBAC. There are over 100 pre-defined role definitions to choose from, such as Owner, Backup Operator, Website Contributor, and SQL Security Manager. Built-in roles are defined for several categories of services, tasks, and users, including General, Networking, Storage, Databases, and more.

- Consider creating custom definitions. Define your own [custom roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles) to meet specific business scenarios for your organization. You can modify the permissions for a built-in role to meet the specific requirements for your organization. You can also create custom role definitions from scratch.

 -Consider limiting access scope. Assign your roles with the minimum level of scope required to perform the job duties. Some users like administrators require full access to corporate resources to maintain the infrastructure. Other users in the organization can require write access to personal or team resource, and read-only access to shared company resources.

- Consider controlling changes to data. Identify data or resources that should only be modified in specific scenarios and apply tight access control. Limit users to the least of amount of access they need to get their work done. A well-planned access management strategy helps to maintain your infrastructure and prevent security issues.

- Consider applying deny assignments. Determine if you need to implement the deny assignment feature. Similar to a role assignment, a deny assignment attaches a set of deny actions to a user, group, or service principal at a particular scope for the purpose of denying access. Deny assignments block users from performing specific Azure resource actions even if a role assignment grants them access.
	
|                | Azure RBAC roles| Azure AD admin roles|
|----------------|------------------|---------------------|
|Access management| Manages access to Azure resources| Manages access to Azure AD resources
Scope assignment| Scope can be specified at multiple levels, including management groups, subscriptions, resource groups, and resources| Scope is specified at the tenant level
Role definitions| Roles can be defined via the Azure portal, the Azure CLI, Azure PowerShell, Azure Resource Manager templates, and the REST API| Roles can be defined via the Azure admin portal, Microsoft 365 admin portal, and Microsoft Graph Azure AD PowerShell

# Apply role-based access control

Built-in role definitions in Azure RBAC are defined for several categories of services, tasks, and users. You can assign built-in roles at different scopes to support various scenarios, and build custom roles from the base definitions.

Azure Active Directory (Azure AD) also provides built-in roles to manage resources in Azure AD, including users, groups, and domains. Azure AD offers [administrator roles](https://learn.microsoft.com/en-us/azure/active-directory/roles/permissions-reference) that you can implement for your organization, such as Global admin, Application admin, and Application developer.
	
- **Azure AD admin roles** are used to manage resources in Azure AD, such as users, groups, and domains. These roles are defined for the Azure AD tenant at the root level of the configuration.

- **Azure RBAC roles** provide more granular access management for Azure resources. These roles are defined for a requestor or resource and can be applied at multiple levels: the root, management groups, subscriptions, resource groups, or resources.
	
You have three virtual machines (VM1, VM2, VM3) in a resource group. A new admin is hired, and they need to be able to modify settings on VM3. They shouldn't be able to make changes to VM1 or VM2. How can you implement RBAC to minimize administrative overhead?

- Assign the admin to the Contributor role on VM3. 
  - When you assign the Contributor role to the specific resource, the admin can change the settings on that resource; in this case, VM3.
	
Explain the main differences between Azure roles and Azure Active Directory (Azure AD) roles.

- Azure roles apply to Azure resources. Azure AD roles apply to Azure AD resources such as users, groups, and domains.
  - Azure roles are used to manage access to VMs, storage, and other Azure resources. Azure AD roles are used to manage access to Azure AD resources like user accounts and passwords.
	
What's included in a custom Azure role definition?

- Operations allowed for Azure resources, and scope of permissions
  - A custom role definition includes the allowed operations, such as read, write, and delete for Azure resources. The custom role definition also includes the scope of these permissions.
	
### Administrator roles
Administrator roles in Azure AD allow users elevated access to control who is allowed to do what. You assign these roles to a limited group of users to manage identity tasks in an Azure AD organization. You can assign administrator roles that allow a user to create or edit users, assign administrative roles to others, reset user passwords, manage user licenses, and more.

If your user account has the User Administrator or Global Administrator role, you can create a new user in Azure AD by using either the Azure portal, the Azure CLI, or PowerShell. In PowerShell, run the cmdlet New-AzureADUser. In the Azure CLI, use az ad user create.

### Member users
A member user account is a native member of the Azure AD organization that has a set of default permissions like being able to manage their profile information. When someone new joins your organization, they typically have this type of account created for them.

Anyone who isn't a guest user or isn't assigned an administrator role falls into this type. A member user is meant for users who are considered internal to an organization and are members of the Azure AD organization. However, these users shouldn't be able to manage other users by, for example, creating and deleting users. Member users don't have the same restrictions that are typically placed on guest users.

### Guest users
Guest users have restricted Azure AD organization permissions. When you invite someone to collaborate with your organization, you add them to your Azure AD organization as a guest user. Then you can either send an invitation email that contains a redemption link or send a direct link to an app you want to share. Guest users sign in with their own work, school, or social identities. By default, Azure AD member users can invite guest users. This default can be disabled by someone who has the User Administrator role.

Your organization might need to work with an external partner. To collaborate with your organization, these partners often need to have a certain level of access to specific resources. For this sort of situation, it's a good idea to use guest user accounts. You'll then make sure partners have the right level of access to do their work, without having a higher level of access than they need.

Add user accounts
You can add individual user accounts through the Azure portal, Azure PowerShell, or the Azure CLI.

If you want to use the Azure CLI, run the following cmdlet:

Azure CLI

```
# create a new user
az ad user create
```	
This command creates a new user by using the Azure CLI.

For Azure PowerShell, run the following cmdlet:

```PowerShell
# create a new user
New-AzureADUser
````
You can bulk create member users and guests accounts. The following example shows how to bulk invite guest users.

```PowerShell
$invitations = import-csv c:\bulkinvite\invitations.csv

$messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo

$messageInfo.customizedMessageBody = "Hello. You are invited to the Contoso organization."

foreach ($email in $invitations)
   {New-AzureADMSInvitation `
      -InvitedUserEmailAddress $email.InvitedUserEmailAddress `
      -InvitedUserDisplayName $email.Name `
      -InviteRedirectUrl https://myapps.microsoft.com `
      -InvitedUserMessageInfo $messageInfo `
      -SendInvitationMessage $true
   }
```
	
You create the comma-separated values (CSV) file with the list of all the users you want to add. An invitation is sent to each user in that CSV file.

Delete user accounts
You can also delete user accounts through the Azure portal, Azure PowerShell, or the Azure CLI. In PowerShell, run the cmdlet Remove-AzADUser. In the Azure CLI, run the cmdlet az ad user delete.

When you delete a user, the account remains in a suspended state for 30 days. During that 30-day window, the user account can be restored.

### Why use SSPR?
In Azure AD, any user can change their password if they're already signed in. But if they're not signed in and forgot their password or it's expired, they'll need to reset their password. With SSPR, users can reset their passwords in a web browser or from a Windows sign-in screen to regain access to Azure, Microsoft 365, and any other application that uses Azure AD for authentication.

SSPR reduces the load on administrators, because users can fix password problems themselves, without having to call the help desk. Also, it minimizes the productivity impact of a forgotten or expired password. Users don't have to wait until an administrator is available to reset their password.

### How SSPR works
The user initiates a password reset either by going directly to the password reset portal or by selecting the Can't access your account link on a sign-in page. The reset portal takes these steps:

 1. Localization: The portal checks the browser's locale setting and renders the SSPR page in the appropriate language.
 2. Verification: The user enters their username and passes a captcha to ensure that it's a user and not a bot.
 3. Authentication: The user enters the required data to authenticate their identity. They might, for example, enter a code or answer security questions.
 4. Password reset: If the user passes the authentication tests, they can enter a new password and confirm it.
 5. Notification: A message is sent to the user to confirm the reset.
	
Authenticate a password reset
It's critical to verify the identity of a user before you allow a password reset. Malicious users might exploit any weakness in the system to impersonate that user. Azure supports six different ways to authenticate reset requests.

As an administrator, you choose the methods to use when you configure SSPR. Enable two or more of these methods so that users can choose the ones that they can use easily. The methods are:

|`Authentication method`| `How to register`| `How to authenticate for a password reset`|
|--------------------|-----------------|--------------------------|
|Mobile app notification| Install the Microsoft Authenticator app on your mobile device, and then register it on the multifactor authentication setup page.| Azure sends a notification to the app, which you can either verify or deny.
|Mobile app code| This method also uses the Authenticator app, and you install and register it in the same way.| Enter the code from the app.
Email| Provide an email address that's external to Azure and Microsoft 365.|	Azure sends a code to the address, which you enter in the reset wizard.
|Mobile phone| Provide a mobile phone number.|	Azure sends a code to the phone in an SMS message, which you enter in the reset wizard. Or, you can choose to get an automated call.
|Office phone| Provide a nonmobile phone number.| You receive an automated call to this number and press #.
|Security questions| Select questions such as "*In what city was your mother born?*" and save responses to them.| Answer the questions.|

Require the minimum number of authentication methods
You can specify the minimum number of methods that the user must set up: one or two. For example, you might enable the mobile app code, email, office phone, and security questions methods and specify a minimum of two methods. Then users can choose the two methods they prefer, like mobile app code and email.

For the security question method, you can specify a minimum number of questions that the user must set up to register for this method. You also can specify a minimum number of questions that they must answer correctly to reset their password.

After your users register the required information for the minimum number of methods you've specified, they're considered registered for SSPR.

### Recommendations
- Enable two or more of the authentication reset request methods.
- Use the mobile app notification or code as the primary method, but also enable the email or office phone methods to support users without mobile devices.
- The mobile phone method isn't a recommended method because it's possible to send fraudulent SMS messages.
- The security question option is the least recommended method because the answers to the security questions might be known to other people. Only use the security question method in combination with at least one other method.
### Accounts associated with administrator roles
- A strong, two-method authentication policy is always applied to accounts with an administrator role, regardless of your configuration for other users.
- The security questions method isn't available to accounts that are associated with an administrator role.
	
### Configure notifications
Administrators can choose how users are notified of password changes. There are two options that you can enable:

- Notify users on password resets: The user who resets their own password is notified to their primary and secondary email addresses. If the reset was done by a malicious user, this notification alerts the user, who can take mitigation steps.
- Notify all admins when other admins reset their password: All administrators are notified when another administrator resets their password.

### License requirements
The editions of Azure AD are free, Premium P1, and Premium P2. The password reset functionality you can use depends on your edition.

Any user who is signed in can change their password, regardless of the edition of Azure AD.

If you're not signed in and you've forgotten your password or your password has expired, you can use SSPR in Azure AD Premium P1 or P2. It's also available with Microsoft 365 Apps for business or Microsoft 365.

In a hybrid situation, where you have Active Directory on-premises and Azure AD in the cloud, any password change in the cloud must be written back to the on-premises directory. This writeback support is available in Azure AD Premium P1 or P2. It's also available with Microsoft 365 Apps for business. 

When is a user considered registered for SSPR?
	
- When they've registered at least the number of methods that you've required to reset a password
  - A user is considered registered for SSPR when they've registered at least the number of methods that you've required to reset a password. You can set this number in the Azure portal.

 When you enable SSPR for your Azure AD organization...
	
- Users can reset their passwords when they can't sign in
  - If the user passes the authentication tests, then they can reset their password.
	
# Implement Azure Storage

Things to know about Azure Storage
You can think of Azure Storage as supporting three categories of data: structured data, unstructured data, and virtual machine data. Review the following categories and think about which types of storage are used in your organization.

Category| Description| Storage examples|
|----------|---------------------------|---------------------------|
Virtual machine data|	Virtual machine data storage includes disks and files. Disks are persistent block storage for Azure IaaS virtual machines. Files are fully managed file shares in the cloud.|	Storage for virtual machine data is provided through Azure managed disks. Data disks are used by virtual machines to store data like database files, website static content, or custom application code. The number of data disks you can add depends on the virtual machine size. Each data disk has a maximum capacity of 32,767 GB.
Unstructured data|	Unstructured data is the least organized. It can be a mix of information that's stored together, but the data doesn't have a clear relationship. The format of unstructured data is referred to as non-relational.|	Unstructured data can be stored by using Azure Blob Storage and Azure Data Lake Storage. Blob Storage is a highly scalable, REST-based cloud object store. Azure Data Lake Storage is the Hadoop Distributed File System (HDFS) as a service.
Structured data|	Structured data is stored in a relational format that has a shared schema. Structured data is often contained in a database table with rows, columns, and keys. Tables are an autoscaling NoSQL store.|	Structured data can be stored by using Azure Table Storage, Azure Cosmos DB, and Azure SQL Database. Azure Cosmos DB is a globally distributed database service. Azure SQL Database is a fully managed database-as-a-service built on SQL.|

Things to consider when using Azure Storage
- Consider durability and availability. Azure Storage is durable and highly available. Redundancy ensures your data is safe during transient hardware failures. You replicate data across datacenters or geographical regions for protection from local catastrophe or natural disaster. Data that's replicated remains highly available during an unexpected outage.

- Consider secure access. All data written to Azure Storage is encrypted by the service. Azure Storage provides you with fine-grained control over who has access to your data.

- Consider scalability. Azure Storage is designed to be massively scalable to meet the data storage and performance needs of modern applications.

- Consider manageability. Microsoft Azure handles hardware maintenance, updates, and critical issues for you.

- Consider data accessibility. Data in Azure Storage is accessible from anywhere in the world over HTTP or HTTPS. Microsoft provides SDKs for Azure Storage in various languages. You can use .NET, Java, Node.js, Python, PHP, Ruby, Go, and the REST API. Azure Storage supports scripting in Azure PowerShell or the Azure CLI. The Azure portal and Azure Storage Explorer offer easy visual solutions for working with your data.

# Explore Azure Storage services

Azure Storage offers four data services that can be accessed by using an Azure storage account:

- Azure Blob Storage (containers): A massively scalable object store for text and binary data.

- Azure Files: Managed file shares for cloud or on-premises deployments.

- Azure Queue Storage: A messaging store for reliable messaging between application components.

- Azure Table Storage: A NoSQL store for schemaless storage of structured data or relational data.
	
### Azure Blob Storage (containers)
Azure Blob Storage is Microsoft's object storage solution for the cloud. Blob Storage is optimized for storing massive amounts of unstructured or non-relational data, such as text or binary data. Blob Storage is ideal for:

- Serving images or documents directly to a browser.
- Storing files for distributed access.
- Streaming video and audio.
- Storing data for backup and restore, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.
Objects in Blob Storage can be accessed from anywhere in the world via HTTP or HTTPS. Users or client applications can access blobs via URLs, the Azure Storage REST API, Azure PowerShell, the Azure CLI, or an Azure Storage client library. The storage client libraries are available for multiple languages, including .NET, Java, Node.js, Python, PHP, and Ruby.
	
### Azure Files
Azure Files enables you to set up highly available network file shares. Shares can be accessed by using the Server Message Block (SMB) protocol and the Network File System (NFS) protocol. Multiple virtual machines can share the same files with both read and write access. You can also read the files by using the REST interface or the storage client libraries.

File shares can be used for many common scenarios:

- Many on-premises applications use file shares. This feature makes it easier to migrate those applications that share data to Azure. If you mount the file share to the same drive letter that the on-premises application uses, the part of your application that accesses the file share should work with minimal, if any, changes.
- Configuration files can be stored on a file share and accessed from multiple virtual machines. Tools and utilities used by multiple developers in a group can be stored on a file share, ensuring that everybody can find them, and that they use the same version.
- Diagnostic logs, metrics, and crash dumps are just three examples of data that can be written to a file share and processed or analyzed later.
The storage account credentials are used to provide authentication for access to the file share. All users who have the share mounted should have full read/write access to the share.

### Azure Queue Storage
Azure Queue Storage is used to store and retrieve messages. Queue messages can be up to 64 KB in size, and a queue can contain millions of messages. Queues are used to store lists of messages to be processed asynchronously.

Consider a scenario where you want your customers to be able to upload pictures, and you want to create thumbnails for each picture. You could have your customer wait for you to create the thumbnails while uploading the pictures. An alternative is to use a queue. When the customer finishes the upload, you can write a message to the queue. Then you can use an Azure Function to retrieve the message from the queue and create the thumbnails. Each of the processing parts can be scaled separately, which gives you more control when tuning the configuration.

### Azure Table Storage (Azure Cosmos DB)
Azure Table Storage is now part of Azure Cosmos DB, which is a fully managed NoSQL database service for modern app development. As a fully managed service, Azure Cosmos DB takes database administration off your hands with automatic management, updates, and patching. It also handles capacity management with cost-effective serverless and automatic scaling options that respond to application needs to match capacity with demand.

In addition to the existing Azure Table Storage service, there's a new Azure Cosmos DB Table API offering that provides throughput-optimized tables, global distribution, and automatic secondary indexes.
	
Things to consider when choosing Azure Storage services
As you think about your configuration plan for Azure Storage, consider the prominent features of the types of Azure Storage and which options support your application needs.

- Consider storage optimization for massive data. Azure Blob Storage is optimized for storing massive amounts of unstructured data. Objects in Blob Storage can be accessed from anywhere in the world via HTTP or HTTPS. Blob Storage is ideal for serving data directly to a browser, streaming data, and storing data for backup and restore.

- Consider storage with high availability. Azure Files supports highly available network file shares. On-premises apps use file shares for easy migration. By using Azure Files, all users can access shared data and tools. Storage account credentials provide file share authentication to ensure all users who have the file share mounted have the correct read/write access.

- Consider storage for messages. Use Azure Queue Storage to store large numbers of messages. Queue Storage is commonly used to create a backlog of work to process asynchronously.

- Consider storage for structured data. Azure Table Storage is ideal for storing structured, non-relational data. It provides throughput-optimized tables, global distribution, and automatic secondary indexes. Because Azure Table Storage is part of Azure Cosmos DB, you have access to a fully managed NoSQL database service for modern app development.

# Determine storage account types


|Storage account| Supported services| Recommended usage|
|---------------|-----------------------|------------------------|	
[Standard general-purpose v2](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-upgrade)|	Blob Storage (including Data Lake Storage), Queue Storage, Table Storage, and Azure Files|	Standard storage account for most scenarios, including blobs, file shares, queues, tables, and disks (page blobs).
|[Premium block blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-block-blob-premium)| Blob Storage (including Data Lake Storage)| Premium storage account for block blobs and append blobs. Recommended for applications with high transaction rates. Use Premium block blobs if you work with smaller objects or require consistently low storage latency. This storage is designed to scale with your applications.
[Premium file shares](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-create-file-share)| Azure Files| Premium storage account for file shares only. Recommended for enterprise or high-performance scale applications. Use Premium file shares if you require support for both Server Message Block (SMB) and NFS file shares.
|[Premium page blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-pageblob-overview)| Page blobs only| Premium high-performance storage account for page blobs only. Page blobs are ideal for storing index-based and sparse data structures, such as operating systems, data disks for virtual machines, and databases.|
	
The data in your Azure storage account is always replicated to ensure durability and high availability. Azure Storage replication copies your data so that it's protected from planned and unplanned events. These events range from transient hardware failures, network or power outages, massive natural disasters, and so on. You can choose to replicate your data within the same data center, across zonal data centers within the same region, and even across regions. Replication ensures your storage account meets the Service-Level Agreement (SLA) for Azure Storage even if there are failures.

We'll explore four replication strategies:

- Locally redundant storage (LRS)
- Zone redundant storage (ZRS)
- Geo-redundant storage (GRS)
- Geo-zone-redundant storage (GZRS)
	
### Locally redundant storage
Locally redundant storage is the lowest-cost replication option and offers the least durability compared to other strategies. If a data center-level disaster occurs, such as fire or flooding, all replicas might be lost or unrecoverable. Despite its limitations, LRS can be appropriate in several scenarios:

- Your application stores data that can be easily reconstructed if data loss occurs.
- Your data is constantly changing like in a live feed, and storing the data isn't essential.
- Your application is restricted to replicating data only within a country due to data governance requirements.

### Zone redundant storage
Zone redundant storage synchronously replicates your data across three storage clusters in a single region. Each storage cluster is physically separated from the others and resides in its own availability zone. Each availability zone, and the ZRS cluster within it, is autonomous, and has separate utilities and networking capabilities. Storing your data in a ZRS account ensures you can access and manage your data if a zone becomes unavailable. ZRS provides excellent performance and low latency.

- ZRS isn't currently available in all regions.
- Changing to ZRS from another data replication option requires the physical data movement from a single storage stamp to multiple stamps within a region.
Geo-redundant storage
Geo-redundant storage replicates your data to a secondary region (hundreds of miles away from the primary location of the source data). GRS provides a higher level of durability even during a regional outage. GRS is designed to provide at least 99.99999999999999% (16 9's) durability. When your storage account has GRS enabled, your data is durable even when there's a complete regional outage or a disaster where the primary region isn't recoverable.

If you implement GRS, you have two related options to choose from:

- GRS replicates your data to another data center in a secondary region. The data is available to be read only if Microsoft initiates a failover from the primary to secondary region.

- Read-access geo-redundant storage (RA-GRS) is based on GRS. RA-GRS replicates your data to another data center in a secondary region, and also provides you with the option to read from the secondary region. With RA-GRS, you can read from the secondary region regardless of whether Microsoft initiates a failover from the primary to the secondary.

For a storage account with GRS or RA-GRS enabled, all data is first replicated with locally redundant storage. An update is first committed to the primary location and replicated by using LRS. The update is then replicated asynchronously to the secondary region by using GRS. When data is written to the secondary location, it's also replicated within that location by using LRS. Both the primary and secondary regions manage replicas across separate fault domains and upgrade domains within a storage scale unit. The storage scale unit is the basic replication unit within the datacenter. Replication at this level is provided by LRS.

Geo-zone redundant storage
Geo-zone-redundant storage combines the high availability of zone-redundant storage with protection from regional outages as provided by geo-redundant storage. Data in a GZRS storage account is replicated across three Azure availability zones in the primary region, and also replicated to a secondary geographic region for protection from regional disasters. Each Azure region is paired with another region within the same geography, together making a regional pair.

With a GZRS storage account, you can continue to read and write data if an availability zone becomes unavailable or is unrecoverable. Additionally, your data is also durable during a complete regional outage or during a disaster in which the primary region isn't recoverable. GZRS is designed to provide at least 99.99999999999999% (16 9's) durability of objects over a given year. GZRS also offers the same scalability targets as LRS, ZRS, GRS, or RA-GRS. You can optionally enable read access to data in the secondary region with read-access geo-zone-redundant storage (RA-GZRS).
	
# Access storage

Every object you store in Azure Storage has a unique URL address. Your storage account name forms the subdomain portion of the URL address. The combination of the subdomain and the domain name, which is specific to each service, forms an endpoint for your storage account.

Let's look at an example. If your storage account name is mystorageaccount, default endpoints for your storage account are formed for the Azure services as shown in the following table:

|Service| Default endpoint|
|-------------------|------------------------------------------|	
|Container service| `//mystorageaccount.blob.core.windows.net`
|Table service| `//mystorageaccount.table.core.windows.net`
|Queue service| `//mystorageaccount.queue.core.windows.net`
|File service| `//mystorageaccount.file.core.windows.net`
	
We create the URL to access an object in your storage account by appending the object's location in the storage account to the endpoint.

To access the myblob data in the mycontainer location in your storage account, we use the following URL address:

`//mystorageaccount.blob.core.windows.net/mycontainer/myblob.`

### Configure custom domains
You can configure a custom domain to access blob data in your Azure storage account. As we reviewed, the default endpoint for Azure Blob Storage is `\<storage-account-name>.blob.core.windows.net.` You can also use the web endpoint that's generated as a part of the static websites feature. If you map a custom domain and subdomain, such as `www.contoso.com`, to the blob or web endpoint for your storage account, your users can use that domain to access blob data in your storage account.

   >**Note**:Azure Storage doesn't currently provide native support for HTTPS with custom domains. You can implement an Azure Content Delivery Network (CDN) to access blobs by using custom domains over HTTPS.

There are two ways to configure a custom domain: direct mapping and intermediary domain mapping.

 - Direct mapping lets you enable a custom domain for a subdomain to an Azure storage account. For this approach, you create a CNAME record that points from the subdomain to the Azure storage account.

The following example shows how a subdomain is mapped to an Azure storage account to create a CNAME record in the domain name system (DNS):

  - Subdomain: blobs.contoso.com
  - Azure storage account: \<storage account>\.blob.core.windows.net
  - Direct CNAME record: contosoblobs.blob.core.windows.net
- Intermediary domain mapping is applied to a domain that's already in use within Azure. This approach might result in minor downtime while the domain is being mapped. To avoid downtime, you can use the asverify intermediary domain to validate the domain. By prepending the asverify keyword to your own subdomain, you permit Azure to recognize your custom domain without modifying the DNS record for the domain. After you modify the DNS record for the domain, your domain is mapped to the blob endpoint with no downtime.

The following example shows how a domain in use is mapped to an Azure storage account in the DNS with the asverify intermediary domain:

  - CNAME record: asverify.blobs.contoso.com
  - Intermediate CNAME record: asverify.contosoblobs.blob.core.windows.net

|Authorization strategy|	Description|
|----------------------|------------------------|	
|Azure Active Directory| Azure AD is Microsoft's cloud-based identity and access management service. With Azure AD, you can assign fine-grained access to users, groups, or applications by using role-based access control.
|Shared Key| Shared Key authorization relies on your Azure storage account access keys and other parameters to produce an encrypted signature string. The string is passed on the request in the Authorization header.
|Shared access signatures| A SAS delegates access to a particular resource in your Azure storage account with specified permissions and for a specified time interval.
|Anonymous access to containers and blobs| You can optionally make blob resources public at the container or blob level. A public container or blob is accessible to any user for anonymous read access. Read requests to public containers and blobs don't require authorization.
	
# Apply Azure Storage security best practices
	
|Recommendation| Description|
|-----------------------|------------------------|
|Always use HTTPS for creation and distribution| If a SAS is passed over HTTP and intercepted, an attacker can intercept and use the SAS. These man-in-the-middle attacks can compromise sensitive data or allow for data corruption by the malicious user.
|Reference stored access policies where possible| Stored access policies give you the option to revoke permissions without having to regenerate the Azure storage account keys. Set the storage account key expiration date far in the future.
|Set near-term expiry times for an unplanned SAS| If a SAS is compromised, you can mitigate attacks by limiting the SAS validity to a short time. This practice is important if you can't reference a stored access policy. Near-term expiration times also limit the amount of data that can be written to a blob by limiting the time available to upload to it.
|Require clients automatically renew the SAS| Require your clients to renew the SAS well before the expiration date. By renewing early, you allow time for retries if the service providing the SAS is unavailable.
|Plan carefully for the SAS start time|	If you set the start time for a SAS to now, then due to clock skew (differences in current time according to different machines), failures might be observed intermittently for the first few minutes. In general, set the start time to at least 15 minutes in the past. Or, don't set a specific start time, which causes the SAS to be valid immediately in all cases. The same conditions generally apply to the expiry time. You might observe up to 15 minutes of clock skew in either direction on any request. For clients that use a REST API version earlier than 2012-02-12, the maximum duration for a SAS that doesn't reference a stored access policy is 1 hour. Any policies that specify a longer term will fail.
|Define minimum access permissions for resources| A security best practice is to provide a user with the minimum required privileges. If a user only needs read access to a single entity, then grant them read access to that single entity, and not read/write/delete access to all entities. This practice also helps lessen the damage if a SAS is compromised because the SAS has less power in the hands of an attacker.
|Understand account billing for usage, including a SAS|	If you provide write access to a blob, a user might choose to upload a 200-GB blob. If you've given them read access as well, they might choose to download the blob 10 times, which incurs 2 TB in egress costs for you. Again, provide limited permissions to help mitigate the potential actions of malicious users. Use a short-lived SAS to reduce this threat, but be mindful of clock skew on the end time.
|Validate data written by using a SAS| When a client application writes data to your Azure storage account, keep in mind there can be problems with the data. If your application requires validated or authorized data, validate the data after it's written, but before it's used. This practice also protects against corrupt or malicious data being written to your account, either by a user who properly acquired the SAS, or by a user exploiting a leaked SAS.
|Don't assume a SAS is always the correct choice| In some scenarios, the risks associated with a particular operation against your Azure storage account outweigh the benefits of using a SAS. For such operations, create a middle-tier service that writes to your storage account after performing business rule validation, authentication, and auditing. Also, sometimes it's easier to manage access in other ways. If you want to make all blobs in a container publicly readable, you can make the container Public, rather than providing a SAS to every client for access.
|Monitor your applications with Azure Storage Analytics| You can use logging and metrics to observe any spike in authentication failures. You might see spikes from an outage in your SAS provider service or to the inadvertent removal of a stored access policy.|
	
### Compare Azure Files to Blob Storage and Azure Disks
It can be difficult to determine exactly when to use Azure Files to store data as file shares rather than Azure Blob Storage or Azure Disks to store data as blobs. The following table compares different features of these services and common implementation scenarios.


|Azure Files (file shares)| Azure Blob Storage (blobs)|	Azure Disks (page blobs)|
|-------------------------|---------------------------|-------------------------|	
|Azure Files provides the SMB and NFS protocols, client libraries, and a REST interface that allows access from anywhere to stored files.|	Azure Blob Storage provides client libraries and a REST interface that allows unstructured data to be stored and accessed at a massive scale in block blobs.|	Azure Disks is similar to Azure Blob Storage. Azure Disks provides a REST interface to store and access index-based or structured data in page blobs.
|_**Azure Files**_ is ideal to lift and shift an application to the cloud that already uses the native file system APIs. Share data between the app and other applications running in Azure.
_**Azure Files**_ is a good option when you want to store development and debugging tools that need to be accessed from many virtual machines.| _**Azure Blob Storage**_ is ideal for applications that need to support streaming and random-access scenarios. _**Azure Blob Storage**_ is a good option when you want to be able to access application data from anywhere.| _**Azure Disks**_ solutions are ideal when your applications run frequent random read/write operations. *Azure Disks* is a good option when you want to store relational data for operating system and data disks in Azure Virtual Machines and databases.|
	
### Things to consider when using Azure File Sync
There are many advantages to using Azure File Sync. Consider the following scenarios, and think about how you can use Azure File Sync with your Azure Files shares.

- Consider application lift and shift. Use Azure File Sync to move applications that require access between Azure and on-premises systems. Provide write access to the same data across Windows Servers and Azure Files.

- Consider support for branch offices. Support your branch offices that need to back up files by using Azure File Sync. Use the service to set up a new server that connects to Azure storage.

- Consider backup and disaster recovery. After you implement Azure File Sync, Azure Backup backs up your on-premises data. Restore file metadata immediately and recall data as needed for rapid disaster recovery.

- Consider file archiving with cloud tiering. Azure File Sync stores only recently accessed data on local servers. Implement cloud tiering so non-used data moves to Azure Files.

### Azure File Sync agent
The Azure File Sync agent is a downloadable package that enables Windows Server to be synced with an Azure Files share. The Azure File Sync agent has three main components:

- FileSyncSvc.exe: This file is the background Windows service that's responsible for monitoring changes on server endpoints, and for initiating sync sessions to Azure.

- StorageSync.sys: This file is the Azure File Sync file system filter that supports cloud tiering. The filter is responsible for tiering files to Azure Files when cloud tiering is enabled.

- PowerShell cmdlets: These PowerShell management cmdlets allow you to interact with the Microsoft.StorageSync Azure resource provider. You can find the cmdlets at the following (default) locations:

  - `C:\\Program Files\\Azure\\StorageSyncAgent\\StorageSync.Management.PowerShell.Cmdlets.dll`
  - `C:\\Program Files\\Azure\\StorageSyncAgent\\StorageSync.Management.ServerCmdlets.dll`
	
Server endpoint
A server endpoint represents a specific location on a registered server, such as a folder on a server volume. Multiple server endpoints can exist on the same volume if their namespaces are unique (for example, F:\\sync1 and F:\\sync2).

Cloud endpoint
 - A cloud endpoint is an Azure Files share that's part of a sync group. As part of a sync group, the entire cloud endpoint (Azure Files share) syncs.

 - An Azure Files share can be a member of one cloud endpoint only.

 - An Azure Files share can be a member of one sync group only.

Consider the scenario where you have a share with existing files. If you add the share as a cloud endpoint to a sync group, the files in the share are merged with files on other endpoints in the sync group.

# Deploy Azure File Sync
	
|Deploy Storage Sync Service| Prepare Windows Server(s)| Install Azure File Synce Agent| Register Windows Servers|
|---------------------------|--------------------------|-------------------------------|-------------------------|
	
### Step 1: Deploy the Storage Sync Service
You can deploy the Storage Sync Service from the Azure portal. You configure the following settings:

- The deployment name for the Storage Sync Service
- The Azure subscription ID to use for the deployment
- A Resource Group for the deployment
- The deployment location 

### Step 2: Prepare each Windows Server to use Azure File Sync
After you deploy the Storage Sync Service, you configure each Windows Server or cloud virtual machine that you intend to use with Azure File Sync, including server nodes in a Failover Cluster.

### Step 3: Install the Azure File Sync agent
When the Windows Server configuration is complete, you're ready to install the Azure File Sync agent. The agent is a downloadable package that enables Windows Server to be synced with an Azure Files share. The Azure File Sync agent installation package should install relatively quickly.

   >**Note**: For the agent installation, Microsoft recommends using the default installation path. Also enable Microsoft Update to ensure your severs are running the latest version of Azure File Sync.

### Step 4: Register each Windows Server with the Storage Sync Service
After the Azure File Sync agent installation completes, the Server Registration window opens.

By registering the Windows Server with a Storage Sync Service, you establish a trust relationship between your server (or cluster) and the Storage Sync Service. For the registration, you need your Azure subscription ID and some of the deployment settings you configured in the first step:

- The Storage Sync Service deployment name
- The Resource Group for the deployment
	
   >**Note**: A server (or cluster) can be registered with only one Storage Sync Service resource at a time.

# Use the AzCopy tool

An alternate method for transferring data is the AzCopy tool. AzCopy v10 is the next-generation command-line utility for copying data to and from Azure Blob Storage and Azure Files. AzCopy v10 offers a redesigned command-line interface (CLI) and new architecture for high-performance reliable data transfers. You can use AzCopy to copy data between a file system and a storage account, or between storage accounts.

### Things to know about AzCopy
Let's look at some of the characteristics of the AzCopy tool.

Every AzCopy instance creates a job order and a related log file. You can view and restart previous jobs, and resume failed jobs.

You can use AzCopy to list or remove files or blobs in a given path. AzCopy supports wildcard patterns in a path, `--include` flags, and `--exclude` flags.

AzCopy automatically retries a transfer when a failure occurs.

When you use Azure Blob Storage, AzCopy lets you copy an entire account to another account with the `Put` command from URL APIs. No data transfer to the client is needed.

AzCopy supports Azure Data Lake Storage Gen2 APIs.

AzCopy is built into Azure Storage Explorer.

AzCopy is available on Windows, Linux, and macOS.

Authentication options
There are two options to authenticate your transferred data when using AzCopy.

|Authentication| Support| Description|
|--------------|---------------|-----------------------------------|
|Azure Active Directory (Azure AD)| Azure Blob Storage and Azure Data Lake Storage Gen2| The user enters the `.\\azcopy` sign-in command to sign in by using Azure AD. The user should have the Storage Blob Data Contributor role assigned, which allows them to write to Blob Storage by using Azure AD authentication. When the user signs in from Azure AD, they provide their credentials only once. This option allows the user to circumvent having to append a SAS token to each command. 
|SAS tokens| Azure Blob Storage and Azure Files| On the command line, the user appends a SAS token to the blob or file path for every command they enter.|
	
### AzCopy and Azure Storage Explorer
Azure Storage Explorer uses the AzCopy tool for all of its data transfers. If you want to use a graphical UI to work with your files, you can use Azure Storage Explorer and gain the performance advantages of AzCopy.

Azure Storage Explorer uses your account key to perform operations. After you sign into Azure Storage Explorer, you don't need to provide your authorization credentials again.

### Things to consider when using AzCopy
Review the following scenarios for using AzCopy. Consider how the tool features can enhance your Azure Storage solution.

- Consider data synchronization. Use AzCopy to synchronize a file system to Azure Blob Storage and vice versa. AzCopy is ideal for incremental copy scenarios.

- Consider job management. Manage your transfer operations with AzCopy. View and restart previous jobs. Resume failed jobs.

- Consider transfer resiliency. Provide data resiliency for your data transfers. If a copy job fails, AzCopy automatically retries the copy.

- Consider fast account to account copy. Use AzCopy with Azure Blob Storage for the account to account copy feature. Because data isn't transferred to the client, the transfer is faster.
	
The basic CLI syntax for AzCopy starts with the `azcopy` command followed by the type of job to perform, such as `copy`. For the `copy` command, you specify the `[source]` path of the files to copy, the `[destination]` path for the copied files, and any `[flags]` for options to apply to the transfer job.
	
```bash
azcopy copy [source] [destination] [flags]
```
	
Here's how you can get a list of available CLI commands for AzCopy:
	
```bash
azcopy --help
```
	
# Authorization options for Azure Storage

Before you enhance your company's patient diagnostic image web app, you'd like to understand all the options for secure access. A shared access signature (SAS) provides a secure way of granting access to resources for clients. But it's not the only way to grant access. In some situations, other options might offer better choices for your organization.

Your company could make use of more than just the SAS method of authentication.

### Access Azure Storage
Files stored in Azure Storage are accessed by clients over HTTP/HTTPS. Azure checks each client request for authorization to access stored data. Four options are available for blob storage:

- Public access
- Azure Active Directory (Azure AD)
- Shared key
- Shared access signature (SAS)
#### Public access
Public access is also known as anonymous public read access for containers and blobs.

There are two separate settings that affect public access:

- The Storage Account. Configure the storage account to allow public access by setting the AllowBlobPublicAccess property. When set to true, Blob data is available for public access only if the container's public access setting is also set.

- The Container. You can enable anonymous access only if anonymous access has been allowed for the storage account. A container has two possible settings for public access: Public read access for blobs, or public read access for a container and its blobs. Anonymous access is controlled at the container level, not for individual blobs. This means that if you want to secure some of the files, you need to put them in a separate container that doesn't permit public read access.

Both storage account and container settings are required to enable anonymous public access. The advantages of this approach are that you don't need to share keys with clients who need access to your files. You also don't need to manage a SAS.

#### Azure Active Directory
Use the Azure AD option to securely access Azure Storage without storing any credentials in your code. AD authorization takes a two-step approach. First, you authenticate a security principal that returns an OAuth 2.0 token if successful. This token is then passed to Azure Storage to enable authorization to the requested resource.

Use this form of authentication if you're running an app with managed identities or using security principals.

#### Shared key
Azure Storage creates two 512-bit access keys for every storage account that's created. You share these keys to grant clients access to the storage account. These keys grant anyone with access the equivalent of root access to your storage.

We recommend that you manage storage keys with Azure Key Vault because it's easy to rotate keys on a regular schedule to keep your storage account secure.

#### Shared access signature
A SAS lets you grant granular access to files in Azure Storage, such as read-only or read-write access, expiration time, after which the SAS no longer enables the client to access the chosen resources. A shared access signature is a key that grants permission to a storage resource, and should be protected in the same manner as an account key.

Azure Storage supports three types of shared access signatures:

- User delegation SAS: Can only be used for Blob storage and is secured with Azure AD credentials.
- Service SAS: A service SAS is secured using a storage account key. A service SAS delegates access to a resource in any one of four Azure Storage services: Blob, Queue, Table, or File.
- Account SAS: An account SAS is secured with a storage account key. An account SAS has the same controls as a service SAS, but can also control access to service-level operations, such as Get Service Stats.
You can create a SAS ad-hoc by specifying all the options you need to control, including start time, expiration time, and permissions.

If you plan to create a service SAS, there's also an option to associate it with a stored access policy. A stored access policy can be associated with up to five active SASs. You can control access and expiration at the stored access policy level. This is a good approach if you need to have granular control to change the expiration, or to revoke a SAS. The only way to revoke or change an ad-hoc SAS is to change the storage account keys.

### Create a SAS in .NET
Because your company provides access to third parties, you can't use Azure AD to create service principals for each third party that requires access to medical images. Your app uses a storage account key for each individual file. The following steps show how to accomplish these steps in code.
	
Create a blob container to connect to the storage account on Azure

```C#
BlobContainerClient container = new BlobContainerClient( "ConnectionString", "Container" );
```	

Retrieve the blob you want to create a SAS token for and create a BlobClient

```C#
foreach (BlobItem blobItem in container.GetBlobs())
{
    BlobClient blob = container.GetBlobClient(blobItem.Name);
}
```	
	
Create a BlobSasBuilder object for the blob you use to generate the SAS token
	
```C#
BlobSasBuilder sas = new BlobSasBuilder
{
    BlobContainerName = blob.BlobContainerName,
    BlobName = blob.Name,
    Resource = "b",
    ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(1)
};

// Allow read access
sas.SetPermissions(BlobSasPermissions.Read);
```
	
Authenticate a call to the ToSasQueryParameters method of the BlobSasBuilder object
```C#
StorageSharedKeyCredential storageSharedKeyCredential = new StorageSharedKeyCredential( "AccountName", "AccountKey");

sasToken = sas.ToSasQueryParameters(storageSharedKeyCredential).ToString();
```

Best practices
To reduce the potential risks of using a SAS, Microsoft provides some guidance:

- To securely distribute a SAS and help prevent man-in-the-middle attacks, always use HTTPS.
- The most secure SAS is user delegation. Use it wherever possible because it removes the need to store your storage account key in code. Azure AD must be used to manage credentials; this option might not be possible for your solution.
- Try to set your expiration time to the smallest useful value. If a SAS key becomes compromised, it can be exploited for only a short time.
- Apply the rule of minimum-required privileges. Only grant the access that's required. For example, in your app, read-only access is sufficient.
- There are some situations where a SAS isn't the correct solution. When there's an unacceptable risk of using a SAS, create a middle-tier service to manage users and their access to storage.
	
The most flexible and secure way to use a service or account SAS is to associate the SAS tokens with a stored access policy. You'll explore these benefits and how they work in a later unit.

The stored access policy you create for a blob container can be used for all the blobs in the container and for the container itself. A stored access policy is created with the following properties:

- Identifier: The name you use to reference the stored access policy.
- Start time: A DateTimeOffset value for the date and time when the policy might start to be used. This value can be null.
- Expiry time: A DateTimeOffset value for the date and time when the policy expires. After this time, requests to the storage will fail with a 403 error-code message.
- Permissions: The list of permissions as a string that can be one or all of acdlrw.	

```bash
az storage container policy create \
    --name <stored access policy identifier> \
    --container-name <container name> \
    --start <start time UTC datetime> \
    --expiry <expiry time UTC datetime> \
    --permissions <(a)dd, (c)reate, (d)elete, (l)ist, (r)ead, or (w)rite> \
    --account-key <storage account key> \
    --account-name <storage account name> \
```
# Plan virtual machines

Before you create an Azure virtual machine, it's helpful to make a plan for the machine configuration. You need to consider your preferences for several options, including the machine size and location, storage usage, and associated costs.

Things to know about configuring virtual machines
Let's walk through a checklist of things you need to consider when configuring a virtual machine.

- Start with the network.
- Choose a name for the virtual machine.
- Decide the location for the virtual machine.
- Determine the size of the virtual machine.
- Review the pricing model and estimate your costs.
- Identify which Azure Storage to use with the virtual machine.
- Select an operating system for the virtual machine.

### Network configuration
Virtual networks are used in Azure to provide private connectivity between Azure Virtual Machines and other Azure services. Virtual machines and services that are part of the same virtual network can access one another. By default, services outside the virtual network can't connect to services within the virtual network. You can, however, configure the network to allow access to the external service, including your on-premises servers.

Network addresses and subnets aren't trivial to change after they're configured. If you plan to connect your private company network to the Azure services, make sure you consider the topology before you put any virtual machines into place.

### Virtual machine name
The virtual machine name is used as the computer name, which is configured as part of the operating system. You can specify a name with up to 15 characters on a Windows virtual machine and 64 characters on a Linux virtual machine.

The virtual machine name also defines a manageable Azure resource, and it's not trivial to change later. You should choose names that are meaningful and consistent, so you can easily identify what the virtual machine does. A good convention uses several of the following elements in the machine name:

|Name element| Examples| Description|
|------------|---------|------------|	
|Environment or purpose| `dev` (development), `prod` (production), `QA` (testing)| A portion of the name should identify the environment or purpose for the machine.
|Location| `uw` (US West), `je` (Japan East), `ne` (North Europe)| Another portion of the name should specify the region where the machine is deployed.
|Instance| `1, 02, 005`| For multiple machines that have similar names, include an instance number in the name to differentiate the machines in the same category.
|Product or service| `Outlook, SQL, AzureAD`| A portion of the name can specify the product, application, or service that the machine supports.
|Role| `security, web, messaging`| A portion of the name can specify what role the machine supports within the organization.|
	
Let's consider how to name the first development web server for your company that's hosted in the US South Central location. In this scenario, you might use the machine name devusc-webvm01. dev stands for development and usc identifies the location. web indicates the machine as a web server, and the suffix 01 shows the machine is the first in the configuration.

### Virtual machine location
Azure has datacenters all over the world filled with servers and disks. These datacenters are grouped into geographic regions like West US, North Europe, Southeast Asia, and so on. The datacenters provide redundancy and availability.

Each virtual machine is in a region where you want the resources like CPU and storage to be allocated. The regional location lets you place your virtual machines as close as possible to your users. The location of the machine can improve performance and ensure you meet any legal, compliance, or tax requirements.

There are two other points to consider about the virtual machine location.

- The machine location can limit your available options. Each region has different hardware available, and some configurations aren't available in all regions.

- There are price differences between locations. To find the most cost-effective choice, check for your required configuration in different regions.

### Virtual machine size
Azure offers different memory and storage options for different [virtual machine sizes](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes). The best way to determine the appropriate machine size is to consider the type of workload your machine needs to run. Based on the workload, you can choose from a subset of available virtual machine sizes.

### Azure Storage
[Azure Managed Disks](https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview) handle Azure storage account creation and management in the background for you. You specify the disk size and the performance tier (Standard or Premium). Azure creates and manages the disk. As you add disks or scale the virtual machine up and down, you don't have to worry about the storage being used.

### Virtual machine pricing options
A subscription is billed two separate costs for every virtual machine: compute and storage. By separating these costs, you can scale them independently and only pay for what you need.

- Compute expenses are priced on a per-hour basis but billed on a per-minute basis. If the virtual machine is deployed for 55 minutes, you're charged for only 55 minutes of usage. You're not charged for compute capacity if you stop and deallocate the virtual machine. The [hourly price](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) varies based on the virtual machine size and operating system you select. For the compute costs, you're able to choose from two payment options:

  - Consumption-based: With the consumption-based option, you pay for compute capacity by the second. You're able to increase or decrease compute capacity on demand and start or stop at any time. Use consumption-based pricing if you run applications with short-term or unpredictable workloads that can't be interrupted. An example scenario is if you're doing a quick test or developing an app in a virtual machine.

  - Reserved Virtual Machine Instances: The Reserved Virtual Machine Instances (RI) option is an advance purchase of a virtual machine for one or three years in a specified region. The commitment is made up front, and in return, you get up to 72% price savings compared to pay-as-you-go pricing. RIs are flexible and can easily be exchanged or returned for an early termination fee. Use this option if the virtual machine has to run continuously, or you need budget predictability, and you can commit to using the virtual machine for at least a year.

- Storage costs are charged separately for the Azure Storage used by the virtual machine. The status of the virtual machine has no relation to the Azure Storage charges that are incurred. You're always charged for any Azure Storage used by the disks.

### Operating system
Azure provides various operating system images that you can install into the virtual machine, including several versions of Windows and flavors of Linux. Azure bundles the cost of the operating system license into the price.

- If you're looking for more than just base operating system images, you can search [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute). There are various install images that include not only the operating system but popular software tools, such as WordPress. The image stack consists of a Linux server, Apache web server, a MySQL database, and PHP. Instead of setting up and configuring each component, you can install an Azure Marketplace image and get the entire stack all at once.

- If you don't find a suitable operating system image, you can create your own disk image. Your disk image can be uploaded to Azure Storage and used to create an Azure virtual machine. Keep in mind that Azure only supports 64-bit operating systems.

|Classification|Description|Scenarios|
|--------------|--------------------------------------------------------------------------------------|------------------------------------------------------------|	
|General purpose|General-purpose virtual machines are designed to have a balanced CPU-to-memory ratio.|<ul><li>Testing and development</li><li>Small to medium databases</li><li>Low to medium traffic web server</li>|
|Compute optimized|Compute optimized virtual machines are designed to have a high CPU-to-memory ratio.|<ul><li>Medium traffic web servers</li><li>Network appliances</li><li> Batch processes</li><li> Application servers</li>|
|Memory optimized| Memory optimized virtual machines are designed to have a high memory-to-CPU ratio.| <ul><li>Relational database servers</li><li>Medium to large caches</li><li>In-memory analytics</li>|
|Storage optimized| Storage optimized virtual machines are designed to have high disk throughput and I/O.|<ul><li>Big Data</li><li>SQL and NoSQL databases</li><li>Data warehousing</li><li>Large transactional databases</li>|
|GPU| GPU virtual machines are specialized virtual machines targeted for heavy graphics rendering and video editing. Available with single or multiple GPUs.| <ul><li>Model training</li><li>Inferencing with deep learning</li>|
|High performance computers| High performance compute offers the fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).|<ul><li>Workloads that require fast performance</li><li>High traffic networks</li>|
	
### Resizing virtual machines
Azure allows you to change the virtual machine size when the existing size no longer meets your needs. You can resize a virtual machine if your current hardware configuration is allowed in the new size. This option provides a fully agile and elastic approach to virtual machine management.

When you stop and deallocate the virtual machine, you can select any size available in your region.

   >**Note**: Be cautious when resizing production virtual machines. Resizing a machine might require a restart that can cause a temporary outage or change configuration settings such as the IP address.
	
### Configure virtual machine image
The Azure portal guides you through the configuration process to create your virtual machine image. The process includes configuring basic and advanced options, and specifying details about the disks, virtual networks, and machine management.


- The Basics tab contains the project details, administrator account, and inbound port rules.

- On the Disks tab, you select the OS disk type and specify your data disks.

- The Networking tab provides settings to create virtual networks and load balancing.

- On the Management tab, you can enable auto-shutdown and specify backup details.

- On the Advanced tab, you can configure agents, scripts, or virtual machine extensions.

- Other settings are available on the Monitoring and Tags tabs.

### Things to know about maintenance planning
An availability plan for Azure virtual machines needs to include strategies for unplanned hardware maintenance, unexpected downtime, and planned maintenance. As you review the following scenarios, think about how these scenarios can impact the example company website.

- An unplanned hardware maintenance event occurs when the Azure platform predicts that the hardware or any platform component associated to a physical machine is about to fail. When the platform predicts a failure, it issues an unplanned hardware maintenance event. Azure uses Live Migration technology to migrate your virtual machines from the failing hardware to a healthy physical machine. Live Migration is a virtual machine preserving operation that only pauses the virtual machine for a short time, but performance might be reduced before or after the event.

- Unexpected downtime occurs when the hardware or the physical infrastructure for your virtual machine fails unexpectedly. Unexpected downtime can include local network failures, local disk failures, or other rack level failures. When detected, the Azure platform automatically migrates (heals) your virtual machine to a healthy physical machine in the same datacenter. During the healing procedure, virtual machines experience downtime (reboot) and in some cases loss of the temporary drive.

- Planned maintenance events are periodic updates made by Microsoft to the underlying Azure platform to improve overall reliability, performance, and security of the platform infrastructure that your virtual machines run on. Most of these updates are performed without any impact to your virtual machines or Cloud Services.

   >**Note**: Microsoft doesn't automatically update your virtual machine operating system or other software. You have complete control and responsibility for those updates. However, the underlying software host and hardware are periodically patched to ensure reliability and high performance.
	
Availability zones are unique physical locations within an Azure region.

Each zone is made up of one or more datacenters that are equipped with independent power, cooling, and networking.

To ensure resiliency, there's a minimum of three separate zones in all enabled regions.

The physical separation of availability zones within a region protects applications and data from datacenter failures.

Zone-redundant services replicate your applications and data across availability zones to protect against single-points-of-failure.

Things to consider when using availability zones
Azure services that support availability zones are divided into two categories.

|Category| Description|	Examples|
|---------------|----------------------------------------------------------|-----------------------------------------------------------------------------|
|Zonal services| Azure zonal services pin each resource to a specific zone.|<ul><li>Azure Virtual Machines</li><li>Azure managed disks</li><li>Standard IP addresses</li>|
|Zone-redundant services| For Azure services that are zone-redundant, the platform replicates automatically across all zones.|<ul><li>Azure Storage that's zone-redundant</li><li>Azure SQL Database</li>|
	
# Compare vertical and horizontal scaling

A robust virtual machine configuration includes support for scalability. Scalability allows throughput for a virtual machine in proportion to the availability of the associated hardware resources. A scalable virtual machine can handle increases in requests without adversely affecting response time and throughput. For most scaling operations, there are two implementation options: vertical and horizontal.

### Things to know about vertical scaling
Vertical scaling, also known as scale up and scale down, involves increasing or decreasing the virtual machine size in response to a workload. Vertical scaling makes a virtual machine more (scale up) or less (scale down) powerful.
	
Here are some scenarios where using vertical scaling can be advantageous:

If you have a service built on a virtual machine that's under-utilized such as on the weekend, you can use vertical scaling to decrease the virtual machine size and reduce your monthly costs.

You can implement vertical scaling to increase your virtual machine size to support larger demand without having to create extra virtual machines.

### Things to know about horizontal scaling
Horizontal scaling, also referred to as scale out and scale in, is used to adjust the number of virtual machines in your configuration to support the changing workload. When you implement horizontal scaling, there's an increase (scale out) or decrease (scale in) in the number of virtual machine instances.
	
# Configure autoscale

Scaling policy: Manual scale maintains a fixed instance count. Custom autoscale scales the capacity on any schedule, based on any metrics.

- Minimum number of VMs: Specify the minimum number of virtual machines that should be available when autoscaling is applied on your Virtual Machine Scale Sets implementation.

- Maximum number of VMs: Specify the maximum number of virtual machines that can be available when autoscaling is applied on your implementation.

### Scale out

- CPU threshold: Specify the CPU usage percentage threshold to trigger the scale-out autoscale rule.

- Duration in minutes: Duration in minutes is the amount of time that Autoscale engine will look back for metrics. For example, 10 minutes means that every time autoscale runs, it will query metrics for the past 10 minutes. This delay allows your metrics to stabilize and avoids reacting to transient spikes.

- Number of VMs to increase by: Specify the number of virtual machines to add to your Virtual Machine Scale Sets implementation when the scale-out autoscale rule is triggered.

### Scale in

- Scale in CPU threshold: Specify the CPU usage percentage threshold to trigger the scale-in autoscale rule.

- Number of VMs to decrease by: Specify the number of virtual machines to remove from your implementation when the scale-in autoscale rule is triggered.

Scale in policy: The [scale-in policy](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-scale-in-policy) feature provides users a way to configure the order in which virtual machines are scaled-in.
	
# Implement Azure App Service plans

In Azure App Service, an application runs in an Azure App Service plan. An App Service plan defines a set of compute resources for a web application to run. The compute resources are analogous to a server farm in conventional web hosting. One or more applications can be configured to run on the same computing resources (or in the same App Service plan).

### Things to know about App Service plans
	
Let's take a closer look at how to implement and use an App Service plan with your virtual machines.

- When you create an App Service plan in a region, a set of compute resources is created for the plan in the specified region. Any applications that you place into the plan run on the compute resources defined by the plan.

- Each App Service plan defines three settings:

  - Region: The region for the App Service plan, such as West US, Central India, North Europe, and so on.
  - Number of VM instances: The number of virtual machine instances to allocate for the plan.
  - Size of VM instances: The size of the virtual machine instances in the plan, including Small, Medium, or Large.
	
- You can continue to add new applications to an existing plan as long as the plan has enough resources to handle the increasing load.

### How applications run and scale in App Service plans
	
The Azure App Service plan is the scale unit of App Service applications. Depending on the pricing tier for your Azure App Service plan, your applications run and scale in a different manner. If your plan is configured to run five virtual machine instances, then all applications in the plan run on all five instances. If your plan is configured for autoscaling, then all applications in the plan are scaled out together based on the autoscale settings.

Here's a summary of how applications run and scale in Azure App Service plan pricing tiers:

- Free or Shared tier:

  - Applications run by receiving CPU minutes on a shared virtual machine instance.
  - Applications can't scale out.
- Basic, Standard, Premium, or Isolated tier:

  - Applications run on all virtual machine instances configured in the App Service plan.
  - Multiple applications in the same plan share the same virtual machine instances.
  - If you have multiple deployment slots for an application, all deployment slots run on the same virtual machine instances.
  - If you enable diagnostic logs, perform backups, or run WebJobs, these tasks use CPU cycles and memory on the same virtual machine instances.
	
### Things to consider when using App Service plans
Review the following considerations about using Azure App Service plans to run and scale your applications. Think about what conditions might apply to running and scaling the hotel website.

- Consider cost savings. Because you pay for the computing resources that your App Service plan allocates, you can potentially save money by placing multiple applications into the same App Service plan.

- Consider multiple applications in one plan. Create a single plan to support multiple applications, to make it easier to configure and maintain shared virtual machine instances. Because the applications share the same virtual machine instances, you need to carefully manage your plan resources and capacity.

- Consider plan capacity. Before you add a new application to an existing plan, determine the resource requirements for the new application and identify the remaining capacity of your plan.

   >**Note**: Overloading an App Service plan can potentially cause downtime for new and existing applications.

- Consider application isolation. Isolate your application into a new App Service plan when:
  - The application is resource-intensive.
  - You want to scale the application independently from the other applications in the existing plan.
  - The application needs resource in a different geographical region.

### Things to know about autoscale
Let's take a closer look at how to use autoscale for your Azure App Service plan and applications.

- To use autoscale, you specify the minimum, and maximum number of instances to run by using a set of rules and conditions.

- When your application runs under autoscale conditions, the number of virtual machine instances are automatically adjusted based on your rules. When rule conditions are met, one or more autoscale actions are triggered.

- An autoscale setting is read by the autoscale engine to determine whether to scale out or in. Autoscale settings are grouped into profiles.

- Autoscale rules include a trigger and a scale action (in or out). The trigger can be metric-based or time-based.
	
  - Metric-based rules measure application load and add or remove virtual machines based on the load, such as "do this action when CPU usage is above 50%." Example metrics include CPU time, Average response time, and Requests.

  - Time-based rules (or, schedule-based) allow you to scale when you see time patterns in your load and want to scale before a possible load increase or decrease occurs. An example is "trigger a webhook every 8:00 AM on Saturday in a given time zone."

- The autoscale engine uses notification settings.

A notification setting defines what notifications should occur when an autoscale event occurs based on satisfying the criteria of an autoscale setting profile. Autoscale can notify one or more email addresses or make calls to one or more webhooks.

### Things to consider when configuring autoscale
There are several considerations to keep in mind when you configure autoscale for your Azure App Service plan and applications.

- Minimum instance count. Set a minimum instance count to make sure your application is always running even when there's no load.

- Maximum instance count. Set a maximum instance count to limit your total possible hourly cost.

- Adequate scale margin. Make sure your maximum and minimum instance count values are different, and set an adequate margin between the two values. You can automatically scale between the minimum and maximum by using rules you create.

- Scale rule combinations. Always use a scale-out and scale-in rule combination that performs an increase and decrease. If you don't set a scale-out rule, your application might fail, or performance might degrade under increased loads. If you don't set a scale-in rule, you can experience unnecessary and extensive costs when the load decreases.

- Metric statistics. Carefully choose the appropriate statistic for your diagnostic metrics, including Average, Minimum, Maximum, and Total.

Default instance count. Always select a safe default instance count. The default instance count is important because autoscale scales your service to the count you specify when metrics aren't available.

- Notifications. Always configure autoscale notifications. It's important to maintain awareness of how your application is performing as the load changes.
	
# Create deployment slots

When you deploy your web app, web app on Linux, mobile backend, or API app to Azure App Service, you can use a separate deployment slot instead of the default production slot.

### Things to know about deployment slots
Let's take a closer look at the characteristics of deployment slots.

- Deployment slots are live apps that have their own hostnames.

- Deployment slots are available in the Standard, Premium, and Isolated App Service pricing tiers. Your app needs to be running in one of these tiers to use deployment slots.

- The Standard, Premium, and Isolated tiers offer different numbers of deployment slots.

- App content and configuration elements can be swapped between two deployment slots, including the production slot.

### Things to consider when using deployment slots
- Consider validation. You can validate changes to your app in a staging deployment slot before swapping the app changes with the content in the production slot.

- Consider reductions in downtime. Deploying an app to a slot first and swapping it into production ensures that all instances of the slot are warmed up before being swapped into production. This option eliminates downtime when you deploy your app. The traffic redirection is seamless, and no requests are dropped because of swap operations. The entire workflow can be automated by configuring Auto swap when pre-swap validation isn't needed.

- Consider restoring to last known good site. After a swap, the slot with the previously staged app now has the previous production app. If the changes swapped into the production slot aren't as you expected, you can perform the same swap immediately to return to your "last known good site."

- Consider Auto swap. Auto swap streamlines Azure DevOps scenarios where you want to deploy your app continuously with zero cold starts and zero downtime for app customers. When Auto swap is enabled from a slot into production, every time you push your code changes to that slot, App Service automatically swaps the app into production after it's warmed up in the source slot. Auto swap isn't currently supported for Web Apps on Linux.

### Things to know about creating deployment slots
Let's review some details about how deployment slots are configured.

- New deployment slots can be empty or cloned.

- Deployment slot settings fall into three categories:

  - Slot-specific app settings and connection strings (if applicable)
  - Continuous deployment settings (when enabled)
  - Azure App Service authentication settings (when enabled)
- When you clone a configuration from another deployment slot, the cloned configuration is editable. Some configuration elements follow the content across the swap. Other slot-specific configuration elements stay in the source slot after the swap.

### Swapped settings versus slot-specific settings
The following table lists the settings that are swapped between deployment slots, and settings that remain in the source slot (slot-specific). As you review these settings, consider which features are required for your App Service apps.

|Swapped settings| Slot-specific settings|
|-----------------------------------------------------------------------------------|-----------------------------------------------------------|	
|<ul><li>General settings, such as framework version, 32/64-bit, web sockets</li><li>App settings *</li><li>Connection strings *</li><li>Handler mappings</li><li>Public certificates</li><li>WebJobs content</li><li>Hybrid connections **</li><li>Service endpoints **</li><li>Azure Content Delivery Network **</li><li>Path mapping</li>|<ul><li>Custom domain names</li><li>Non-public certificates and TLS/SSL settings</li><li>Scale settings</li><li>Always On</li><li>IP restrictions</li><li>WebJobs schedulers</li><li>Diagnostic settings</li><li>Cross-origin resource sharing (CORS)</li><li>Virtual network integration</li><li>Managed identities</li><li>Settings that end with the suffix _EXTENSION_VERSION</li>|

	
* Setting can be configured to be slot-specific.

** Feature isn't currently available.

# Create custom domain names

When you create a web app, Azure assigns the app to a subdomain of azurewebsites.net. Suppose your web app is named contoso. Azure creates a URL for your web app as contoso.azurewebsites.net. Azure also assigns a virtual IP address for your app. For a production web app, you might want users to see a custom domain name.
	
Configure a custom domain name for your app
There are three steps to create a custom domain name. The following steps outline how to create a domain name in the Azure portal.

   >**Note**: To map a custom DNS name to your app, you need a paid tier of an App Service plan for your app.

1. Reserve your domain name. If you haven't registered for an external domain name for your app, the easiest way to set up a custom domain is to buy one directly in the Azure portal. (This name isn't the Azure assigned name of \*.azurewebsites.net.) The registration process enables you to manage your web app's domain name directly in the Azure portal instead of going to a third-party site. Configuring the domain name in your web app is also a simple process in the Azure portal.

2. Create DNS records to map the domain to your Azure web app. The Domain Name System (DNS) uses data records to map domain names to IP addresses. There are several types of DNS records.

- For web apps, you create either an A (Address) record or a CNAME (Canonical Name) record.

  - An A record maps a domain name to an IP address.
  - A CNAME record maps a domain name to another domain name. DNS uses the second name to look up the address. Users still see the first domain name in their browser. As an example, you could map contoso.com to your webapp.azurewebsites.net URL.
	
- If the IP address changes, a CNAME entry is still valid, whereas an A record must be updated.

- Some domain registrars don't allow CNAME records for the root domain or for wildcard domains. In such cases, you must use an A record.

3. Enable the custom domain. After you have your domain and create your DNS record, use the Azure portal to validate your custom domain and add it to your web app. Be sure to test your domain before publishing.
	
# Back up and restore your App Service app

### Things to know about Backup and Restore

To use the Backup and Restore feature, you need the Standard or Premium tier App Service plan for your app or site.

You need an Azure storage account and container in the same subscription as the app to back up.

Azure App Service can back up the following information to the Azure storage account and container you configured for your app:

App configuration settings
File content
Any database connected to your app (SQL Database, Azure Database for MySQL, Azure Database for PostgreSQL, MySQL in-app)
	
- Consider full backups. Do a full backup to easily save all configuration settings, all file content, and all database content connected with your app or site.

When you restore a full backup, all content on the site is replaced with whatever is in the backup. If a file is on the site, but not in the backup, the file is deleted.

- Consider partial backups. Specify a partial backup so you can choose exactly which files to back up.

When you restore a partial backup, any content located in an excluded folder or file is left as-is.

- Consider browsing back-up files. Unzip and browse the Zip and XML files associated with your backup to access your backups. This option lets you view the content without actually performing an app or site restore.

- Consider firewall on back-up destination. If your storage account is enabled with a firewall, you can't use the storage account as the destination for your backups.
	
|Compare| Containers| Virtual machines|
|-------|-----------|-----------------|	
|Isolation| A container typically provides lightweight isolation from the host and other containers, but a container doesn't provide as strong a security boundary as a virtual machine.| A virtual machine provides complete isolation from the host operating system and other virtual machines. This separation is useful when a strong security boundary is critical, such as hosting apps from competing companies on the same server or cluster.
|Operating system| Containers run the user mode portion of an operating system and can be tailored to contain just the needed services for your app. This approach helps you use fewer system resources.| Virtual machines run a complete operating system including the kernel, which requires more system resources (CPU, memory, and storage).
|Deployment| You can deploy individual containers by using Docker via the command line. You can deploy multiple containers by using an orchestrator such as Azure Kubernetes Service.|	You can deploy individual virtual machines by using Windows Admin Center or Hyper-V Manager. You can deploy multiple virtual machines by using PowerShell or System Center Virtual Machine Manager.
|Persistent storage| Containers use Azure Disks for local storage for a single node, or Azure Files (SMB shares) for storage shared by multiple nodes or servers.|	Virtual machines use a virtual hard disk (VHD) for local storage for a single machine, or an SMB file share for storage shared by multiple servers.
|Fault tolerance| If a cluster node fails, any containers running on the node are rapidly recreated by the orchestrator on another cluster node.| Virtual machines can fail over to another server in a cluster, where the virtual machine's operating system restarts on the new server.|
