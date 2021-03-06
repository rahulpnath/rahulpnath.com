---
layout: post
title: "Azure Key Vault and Powershell Module Version"
date: 2015-01-17 18:05:53 
comments: true
categories:
- Azure
- Azure Key Vault
tags: 
- azure
keywords: "azure, azure key vault, powershell"
description: "Details out on the error, 'Please install Azure Powershell module version 0.8.13 or newer.' when trying Azure Key vault."
---

I was trying out the public preview of the  Azure Key Vault service that has been released recently. While following the steps as mentioned in their [blog](http://blogs.technet.com/b/kv/archive/2015/01/08/azure-key-vault-making-the-cloud-safer.aspx), came across the below error when trying the '*New-AzureKeyVault*' command.

> Please install Azure Powershell module version 0.8.13 or newer.

I did have the [latest powershell for azure](http://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/#Install) installed, but still this error is thrown. 

<img class="center" alt="azure powershell installed version" src="/images/azure_powershell_installed.png" />

Exploring the [powershell scripts for key vault](http://msdn.microsoft.com/library/dn868052.aspx), below is where the error is thrown from '*Common.ps1*'. The script looks for the [AzureResourceManager](http://msdn.microsoft.com/en-us/library/dn654592.aspx), which got introduced in poweshell version 0.8.0 and lets you manage resources in a completely different way. 
``` powershell
Function Azure-Version-Check{
    $expectedMinVersion = New-Object -TypeName System.Version -ArgumentList "0.8.13"

    $azureModule = Get-Module AzureResourceManager
            
    if ((-not $azureModule) -or ($azureModule.Version -lt $expectedMinVersion))
    {
        Throw 'Please install Azure Powershell module version 0.8.13 or newer.'
    }    
}
```

> When you use the Azure PowerShell cmdlets, the Azure module is imported into the session by default. To remove the Azure module from the session and import the AzureResourceManager and AzureProfile modules, use the Switch-AzureMode cmdlet. 

This is what is exactly causing the issue, we need to switch to use the azure resource manager. Running the below command and trying the '*New-AzureKeyVault*' command works like a charm

``` powershell
Switch-AzureMode -Name AzureResourceManager
```

*You would face this issue only if you started trying out the steps as mentioned in the [azure key vault blog](http://blogs.technet.com/b/kv/archive/2015/01/09/azure-key-vault-step-by-step.aspx)(as I did), since the steps in the [documentation site](http://azure.microsoft.com/en-us/documentation/articles/key-vault-get-started/)* is updated with the above step.*  
