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
