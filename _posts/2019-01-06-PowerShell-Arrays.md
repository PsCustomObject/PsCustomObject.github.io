---
title: "PowerShell Arrays"
excerpt: "Everything you always wanted to know about PowerShell Arrays and never dared to ask"
categories:

  - PowerShell
tags:
  - PowerShell
  - Hash
  - PowerShell Core
  - Hashtables
  - PowerShell Arrays
  - Lists
---

**Arrays** are a data structure designed to store a collection of items that can be of the same or different type. *Arrays* are a fundamental tool in PowerShell and chances are you already used them even without knowing.

## Arrays - A collection of items

As already mentioned an array is a collection of items and can be declared like this

```powershell
# Declare and assign values to an array
$myArray = 22,5,10,8,12,9,80

# Create an empty array
$myEmptyArray = @()

# Range array with elements 0 to 9
$anotherArray = 100 .. 110
```

An array has a **length** properties which we can use to get the number of elements in the array, for example *$anotherArray* from the previous example will have a length of **10**

```powershell
# Get number of elements
$anotherArray.Length

11
```

## Accessing Elements in an Array - Index

All elements in an array have an **index** which is the position of the element within the array, it is important to remember that arrays are created and displayed according to the order you specified for example

```powershell
# Print content
$anotherArray

100
101
102
103
104
105
106
107
108
109
110
```

You can access each element of an array using its index number

```powershell
$anotherArray[6]

105
```

You probably already know or noticed this but PowerShell starts indexing an *array* at **0** so, in our example, the 6th element will be **105**

## Accessing Elements in an Array - Iteration

Another common way to access elements of an array is **looping** through them like this for example

```powershell
foreach ($index in $anotherArray)
{
    Write-Host $index
}

# Output
100
101
102
103
104
105
106
107
108
109
110
```

## Empty Arrays

Up to this point we have illustrated how to create and immediately assign a value to arrays PowerShell also supports creating *empty* arrays with the following syntax:

```powershell
# Create empty array
$myEmptyarray = @()

# Display length
$myEmptyarray.Length

0
```

This technique is useful when you want to instantiate a new **Array** but don't know the number of elements it will contain.  

Here's another example of iterating through an array that I'm sure anybody dealing with an Active Directory domain used at least once:

```powershell
# Create the array
$matchingUsers = @()

# Get all enabled users
$matchingUsers = Get-AdUser -Filter * -Server 'MyDc.myDomain.com'

# Iterate throguh users
foreach ($user in $matchingUsers)
{
    # Take some action
}
```

## Strongly Typed Arrays

PowerShell is not a strongly typed language meaning we don't need to declare the type of data a variable contains as PowerShell will try to guess this automatically:

```powershell
# PowerShell will automatically create the correct
# object type
$aString = 'This is string'
$anInt = 32

$aString.GetType()

IsPublic IsSerial Name   BaseType
-------- -------- ----   --------
True     True     String System.Object
   
$anInt.GetType()

IsPublic IsSerial Name   BaseType
-------- -------- ----   --------
True     True     Int32  System.ValueType
```

The same happens with *arrays* but we have the option to create a *strongly typed array* that is, an array that can contain only values of a particular type:

```powershell
[int32[]]$intArray = 1300,22,4578,8000
```

**Note:** In the above example the doubt square brackets **[ ]** after the type *int32* tell PowerShell what follows is an array.

## Manipulating Arrays Elements

As closing topic let's see how we can manipulate elements in an *array* like **adding**, **removing** and **updating** elements. 

### Change Elements Values in an Array

Let's say we want to update the first element of the **$anotherArray** we previously created with a value of *200* this can be accomplished like this

```powershell
# Update element with index 0
$anotherArray[0] = 200

$anotherArray
200
101
102
103
104
105
106
107
108
109
110
```

Arrays support the **SetValue** method to change a value syntax is as follows

```powershell
# Set element with index 1 to 201
$a.SetValue(201,1) #value, index
```

### Add Elements in an Array

Elements can be added to an array with the **+=** notation like this

```powershell
# Update element with index 0
$anotherArray += 500

$anotherArray

200
201
102
103
104
105
106
107
108
109
110
500
```

**Note:** I wrote a detailed explanation of what happens when using the above notation in my previous article [Add or remove elements from an array](https://pscustomobject.github.io/powershell/Add-Remove-Items-From-Array/). In a nutshell PowerShell is creating a copy of the array and copying all values and adding the one we specified.

### Remove Elements in an Array

Strictly speaking, similarly to what happens with adding elements, it is not possible to delete them from an array. The trick is creating a new array that will contain *only* relevant elements. 

Here's an example where we are removing the last element of the *$anotherArray*:

```powershell
# Remove last element from the $anotherArray
$removeElement = $anotherArray[0..($anotherArray.Length -2)]

200
201
102
103
104
105
106
107
108
109
110

# Create an array containing only some values
$newArray = $anotherArray[0,1,3]

200
201
103
```

### Deleting an array

To remove array's content but not the array itself you can simply assign a *$null* value to it. This will remove all data contained in the array but no the object itself

```powershell
$newArray = $null
```

You can also use the *Remove-Item* cmdlet but assigning a $null value is the preferred approach as it is faster especially when dealing with large arrays.
