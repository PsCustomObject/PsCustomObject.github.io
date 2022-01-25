---
title: "Variables in Azure Automation"
excerpt: "In this post we will explore how we can use variables in Azure Automation. We will explore both encrypted and unencrypted variables and how we can leverage them in our scripts."
categories:
  - PowerShell
  - Azure

tags:
  - PowerShell
  - Office365
  - Azure
  - Azure Automation

toc: true
header:
    teaser: "/assets/images/Azure_Automation_Logo.png"
---

## Azure Automation Variables as Shared Resources

Shared Resources in Azure Automation allow us to reuse credentials, modules, schedules, connections, certificates and **variables** which will be will be the main focus of the post.

To better understand the importance of shared resources and *variables in Azure Automation* let's go through a practical example. I have a runbook pulling data from a web service which requires to reference an API secret as part of the process to obtain an authentication token.

When running the script through a scheduled task or through on-prem System Center Orchestrator you would either need to store the key in plain text in the script itself or use methods described in my post *[Store Credentials in PowerShell Script](https://pscustomobject.github.io/powershell/howto/Store-Credentials-in-PowerShell-Script/)*

```powershell
# Plain text
[strint]$apiSecrect = '14??!==AbZyC78mk'

# Use New-StringDecryption cmdlet from IT-ToolBox module
New-StringDecryption -EncryptedString $apiSecret
```

While both approaches will work that's not without drawbacks implying security considerations. Another consideration is the fact secret key at some point could change and, if a large number of scripts/runbooks, is using it we would need to update code all of solutions using it. This is were *Azure Automation variables* come into play.

## Azure Automation Variables Types

Azure Automations supports two types of variables *Encrypted* and *Unenrcrypted* with the following types being supported:

- Integers
- Strings
- DateTime
- Boolean
- Null

Full documentation it is available [here](https://docs.microsoft.com/en-us/azure/automation/shared-resources/variables).

### Unencrypted variables

As the name implies an *unencrypted* variables are stored in Azure with their values being *visible* and to both runbooks/scripts and administrators.

An unencrypted variable can be created in the Azure portal going to **[Automation Account] / [Shared Resources] / [Variables]** and selecting **Add Variable** being sure to select *No* under the **Encrypted** section.

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure_Automation_Unencrypted_Variable.png">
  <img src="/assets/images/Azure_Automation_Unencrypted_Variable.png"></a>
</figure>

**Note:** Typo in the above variable value is intended.
{: .notice--primary}

The same operation can be achieved via PowerShell assuming the **Az** module is installed on the system:

```powershell
$paramNewAzAutomationVariable = @{
    ResourceGroupName     = $azResourceGroup
    AutomationAccountName = $azAccount
    Name                  = 'Test Variable'
    Encrypted             = $false
    Value                 = 'Test Value'
}

New-AzAutomationVariable @paramNewAzAutomationVariable

# Output
Value                 : Test Value
Encrypted             : False
ResourceGroupName     : Dev-PsCustomObject-AutomationRsg
AutomationAccountName : Dev-PsCustomObject
Name                  : Test Variable
CreationTime          : 26.09.2020 17:53:23 +02:00
LastModifiedTime      : 26.09.2020 17:53:23 +02:00
Description           :
```

Once variable has been created it will be visible in the console:

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure-Automatin-Unencrypted-Variables-Console.png">
  <img src="/assets/images/Azure-Automatin-Unencrypted-Variables-Console.png"></a>
</figure>

As you can see values of the variable is visible both in the PowerShell output and the *Value* column of the variables blade in Azure Portal. We will discuss this in more detail in a minute.

### Encrypted variables

Creation *encrypted* variables is identical in terms of steps in the console and via PowerShell simply requires us to specify the *-Encrypted $True* parameter.

```powershell
$paramNewAzAutomationVariable = @{
    ResourceGroupName     = $azResourceGroup
    AutomationAccountName = $azAccount
    Name                  = 'Test Encrypted Variable'
    Encrypted             = $true
    Value                 = 'Test Value'
}

New-AzAutomationVariable @paramNewAzAutomationVariable

# Output
Value                 :
Encrypted             : True
ResourceGroupName     : Dev-PsCustomObject-AutomationRsg
AutomationAccountName : Dev-PsCustomObject
Name                  : Test Encrypted Variable
CreationTime          : 26.09.2020 18:51:53 +02:00
LastModifiedTime      : 26.09.2020 18:51:53 +02:00
Description           :
```

As you can see when creating an encrypted variable Value is *omitted* in the output, in the *variables* blade it will be displayed like this

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure_Automation_Encrypted_Variable.png">
  <img src="/assets/images/Azure_Automation_Encrypted_Variable.png"></a>
</figure>

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Add_Application.png">
  <img src="/assets/images/Add_Application.png"></a>
</figure>

This is the expected behaviour as encrypted variables are *secured* with a unique key generated for each **automation account**. It goes alone encrypted variables are more secure in nature but it has to be kept in mind that, once created, azure automation variables have been created **values cannot be seen only updated**.

### Update and Retrieve Azure Automation variable values

Now that we know how to create azure automation variables lets see how to work with them and update their values. Recall I had a typo in the unencrypted variable, trailing '[' character, let's fix that from the console

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure_Automation_Update_Variable_Value.png">
  <img src="/assets/images/Azure_Automation_Update_Variable_Value.png"></a>
</figure>

Or from PowerShell

```powershell
$paramSetAzAutomationVariable = @{
    ResourceGroupName     = $azResourceGroup
    AutomationAccountName = $azAccount
    Name                  = 'Test Variable'
    Value                 = 'Setting new value from PowerShell'
    Encrypted             = $False
}

Set-AzAutomationVariable @paramSetAzAutomationVariable

Value                 : Setting new value from PowerShell
Encrypted             : False
ResourceGroupName     : Dev-PsCustomObject-AutomationRsg
AutomationAccountName : Dev-PsCustomObject
Name                  : Test Variable
CreationTime          : 26.09.2020 17:53:23 +02:00
LastModifiedTime      : 26.09.2020 19:08:28 +02:00
Description           : Fixed typo in value
```

When working with an encrypted variable things will be slightly different. **Encrypted variables can only have their value updated but never shown** to do so simply click the *Edit value* button and then save it

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure_Automation_Update_Encrypted_Variable.png">
  <img src="/assets/images/Azure_Automation_Update_Encrypted_Variable.png"></a>
</figure>

From PowerShell command will not be much different but, again, we will not get back the value from the cmdlet

```powershell
$paramSetAzAutomationVariable = @{
    ResourceGroupName     = $azResourceGroup
    AutomationAccountName = $azAccount
    Name                  = 'Test Encrypted Variable'
    Value                 = 'Setting new value from PowerShell'
    Encrypted             = $true
}

Set-AzAutomationVariable @paramSetAzAutomationVariable

Value                 :
Encrypted             : True
ResourceGroupName     : Dev-PsCustomObject-AutomationRsg
AutomationAccountName : Dev-PsCustomObject
Name                  : Test Encrypted Variable
CreationTime          : 26.09.2020 18:51:53 +02:00
LastModifiedTime      : 26.09.2020 19:14:46 +02:00
Description
```

## Getting Variable Values

Up to this point we've seen how to create and update Azure Automation variables, let's explore how to get values for configured variables.
*Az* module makes available a cmdlet for the purpose

```powershell
Get-AzAutomationVariable -ResourceGroupName $azureResourceGroup -AutomationAccountName $azureAccount -Name 'Test Variable'

Value                 : Setting new value from PowerShell
Encrypted             : False
ResourceGroupName     : Dev-PsCustomObject-AutomationRsg
AutomationAccountName : Dev-PsCustomObject
Name                  : Test Variable
CreationTime          : 26/09/2020 17:53:23 +02:00
LastModifiedTime      : 26/09/2020 19:08:28 +02:00
Description           : Fixed typo in value
```

**Note:** Omitting the *-Name* parameter will return all configured variables
{: .notice--primary}

As I mentioned it is not possible to retrieve values for encrypted variables as they're available within the runbook at runtime via the **Get-AutomationVariable** cmdlet. Using the *Test Encrypted Variable* as example I've created a small script that will show this behaviour

```powershell
$encryptedVariableValue = Get-AutomationVariable -Name 'Test Encrypted Variable'

Write-output "The encrypted variable value is: $encryptedVariableValue"
```

And here the result from Azure Automation test pane:

<figure>
  <a href="https://pscustomobject.github.io//assets/images/Azure_Automation_Encrypted_Variable_Runbook.png">
  <img src="/assets/images/Azure_Automation_Encrypted_Variable_Runbook.png"></a>
</figure>

As simple as that, the same snippet can be used in production script to safely store *secrets* in our scripts.