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


