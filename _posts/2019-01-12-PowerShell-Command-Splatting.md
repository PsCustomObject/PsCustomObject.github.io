---
title: "PowerShell - Command Splatting"
excerpt: "What is command splatting, how to use it and implement dynamic parameters"
categories:
  - PowerShell Tips
tags:
  - PowerShell
  - Splatting
  - PowerShell Core
  - Command Splatting
---

## PowerShell Command Splatting

When using a PowerShell cmdlet we can use parameters to alter the way it will carry on its assigned task. There are situations where it is desirable using a parameter rather than another depending on the result of another cmdlet output or function return value.

What I see to often is something like this:

```powershell
# Check return value from function
if ($userCity)
{
  Set-AdUser -Identity $userIdentity -Description "$userCity - Office" -Server $adDsServer
}
else
{
  Set-AdUser -Identity $userIdentity -Description 'Office not defined for user' -Server $adDsServer
}
```

In the above example if the *$userCity* variable holds a value we set user description with the office name, if not set a default value.

## PowerShell Splatting for Code readability

Technically speaking nothing is wrong with the above example, it will work just fine. What if we could make the code more readable while typing less?

**Command splatting**  is, quoting from [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-5.1),

> Splatting is a method of passing a collection of parameter values to a command as unit. PowerShell associates each value in the collection with a command parameter. Splatted parameter values are stored in named splatting variables, which look like standard variables, but begin with an At symbol (@) instead of a dollar sign ($). The At symbol tells PowerShell that you are passing a collection of values, instead of a single value.

Here's an example of how a *splatted* command can be written:

```powershell
# Copy file
$hashArgs = @{
  Path = 'File1.txt'
  Destination = 'File2.txt'
}

Copy-Item @hashArgs

# Non splatted equivalent
Copy-Item -Path 'File1.txt' -Destination 'File2.txt'
```

This is a simple example but shows how splatting a command can make code more easier to read, this when dealing with large scripts and/or long commands can make a huge difference.

Going back to our original example we can rewrite the command like this:

```powershell
# Set user Description
$setUserHash = @{
  Identity = $userIdentity
  Description = $userCity
  Server = $adServer  
}

Set-AdUser @setUserHash
```

## PowerShell Splatting and Dynamic Parameters

Using splatted commands has another big advantage, it allows us to implement *dynamic parameters* that is building our command based on output of a function or value of a variable.

In our *Set-AdUser* example we used an *if else* statement writing twice the same command with a slight variation. What if we need to check for 4 or even 8 different conditions? We would end up writing the same command over and over just with a slight variation in parameters which, in addition to duplication, could lead to typos and errors.

This is where *command splatting* and *dynamic parameters* come into play. It is probably clear by now but splatting is using an **[hashtable](https://pscustomobject.github.io/powershell/PowerShell-HashTables/)** to build the parameter set and *hashtables* support elements addition.

Rather than using a complex *if else* chain or *switch* statement to build up our command we can rewrite our example like this

```powershell
# Define common parameters
[hashtable]$setUserHash = @{ }
$setUserHash.Add('Identity', $userIdentity)
$setUserHash.Add('Server', $adServer)

# Dynamic parameters
if ($userCity)
{
  $setUserHash.Add('Description', $userCity)
}
else
{
  $setUserHash.Add('Description', 'Office not defined for user')
}

# Execute command
Set-AdUser @setUserHash
```

This is a much more concise version of our code which leads to easier implementation and maintenance.

## Dynamic Parameters Pitfalls

I will close this post with a small tip to avoid a common mistake we all do. When using a *foreach* loop remember hashtables will only allow you to specify a key **once**.

For example if our hashtable already contains a *Description* key/parameter and we do something like this:

```powershell
foreach ($user in $matchingUsers)
{
  if ($userCity)
  {
    $setUserHash.Add('Description', $userCity)
  }
  else
  {
    $setUserHash.Add('Description', 'Office not defined for user')
  }
}
```

First iteration will go through just fine but on any subsequent iteration you will get an exception like:

```powershell
Exception calling "Add" with "2" argument(s): "Item has already been added. Key in dictionary: 'Description'  Key being added: 'Description'"
At line:1 char:1
+ $myHash.Add('Description',$userCity)
+ ~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
+ FullyQualifiedErrorId : ArgumentException
```

Solution is to simply remove any dynamic parameter defined as key in our hashtable like this:

```powershell
$myHash.Remove('Description')
```
