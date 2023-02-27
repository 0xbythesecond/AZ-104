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
$vm | Remove-AzNetworkInterface –Force
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
```
az find blob
```       
If you already know the name of the command you want, the --help argument for that command will get you more detailed information on the command, and a list of the available subcommands for a command group. So, with our storage example, here's how you can get a list of the subgroups and commands for managing blob storage:
```
az storage blob --help
```       
### Connect
Since you're working with a local install of the Azure CLI, you'll need to authenticate before you can execute Azure commands by using the Azure CLI login command.

``` 
 az login
```

After a successful sign-in, you'll be connected to your Azure subscription.

### Create
You'll often need to create a new resource group before you create a new Azure service, so we'll use resource groups as an example to show how to create Azure resources from the CLI.

The Azure CLI group create command creates a resource group. You must specify a name and location. The name must be unique within your subscription. The location determines where the metadata for your resource group will be stored. You use strings like "West US", "North Europe", or "West India" to specify the location; alternatively, you can use single-word equivalents, such as westus, northeurope, or westindia. The core syntax is:

```
az group create --name <name> --location <location>
```       
### Verify
For many Azure resources, the Azure CLI provides a list subcommand to view resource details. For example, the Azure CLI group list command lists your Azure resource groups. This is useful to verify whether the resource group was successfully created:

```
az group list
```       
To get a more concise view, you can format the output as a simple table:

```
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
