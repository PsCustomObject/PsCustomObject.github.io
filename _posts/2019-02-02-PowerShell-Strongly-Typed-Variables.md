---
title: "PowerShell Strongly Typed Variables"
excerpt: "PowerShell Strongly Typed Variables, what they are, why you should care and why you should use them in your code to avoid pitfalls and potential issues with data types"
categories:

  - PowerShell
  - Coding Habits
  - PowerShell Best Practice
tags:
  - PowerShell
  - PowerShell Core
  - Variables
  - PowerShell Variables
---

## PowerShell Strongly Typed Variables

PowerShell is dynamically implicit typed, this is a fact. What does that mean?

Unlike other languages, C# or even C/C++, changing a variable's type won't generate an error. Let's look at an example

```powershell
# Declar an int variable
$myVar = 1000

# Get type
$myVar.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType

# Change type dynamically
$myVar = "Now this is a string"

$myVar.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object
```

In the above example we declared *myVar* with a type of **Int** but afterwards assigned a **String** value. PowerShell did not complain and dynamically converted the variable's type.

## Implicitly Typed Variables

In addition to allow a runtime conversion PowerShell will usually try its best to guess the *type* of the object we are trying to instantiate and use.  As an example, writing something like this

```powershell
$myArray = @(1,2,'Test')
```

Will tell PowerShell we're trying to create an Array which can be easily verified with

```powershell
$myArray.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array
```

This is different from what we have in strongly typed languages, like C# for example, where you cannot declare a variable without also specifying the data type it will contain.

Just as an example here's how we would declare a string and integer variable in C#

```c#
string myString = "Some String";
int myInt = 10;

// This will generate a compilation error
anotherString = "Other text";

// This will also generate a compilation error
myInt = "A string" // We cannot implicitly convert an Int to String
```

## Strongly Typed Variables

If you downloaded any of my Gist or functions from [my repository](https://github.com/PsCustomObject?tab=repositories) you probably have noticed that I do use a lot **strongly typed variables**, but before proceeding any further let me clarify *this is not required by PowerShell but it is what I consider a good coding practice*.

So how do we define a *strongly typed* variable? In reality this is not very different from what we have seen in the small C# example, for example to declare a *string* variable we would write code like this:

```powershell
# Create a string object
[string]$myVariable = 'This is a strongly typed string'

# Create an array object
[array]$myArray = @(1,2,'Test')
```

Main difference between the above example and what we have seen earlier is we're *telling* PowerShell the type of data we will be storing in the variable without having it do the guesswork.

As I already said this is not strictly required by PowerShell rules but it is what I consider a good coding habit and I can assure the extra typing will be paying in both **code readability** and **ease of debugging** when doing a code review or trying to figure out what went wrong with a script.

First of all when using strongly typed variables it does not matter how many lines our script is composed of we will always know which *properties* and *methods* are available and can be applied to a specific variable. As an example a *string* object will always have a *length* property or a *ToLower* method, the same holds true for all other values like *int*, *arrays* and so on.

To illustrate the second aspect of strongly typed variables let me quote what I wrote earlier

> ...we declared *myVar* with a type of **Int** but afterwards assigned a **String** value. PowerShell did not complain and dynamically converted the variable's type...

PowerShell normally allows us to change data type stored in a variable at runtime, while this flexibility can be handy sometime it also has potential to introduce issues in our code. When we're using strongly typed variables **PowerShell will not allow us to change data type** without creating a new variable with the correct data type in it.

Let's see this behavior in action using an implicitly typed variable

```powershell
# Implicitly typed variable
$myVariable = 'This is now a string'

# Get Type
$myVariable.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object

# Assign new value
$myVariable = Get-Date

# Get Type
$myVariable.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     DateTime                                 System.ValueType
```

In the above example *$myVariable* is initially created as a *string* but after we assign content of the *Get-Date* cmdlet it is automatically converted to a *DateTime* object. This is behaviour is not an issue per se, it is just the way PowerShell works, but can create unexpected results if it is not properly handled in script code.

Rewriting the above example using strongly typed variables will yield a different result

```powershell
# Implicitly typed variable
[string]$myVariable = 'This is now a string'

# Get Type
$myVariable.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object

# Assign new value
$myVariable = Get-Date

# Get Type
$myVariable.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object
```

As it is possible to see in the above example the *$myVariable* data type did not change and output of the *Get-Date* cmdlet was automatically converted to a string object. The above is possible as the *Get-Date* cmdlet has a *toString* method but if we tried to something *illegal* like assigning a string to an *Int* value we would get an error

```powershell
[int]$myIntVariable = 100

$myIntVariable = 'String Value'

Cannot convert value "String Value" to type "System.Int32". Error: "Input string was not in a correct format."
At line:1 char:1
+ $myIntVariable = 'String Value'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : MetadataError: (:) [], ArgumentTransformationMetadataException
+ FullyQualifiedErrorId : RuntimeException
```

PowerShells is protecting us from carrying on an illegal operation like assigning a string value to an int variable, keep in mind PowerShell is smart enough to handle situations like this 

```powershell
[int]$myIntVariable = 100

$myIntVariable = '5' # This is a string value

Write-Host $myIntVariable

5

$myIntVariable.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType
```

What happened is PowerShell called the *ToInt32* method on the *string* value we specified and converted value to an int.

On a closing note I want to call you attention on a small detail, you need to specify the data type for a variable **only once** when you declare the variable, while there is nothing stopping you from using the notation *[DataType]$variableName* for each occurrence this is not strictly necessary and would only lead to extra typing.
