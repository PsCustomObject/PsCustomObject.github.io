---
title: "Check function parameters with PSBoundParameters"
excerpt: "Or how do I know which parameters have been passed to a function?"
categories:

  - PowerShell Best Practice
  - PowerShell
tags:
  - Tutorial
  - HowTo
  - PowerShell
---

## PSBoundParameters Automatic variable

Automatic variable *PSBoundParameters* is a special [hashtable](https://pscustomobject.github.io/powershell/PowerShell-HashTables/) containing all parameters that are passed to a script or a **function**.

*PSBoundParameters* can be used to call a subordinate function or cmdlet passing the same parameters.
PowerShell will automatically [splat](https://pscustomobject.github.io/powershell%20tips/PowerShell-Command-Splatting/) the hashtable's values instead of having to type each of the parameters individually.

## Get function parameters - The bad way

Let's start defining a dummy function that we will use throughout the article.

```powershell
# Wish it was so easy :-)
function Start-WorldFix
{
  Param
  (
    [switch]$Verbose,
    [string]$IssueName
  )

  Write-Host -Message 'Fixing the world - Please wait...'
}
```

Despite the name the above function will do absolutely nothing but we specified two **parameters** that can optionally be used together with the function.

*How do I know which, if any, parameters have been used?* you are maybe wondering.

A lot of time, and yes I did this a lot as well, this is what's in the code:

```powershell
if ($Verbose)
{
  Write-Verbose -Message 'You used the -Verbose paraemter'
}
```

While this will work, as the title implies, there are better alternatives. 

## Get function parameters - PSBoundParameters the good way

*PSBoundParameters* being an hashtable allows us a lot of flexibility in checking if a parameter has been specified.

```powershell
# Method 1
switch ($PSBoundParameters.Keys)
{
  'Verbose'
  {
    Write-Verbose -Message 'You used the -Verbose paraemter'
  }
  'IssueName'
  {
    Write-Host -Message 'You used the -IssueName parameter'
  }
}

# Method 2
if ($PSBoundParameters['Verbose'])
{
  # Take action
}

# Method 3
if ($PSBoundParameters.ContainsKey('Verbose'))
{
  # Take action
}
```

Personally I'm fan, abuser would maybe more appropriate, of *method 1* as it makes easy for me to dynamically build commands but any other of the illustrate examples will equally do.

Remember **$PSBoundParameters** is an hashtable and we can use this to our advantage like

```powershell
$PSBoundParameters.Add('Key','Value')
```

This is specifically handy when calling sub functions or want to alter the flow of the function code.

## PSBoundParameters - Print parameters

If all we want to accomplish is printing which parameter have been specified for a function this is easily accomplished with

```powershell
foreach ($boundParam in $PSBoundParameters.GetEnumerator())
{
  'Key={0} Value={1}' -f $boundParam.Key, $boundParam.Value
}
```

Or even simply:

```powershell
$PSBoundPArameters
```
