---
title: "PowerShell Hashtables"
excerpt: "Everything you always wanted to know about PowerShell Hashtables  and never dared to ask"
categories:

  - PowerShell
tags:
  - PowerShell
  - Hash
  - PowerShell Core
  - Hashtables
  - PowerShell Arrays
  - Dictionaries
---

## PowerShell Hashtables

**Hashtables**, *Dictionaries*, *Associative Arrays* or simply *Hash* are an invaluable tool in PowerShell development even if sometimes they pose some challenges for beginners.

I love **hashtables** for their speed and flexibility or simply because they played a huge role in the first *serious* automation script I wrote back at the time.

## Hashtables - An Introduction

In this section I will give you a quick rundown of *PowerShell Hashtables* how they are created, how to add elements and how to update them.

### Creating a Hashtable

At the very core an *hashtable* is nothing else than a collection of items of the same or different type. Does it sound familiar? If it rings a bell probably it has to do with the fact the same concept applies to **[arrays](https://pscustomobject.github.io/powershell/PowerShell-Arrays/)** with a key difference. A **hashtable** stores each value (or object) using a **key**.

Here's a an example which should make this clearer:

```powershell
# Array
[array]$myArray = @(
'Element 1',
'Element 2',
'Element 3'
)

# Hashtable
[hashtable]$myHashtable = @{
    1 = 'Element 1';
    2 = 'Element 2';
    3 = 'Element 3'
}

# Empty Hashtable
[hashtable]$myOtherHashtable = @{ }
```

Notice how arrays use *parentheses* while hashtables use *braces* and also a hashtable organises values in pairs *key/value*.

### Adding Elements to a Hashtable

Hashtables are *dynamic* in the sense you can easily add elements to them out of the box, syntax is really simple

```powershell
# Create hashtable
[hashtable]$myHashTable = @{ }

# Add element to hash
$myHashTable.Add( 'John', 31 )
$myHashTable.Add( 'Jim', 42 )
$myHashTable.Add( 'Jane', 35 )

$myHashTable

# Output
Name                           Value
----                           -----
2                              Element 2
1                              Element 1
```

**Note:** Remember that each *key* in the hashtable must be **unique**

### Accessing Elements in a Hashtable

Similar to adding new elements even accessing existing ones is really easy. Let's assume we want to store *Jane's* age in a variable, we already know it is in our hashtable so we would use the following syntax

```powershell
$age = $myHashTable['Jane']

$age

35
```

### Update Elements in a Hashtable

Hashtables support updating values stored with the following syntax

```powershell
$myHashTable['Jane'] = 36
```

As you can see we use the same *brackets* syntax that is used to specify hashtable key follows by the value we want to assign to it, this of this as accessing an *array* via its index value.

## Hashtables Practical Use

Now that we've introduced most important concepts about *hashtables* we can go through their usage and some advanced scenarios.

### Hashtables as Lookup Tables

One common use for hashtables is to create lookup tables, here's a practical example of this

```powershell
# Create an hashtable with OUs DN
[hashtable]$ouPath = @{
    'New York' = 'OU=New York,DC=Domain,DC=com'
    'Los Angeles'   = 'OU=Los Angeles,DC=Domain,DC=com'
    'Chicago'  = 'OU=Chicago,DC=Domain,DC=com'
}

# Define user City
[string]$userCity = 'New York'

Move-AdUSer -Identity $user -DestinationPath $ouPath[$userCity]
```

As you can see the hashtable contains DNs for some OUs in our AD structure. We assign the *$userCity* variable a value and finally use that to find correspondent DN for the destination OU. 

The above example is based on a real automation script I have delivered for a customer. Help Desk operator would select a value from a dropdown in a web interface and a back-end script would move the user to the appropriate OU.

### Nested Hashtables

Hashtables can also be nested inside other hashtables this can be useul when modeling data around the object we need to represent and manage, extending the previous example we could create the following nested hashtable:

```powershell
# Create nested hashtable
[hashtable]$ouPath = @{
    'New York' = 'OU=New York,DC=Domain,DC=com'
    'Los Angeles'   = 'OU=Los Angeles,DC=Domain,DC=com'
    'Chicago'  = 'OU=Chicago,DC=Domain,DC=com'
    'Location Details' = @{
      'countryCode' = '840'
      'ISO Code' = 'US'
    }
}

# Display content
$ouPath

Name                           Value
----                           -----
Los Angeles                    OU=Los Angeles,DC=Domain,DC=com
Location Details               {ISO Code, countryCode}
New York                       OU=New York,DC=Domain,DC=com
Chicago                        OU=Chicago,DC=Domain,DC=com

# Display sub hash
$ouPath.'Location Details'

Name                           Value
----                           -----
ISO Code                       US
countryCode                    840

$ouPath.'Location Details'.'countryCode'
```

### Exporting Hashtables to CSV

One task in the past I've struggled with is exporting hashtables content to CSV, if you need to carry on such task this can be accomplished like this

```powershell
$ouPath | ForEach-Object{ [pscustomobject]$_ } | Export-CSV -Path $csvPath
```

### Converting Hashtables to string

When using a [log function](https://github.com/PsCustomObject/New-LogEntry) that writes data to a log file on disk we cannot simply output the hashtable to the log message but first need to convert it to a proper string format which can be achieved like this

```powershell
# Convert hashtable to string format
[string]$hashString = ($ouPath | Out-String).Trim()
```

This will output hashtable content to the *Out-String* cmdlet and save content to a variable that can be then used to write content to log file.

### Closing Notes

Hashtables are a fascinating and very powerful topic, there are some more advanced topics that I did not cover in this post like converting from or to json objects but I will save that for another article.
