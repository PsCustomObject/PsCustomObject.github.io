---
title: "PowerShell Exception Handling"
excerpt: "An introduction to PowerShell error handling. In this post we will explore what error handling is and how to implement it in PowerShell scripts"
categories:

  - PowerShell
tags:
  - PowerShell
  - PowerShell Basics
  - PowerShell Core
  - Error Handling
  - Exception Handling
---

## What is PowerShell Exception handling

I think the best definition of *error* or *exception* handling I can quote is the one coming from Wikipedia which reads:

> **Exception handling**  is the process of responding to the occurrence, during computation, of *exceptions* – anomalous or exceptional conditions requiring special processing – often changing the normal flow of program execution.

In other words **exception handling** is the art of pre-empting any unforeseen or unexpected situation which could arise during program or script execution. I chose word *art* on purpose as it takes time and practice to master it.

In the post I will got through **error types**, **error action preference**, the **$Error** variable, **try/catch/finally** blocks and will finally cover the **$LastExitCode** variable to trap errors generated *outside* PowerShell processes.

It seems a lot of material but **Exception Handling** is an essential part of any well written script, without it you risk to implement to quickly lose control of what your script did or *did not*.

I have broken down the post into sections to make navigation easier given the amount of content.

## Exception Handling - An Example

Before diving error types I want to introduce a small example which hopefully will help illustrate pitfalls of a script not implementing exception handling.

Imagine we've been tasked with the creation of a script that will check a specific application folder and delete all temporary files an application is creating in it, script should run daily on a schedule.

```powershell
# Define Application files path
$tempFilesPath = 'C:\MyApp\Temp1\'

# Get all files
$tempFiles = Get-ChildItem -Path $tempFilesPath

# Remove files
foreach ($tempFile in $tempFiles)
{
    Remove-Item -Path $tempFile
}

Write-Host 'Script execution complete!'
```

Once tested our script we schedule it and take on another task, unfortunately the user account under which the scheduled task runs has no permissions to access the folder so script won't be able to complete successfully. As we implemented no proper exception handling we would find out about this only when files would start accumulating filing up the disk on the server.

This is a very simple example to show why you **should always** implement exception handling in your scripts.

## Error Types

In PowerShell we have two error type *Terminating Errors* and *Non-Terminating Errors* which I describe in more details below.

### Terminating Errors

A **terminating error** is a serious error that causes the command or script execution to halt completely. 

Among terminating errors:

- A non existing cmdlet
- A syntax error that would prevent the cmdlet to correctly continue execution (*Get-ChildItem -Path $null*)
- Other fatal errors like for example a missing *required* parameter

### Non-Terminating Errors

A **non-terminating** error, as the name implies, is a non *serious* error that will allow execution to continue.

Among non-terminating errors:

- *File not found* type of errors
- Permission problems

If you're wondering error reporting is defined by the cmdlet developer and [MSDN](https://docs.microsoft.com/it-it/powershell/developer/cmdlet/error-reporting-concepts) has more information about this.

**Note:** sure to remember difference between **terminating** and **non-terminating** errors as it is essential to proper exception handling in your code.

## Error Action Preference

Now that we know more about *error types* in PowerShell it is time to talk about **Error Action Preference**.

We know that a *terminating* error will halt script execution but what of *non-terminating* errors? Short answer is we can tell PowerShell what to do when such errors are encountered through the **$ErrorActionPreference** variable.

The **$ErrorActionPreference** supports the following values:

- **SilentlyContinue** - Execution continues and any error message is suppressed
- **Stop** - Pretty self explanatory, it forces execution to top conceptually identical to a *terminating* error
- **Continue** - This is the default setting. Errors will be displayed and execution continue
- **Inquire** - As the name implies it will ask for user input to check if execution should proceed
- **Ignore** - Error is completely ignored and **not** logged to the error stream

To check **$ErrorActionPreference** value simply type the following:

```powershell
# Print value
$ErrorActionPreference

# Output
Continue
```

You can of course change value of the *$ErrorActionPreference* variable both at the command level 

```powershell
# Set Error action to Inquire
Get-ChildItem Get-ChildItem 'C:\Temp' -ErrorAction 'Inquire'

# Set Error action to Ignore
Get-ChildItem Get-ChildItem 'C:\Temp' -ErrorAction 'Ignore'
```

Or at the script level

```powershell
# Set Error action for the whole script
$ErrorActionPreference = 'Stop'
```

The same holds true for an interactive PowerShell session of course.

## The $Error Variable

When an error occurs during execution, be it terminating or not, information is logged to a variable called **$Error** which is nothing else than an [arraylist object](https://pscustomobject.github.io/powershell/Add-Remove-Items-From-Array/)

```powershell
$Error.GetType()

# Output
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     ArrayList                                System.Object
```

When you start a new PowerShell session the variable will be of course empty but yet it will be initialized.

Let's see how this works trying to run a cmdlet that does not exist:

```powershell
# This does not exist
Get-MyCmdLet

# Get content of the $Error variable
$Error
Get-MyCmdLet : The term 'Get-MyCmdLet' is not recognized as the name of a cmdlet, function, script file, or operable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
At line:1 char:1
+ Get-MyCmdLet
+ ~~~~~~~~~~~~
+ CategoryInfo          : ObjectNotFound: (Get-MyCmdLet:String) [], CommandNotFoundException
+ FullyQualifiedErrorId : CommandNotFoundException
```

In the above snippet I've suppressed output of the inexistent cmdlet as it is the same contained in the **$Error** variable which logged the exception. 

What if you have multiple errors in your session? The $Error variable will contain all of them with the most recent error on top, you can easily verify this via the *count* property

```powershell
$Error.Count
2
```

As the **$Error** variable is an *Array* you can always access last element specifying the index number like this

```powershell
# Print latest error
$Error[0]
```

The **ErrorRecord**, the object type for the $Error variable, has many useful properties that we can access and use in our scripts, to list all of them you can use the following

```powershell
$Error[0] | Get-Member

   TypeName: System.Management.Automation.ErrorRecord

Name                  MemberType     Definition
----                  ----------     ----------
Equals                Method         bool Equals(System.Object obj)
GetHashCode           Method         int GetHashCode()
GetObjectData         Method         void GetObjectData(System.Runtime.Seria...
GetType               Method         type GetType()
ToString              Method         string ToString()
CategoryInfo          Property       System.Management.Automation.ErrorCateg...
ErrorDetails          Property       System.Management.Automation.ErrorDetai...
Exception             Property       System.Exception Exception {get;}
FullyQualifiedErrorId Property       string FullyQualifiedErrorId {get;}
InvocationInfo        Property       System.Management.Automation.Invocation...
PipelineIterationInfo Property       System.Collections.ObjectModel.ReadOnly...
ScriptStackTrace      Property       string ScriptStackTrace {get;}
TargetObject          Property       System.Object TargetObject {get;}
PSMessageDetails      ScriptProperty System.Object PSMessageDetails {get=& {...
```

Among the various properties the two you I usually tend to use more are:

- **$Error[0].Exception** - Contains the original exception object as it was thrown to PowerShell
- **$Error[0].InvocationInfo** - Contains details about the context which the command was executed, if available

The former is handy to log exception thrown by scripts in my [script log file](https://github.com/PsCustomObject/New-LogEntry) the latter is useful when troubleshooting script execution to know what exactly went wrong and where.

But there is more, the **Exception** object *hides* some additional useful properties and methods that are not immediately visible.

### The Hidden Secrets of $Error.Exception

Piping the **$Error[0].Exception** to the *Get-Member* cmdlet we can see there are many additional properties to the exception for example here's what I can see from the exception thrown by the inexistent cmdlet

```powershell
$Error[0].exception | Get-Member

   TypeName: System.Management.Automation.CommandNotFoundException

Name                        MemberType Definition
----                        ---------- ----------
Equals                      Method     bool Equals(System.Object obj)
GetBaseException            Method     System.Exception GetBaseException()
GetHashCode                 Method     int GetHashCode()
GetObjectData               Method     void GetObjectData(System.Runtime.Ser...
GetType                     Method     type GetType()
ToString                    Method     string ToString()
CommandName                 Property   string CommandName {get;set;}
Data                        Property   System.Collections.IDictionary Data {...
ErrorRecord                 Property   System.Management.Automation.ErrorRec...
HelpLink                    Property   string HelpLink {get;set;}
HResult                     Property   int HResult {get;set;}
InnerException              Property   System.Exception InnerException {get;}
Message                     Property   string Message {get;}
Source                      Property   string Source {get;set;}
StackTrace                  Property   string StackTrace {get;}
TargetSite                  Property   System.Reflection.MethodBase TargetSi...
WasThrownFromThrowStatement Property   bool WasThrownFromThrowStatement {get...
```

As you can see in the above example we can access hidden information like *StackTrace*, *InnerException* or the *Source*.

It has to be noted that we don't always need access to this level of information but it can be very useful in a troubleshooting scenario especially when dealing with 3rd party cmdlets or modules.

## The try/catch/finally statements

With our newly acquired knowledge we're now ready to dive deep into **try/catch** blocks that are the foundation of exception handling in PowerShell.

The **try/catch/finally** statements allows us to alter script execution flow and *respond* to any error the could be thrown by our code. Default behavior for *try/catch/finally* statement is to *catch* all terminating errors while any non-terminating error will be *ignored* by it.

Syntax is rather straightforward and can be summarized as:

- **Try** - Is the command we want PowerShell to execute
- **Catch** - Is the error condition we want to test for and take action on. We can specify multiple *catch* for different exceptions
- **Finally** any code in the *finally* block will always be executed no matter what. This block is optional and you can omit if you don't plan to use it

Let's see an example with our inexistent cmdlet once again

```powershell
Try
{
    # Will generate a temrinating error
    Get-MyCmdlet -Path 'C:\temp'
}
Catch
{
    Write-Warning 'I could not run command!'
}
Finally
{
    # Will always be executed
    Write-Host 'Script Execution complete!'
}
```

The above is a very simple example of how you would use the *try/catch* statement but, as I wrote earlier, this will work only for terminating errors.

Take the following example:

```powershell
try
{
   $networkCard = Get-WmiObject Win32_NetworkAdapterConfiguration -ComputerName $Computername -Credential $Credential -Filter "IPEnabled = $True"
}

# Catch exception
catch [GetWMICOMException]
{
    Write-Warning 'Error running command!'
}
```

Despite syntax being correct if there is an issue with the code in the *try* block execution will continue and code in the *catch* block will never be executed. Reason for this is that *Get-WmiObject* throws non-terminating errors.

Solution to the above issue lies in the **$ErrorActionPreference** variable which, as I wrote earlier, can be set to any of the supported values according to our need.

Here is the rewritten example:

```powershell
try
{
    # Update variable
    $ErrorActionPreference = 'Stop'

    $networkCard = Get-WmiObject Win32_NetworkAdapterConfiguration -ComputerName $Computername -Credential $Credential -Filter "IPEnabled = $True"
}

# Catch exception
catch [GetWMICOMException]
{
    Write-Warning 'Error running command!'
}

Finally
{
    # Set value back to default value
    $ErrorActionPreference = 'Continue'
}
```

The above code will update the $ErrorActionPreference variable so any error will be treated as terminating allowing us to *catch* it, in the *finally* block we set value back to its default value.

The example also illustrate another important concept which is catching a specific exception which can be useful when we want to take different actions depending on the exception thrown.

To find the specific exception we use again the *Get-Member* cmdlet applied to the exception object

```powershell
$Error[0].Exception | Get-Member

   TypeName: System.Management.Automation.CommandNotFoundException
```

The above is from the previous example with the inexistent cmdlet but it is exactly how it works with any cmdlet, the *TypeName* will contain the exception type/name.

**Note:** If you are interested in knowing more [MSDN](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.errorrecord?redirectedfrom=MSDN&view=powershellsdk-1.1.0) has great documentation on the Error Record class and how to use it.

## Exception Handling - A Better Example

At the beginning of the post I wrote an example to illustrate how proper exception handling is very important in any script or automation project.

Here's the code rewritten using techniques and concept we covered in the post

```powershell
# Define Application files path
$tempFilesPath = 'C:\MyApp\Temp1\'
$isException = $false

try
{
    # Get all files
    $tempFiles = Get-ChildItem -Path $tempFilesPath
}
catch [ItemNotFoundException]
{
    <#
        Any corrective action code here
    #>
    
    $isException = $true
}
finally
{
    if ($isException)
    {
        <#
            Code to notify admins here
        #>
    }
    else
    {
        # Log success
    }
}

# Remove files
foreach ($tempFile in $tempFiles)
{
    try
    {
        Remove-Item -Path $tempFile
    }
    catch
    {
        <#
            Any corrective action code here
        #>
    
        $isException = $true
    }
    finally
    {
        if ($isException)
        {
            <#
                Code to notify admins here
            #>
        }
        else
        {
            # Log success
        }
    }
}

Write-Host 'Script execution complete!'
```

## Closing Thoughts

In the post we've illustrated importance of *exception handling* and how to implement it in PowerShell. It does not matter how simple or *short* the script you are writing is you should always try to implement proper exception handling to save yourself some work when things don't go the way you planned them.
