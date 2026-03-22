Azure Cloud Shell and Local PowerShell VM Test

Goal

Understand the basic idea of Azure Cloud Shell, then test doing similar Azure work from local PowerShell on my MacBook instead of only using the browser.

Lesson Summary

Azure Cloud Shell is a browser-based terminal for Azure.
It can run PowerShell or Bash.
It works from any device.
It does not need local setup.
The session is temporary.
Files saved in CloudDrive stay.

Simple takeaway: use Azure from anywhere and keep important files in CloudDrive.

Why I tested more than the lesson

The lesson focused on using the web shell, but I wanted to test Azure from my local MacBook PowerShell session as well.
I had not used local command line on my Mac in a while, so this was also a good way to refresh that workflow.

Lab Setup

Device: MacBook
OS: macOS
Shell: PowerShell 7
Azure tool: Az PowerShell module
Initial region tested: australiaeast

Local PowerShell setup

Install-Module Az

![Local Azure module install and resource group check](./Photos/001Installazlocaly.jpeg)
Installed the Azure module locally, signed in, and confirmed I could query Azure resources.

Authentication testing

Connect-AzAccount
Connect-AzAccount -UseDeviceAuthentication

![Device authentication](./Photos/002Difrentauth.jpeg)
Used device authentication when normal sign-in redirected me to the default browser.

Why this mattered

I had seen this authentication flow before when testing MFA-related work tasks in Entra, so this connected the lab to something I had already dealt with in a real support context.

Azure access check

Get-AzResourceGroup

![Local Azure module install and resource group check](./Photos/001Installazlocaly.jpeg)
Listed existing resource groups to confirm the local PowerShell session was connected to Azure correctly.

VM deployment attempt from local PowerShell

$rg = "rg-b1s-linux"
$loc = "australiaeast"
$vm = "vm-b1s-01"

New-AzResourceGroup -Name $rg -Location $loc

New-AzVm `
-ResourceGroupName $rg `
-Name $vm `
-Location $loc `
-Image "Ubuntu2204" `
-Size "Standard_B1s" `
-Credential $cred `
-OpenPorts 22

![Local VM deployment failure](./Photos/003VMno1.png)
Created a test resource group and tried to deploy a small Ubuntu VM, but the selected VM size was not available in that region.

Main issue found

Standard_B1s was not available in australiaeast.
This blocked the cheapest VM test I wanted to use first.

What I wanted to do next

My next plan was to get the VM working and then test basic connectivity from it, such as pinging my Pi server.

Execution policy test on macOS

Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

![Execution policy not supported](./Photos/006Notsupported.png)
Tested execution policy and confirmed that this command is not supported on macOS PowerShell.

Cloud Shell comparison

![VM attempt in Azure Cloud Shell](./Photos/newvm%20in%20web.png)
Retried the VM creation idea in Azure Cloud Shell to compare local PowerShell with the browser-based method.

What happened in Cloud Shell

Cloud Shell did not remove the Azure-side deployment problems.
I also ran into an SSH key name conflict there.

Cleanup and resource review

Get-AzResource | Select-Object Name, ResourceGroupName, ResourceType, Location
Remove-AzResourceGroup "rg-b1s-linux" -Force

![Resource cleanup](./Photos/004resourceDelete.png)
Checked what was left after the failed deployment and removed the test resource groups.

Why cleanup mattered

Even though the VM did not finish deploying, related resources were still created.
That made cleanup part of the test.

Saved deployment script

Get-Content ./deploy-pingvm.ps1

![Saved deployment script](./Photos/005Firstsavedscript.png)
Saved the deployment steps into a reusable PowerShell script so I could test again faster later.

Commands used

Install-Module Az
Connect-AzAccount
Connect-AzAccount -UseDeviceAuthentication
Get-AzResourceGroup
New-AzResourceGroup
New-AzVm
Get-AzResource | Select-Object Name, ResourceGroupName, ResourceType, Location
Remove-AzResourceGroup
Get-Content ./deploy-pingvm.ps1
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

Key findings

Azure can be managed from local PowerShell on macOS.
Device authentication is a useful fallback when browser-based sign-in is not ideal.
VM deployment depends on region and SKU availability.
Cloud Shell does not remove Azure-side limits such as SKU availability or SSH key naming issues.
Failed deployments can still leave resources behind and require cleanup.
Saving the workflow into a script makes repeat testing easier.

Result

I successfully installed Azure tools locally, authenticated to Azure, listed resource groups, created a test resource group, checked leftover resources, cleaned them up, and saved the deployment process into a reusable script.
The VM deployment itself did not complete because the selected VM size was not available in the region I tested.

Next step

Check VM size availability in PowerShell before deploying again and test another region that supports the Linux VM size I want.
