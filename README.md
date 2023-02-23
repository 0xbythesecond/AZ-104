# AZ-104
Notes to help me prepare for the Microsoft AZ-104


Connect
When you're working with a local install of Azure PowerShell, you'll need to authenticate before you can execute Azure commands. The `Connect-AzAccount` cmdlet prompts for your Azure credentials, then connects to your Azure subscription. It has many optional parameters, but if all you need is an interactive prompt, you don't need any parameters:

Use the `Get-AzContext` cmdlet to determine which subscription is active.

Get a list of all subscription names in your account with the Get-AzSubscription command.

You can retrieve a list of all Resource Groups in the active subscription.
`Get-AzResourceGroup`
To get a more concise view, you can send the output from the Get-AzResourceGroup to the Format-Table cmdlet using a pipe '|'.
`Get-AzResourceGroup` | `Format-Table`

You can create resource groups by using the New-AzResourceGroup cmdlet. You must specify a name and location. The name must be unique within your subscription. The location determines where the metadata for your resource group will be stored (which may be important to you for compliance reasons). 

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
You can supply these parameters directly to the cmdlet as shown in the previous example. Alternatively, you can use other cmdlets to configure the virtual machine, such as Set-AzVMOperatingSystem, Set-AzVMSourceImage, Add-AzVMNetworkInterface, and Set-AzVMOSDisk.

Here's an example that strings the Get-Credential cmdlet together with the -Credential parameter:

    | Command | Description |
    | --- | --- |
    | Remove-AzVM | Deletes an Azure VM |
    | Start-AzVM | 	Start a stopped VM |
    | Stop-AzVM | Stop a running VM |
    | Restart-AzVM | Restart a VM |
    | Update-AzVM	| Updates the configuration for a VM |
