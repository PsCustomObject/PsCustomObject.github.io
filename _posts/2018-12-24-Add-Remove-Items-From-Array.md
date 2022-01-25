---
title: "PowerShell add or remove elements from an Array"
excerpt: "Learn how to create you can add and remove elements from a PowerShell array through the use of list objects."
categories:
  - PowerShell
tags:
  - PowerShell
  - Arrays
  - PowerShell Basics
  - HowTo

toc: true
---

This is a question I get rather frequently so I thought to write an article about it. As the title implies I will show how you can add or remove elements to a PowerShell array once it has been created.

## PowerShell Arrays

Let's start with the definition of an array as it comes from PowerShell documentation:

> An array is a data structure that is designed to store a collection of items. The items can be the same type or different types.
> Beginning in Windows PowerShell 3.0, a collection of zero or one object has some properties of arrays.

Creating an array in PowerShell is really simple and most probably you already did when, for example, getting a list of users from Active Directory

```powershell
$adUsers = Get-AdUser -Filter '*' -Server $adDomainController
```

The above will return an array of AD objects containing all users matching the used filter.

Of course you can initialise an empty array with the following syntax

```powershell
$myArray = @()

# Specify object type
[array]$myArray = @()
```

The above will initialise an empty array that we can, for example, fill with an AD query or adding static elements like this

```powershell
$myArray = (1,2,3,4,5)
```

The above will add elements 1 to 5 to *myArray* object instantiating a new object with length of **5**

## PowerShell Add Elements to an Array

We have see how to create and assign values to an array but what if we want to add a sixth element to *myArray*? If you try the following

```powershell
$myArray.Add(6)
```

It will fail with the following exception

```powershell
Exception calling "Add" with "1" argument(s): "Collection was of a fixed size."
At line:1 char:1
+ $myArray.Add(6)
+ ~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
+ FullyQualifiedErrorId : NotSupportedException
```

The above exception is thrown as a PowerShell array is a collection of fixed size and error is telling just that, it cannot be extended.

One common solution is to use the **+=** operator like for example

```powershell
$myArray += 6

# Print array's length
$myArray.length

6
```

When using the *+=* operator what happens under the hood is

- PowerShell creates a new array with the same elements as the old one **plus** the new item
- PowerShell will overwrite existing array with the new content

All of this is transparent to the user so you won't see any difference. 

## Add Elements to an Array - Enter ArrayList

If you want to avoid all the copying/moving data you can instantiate *myArray* as a **[ArrayList](https://docs.microsoft.com/en-us/dotnet/api/system.collections.arraylist?view=netframework-4.7.2)** which is dynamic and will allow you to add remove elements on the fly

```powershell
# Initialize object
$myArrayList = New-Object System.Collections.ArrayList($null)

# Add elements to list
[void]($myArrayList.Add(1))
[void]($myArrayList.Add(2))

# Print array length
2
```

Similarly you can remove elements from an ArrayList like this

```powershell
# Will corresponding item by index
$myArrayList.RemoveAt(1)
```

**Note:** Remove method will accept element's value so *1* in the above example refers to the value not the item's index.

>Cast to [void] to suppress *Add* method printing new array's length

### Create Array List the alternate way

The above example will call the *New-Object* method to instantiate a new *ArrayList* but this is relatively expensive in terms of computing resources so the above can be rewritten as

```powershell
[System.Collections.ArrayList]$myArray = @()
```

In addition of being shorter it has the added benefit of avoiding the negative performance hit of *New-Object* and optimizing code is always a good idea especially when dealing with larger scripts.

As an alternative you can cast PowerShell Array to *ArrayList*

```powershell
$myArray = [System.Collections.ArrayList]@()
```

## Closing Notes

When writing code, despite not being required by PowerShell, I try to alway declare the object's type as that is helpful to know, at any given moment, Properties and Methods supported by the object so I alway write

```powershell
[string]$myString = 'Some Text'
```

Rather than 

```powershell
$myString = 'Some Text'
```

This is not required by PowerShell but I find it helps code's readability and simplifies working with an object declared 800/900 lines up in the code without having to jump back to variable's declaration.
