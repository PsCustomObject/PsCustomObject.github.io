---
title: "PowerShell load .Net Assembly"
excerpt: "How to load external assemblies in PowerShell to extend functionality and leverage functionalities not available in PowerShell."
toc: true
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell Assemblies
  - PowerShell Tips
  - PowerShell Core
---

One common technique is loading .Net *assemblies* in PowerShell script or module to leverage functionalities otherwise not available natively in PowerShell.

A good example of this is in my [**PowerScp** module](https://github.com/PsCustomObject/PowerScp) which is distributed together with the **WinSCPnet.dll** assembly which makes required classes and methods available from within the PowerShell session.

## What is a .Net Assembly and why use them

Before proceeding let's formally define of what an *assembly* is and why we want to use them in our scripts, straight from MSDN an assembly is

> ...(a) collection of types and resources that are built to work together and form a logical unit of functionality. Assemblies take the form of executable (.exe) or dynamic link library (.dll) files, and are the building blocks of .NET applications. They provide the common language runtime with the information it needs to be aware of type implementations...

If you want to read more about this the full article quoted above is available [here](https://docs.microsoft.com/en-us/dotnet/standard/assembly/).

There are multiple methods we can use to add assemblies to PowerShell which we're going to explore in the post.

## Method 1 - Add assembly with Add-Type cmdlet

Building on the *WinSCPnet.dll* example let's how we can import in our PowerShell code make use of the new types and methods made available by it.

Aim of my **PowerScp** module is creating a wrapper around to the fantastic *WinSCP* GUI package to be used within PowerShell, to make this possible I leveraged the *WinSCP* assembly made available via compiled DLL file so to make available functions and objects that are not native to the .Net Framework and PowerShell.

Exploring this by example in the below snippet, taken from the *New-ScpSession* cmdlet, I'm creating a new object of *WinSCP.Session* type but without importing the required assemblies PowerShell will throw the following exception:

```powershell
# This will fail
[WinSCP.Session]$scpSessionObject =  New-Object -TypeName  'WinSCP.Session'

New-Object : Cannot find type [WinSCP.Session]: verify that the assembly containing this type is loaded.
At line:1 char:1
+ New-Object -TypeName  'WinSCP.Session'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidType: (:) [New-Object], PSArgumentException
    + FullyQualifiedErrorId : TypeNotFound,Microsoft.PowerShell.Commands.NewObjectCommand
```

Error message is rather clear but the important bit is

> Cannot find type [WinSCP.Session]: verify that the assembly containing this type is loaded.

As expected PowerShell is complaining as it cannot find a definition for the *WinsSCP.Session* object this is where adding assemblies to out scripts come into play.

```powershell
# Define Assembly path
[string]$assemblyPath = 'C:\Assemblies\WinSCPnet.dll'

# Add assembly DLL
Add-Type -Path $assemblyPath
```

The above will return no output and import into the PowerShell session all classes and methods associated with the DLL.

```powershell
# Define Assembly path
[string]$assemblyPath = 'C:\Assemblies\WinSCPnet.dll'

# Add assembly DLL
Add-Type -Path $assemblyPath

# Create new object
[WinSCP.Session]$scpSessionObject =  New-Object -TypeName  'WinSCP.Session'

# Get object type and members
$scpSessionObject | Get-Member

TypeName: WinSCP.Session # Note new not native type

Name                          MemberType Definition
----                          ---------- ----------
Failed                        Event      WinSCP.FailedEventHandler Failed(System.Object, WinSCP.FailedEventArgs)
FileTransferProgress          Event      WinSCP.FileTransferProgressEventHandler FileTransferProgress(System.Object,...
FileTransferred               Event      WinSCP.FileTransferredEventHandler FileTransferred(System.Object, WinSCP.Tr...
OutputDataReceived            Event      WinSCP.OutputDataReceivedEventHandler OutputDataReceived(System.Object, Win...
QueryReceived                 Event      WinSCP.QueryReceivedEventHandler QueryReceived(System.Object, WinSCP.QueryR...
Abort                         Method     void Abort()
AddRawConfiguration           Method     void AddRawConfiguration(string setting, string value)
CalculateFileChecksum         Method     byte[] CalculateFileChecksum(string algorithm, string path)
Close                         Method     void Close()
TranslateRemotePathToLocal    Method     string TranslateRemotePathToLocal(string remotePath, string remoteRoot, str...
AdditionalExecutableArguments Property   string AdditionalExecutableArguments {get;set;}
DebugLogLevel                 Property   int DebugLogLevel {get;set;}
DebugLogPath                  Property   string DebugLogPath {get;set;}
DefaultConfiguration          Property   bool DefaultConfiguration {get;set;}
DisableVersionCheck           Property   bool DisableVersionCheck {get;set;}
ExecutablePath                Property   string ExecutablePath {get;set;}
ExecutableProcessPassword     Property   securestring ExecutableProcessPassword {get;set;}
ExecutableProcessUserName     Property   string ExecutableProcessUserName {get;set;}
```

**Note:** Above output is snipped for brevity as the assembly supports a lot of methods and properties

## Method 2 - Add assembly with Reflection Assembly

This is a legacy method that was replaced by the *Add-Type* cmdlet in Version 2.0 of PowerShell but still works in current versions syntax is slightly more *complex* but will get the job done

```powershell
# Define Assembly path
[string]$assemblyPath = 'C:\Assemblies\WinSCPnet.dll'

# Load assembly and suppress output
[void]([System.Reflection.Assembly]::LoadFrom($assemblyPath))
```

In the previous example casting void will suppress output which is, by default, printed by the command which would look similar to the this

```powershell
[System.Reflection.Assembly]::LoadFrom($assemblyPath)

GAC    Version        Location
---    -------        --------
False  v4.0.30319     C:\Assemblies\WinSCPnet.dll
```

## Method 3 - Add assembly via a string object

The methods I have illustrated to this point have similar functionality and will work equally well but both have a drawback *they will put a lock* on the DLL file they're using, this means if you need to cleanup the environment, rename, delete or move the file you will not be able to do so as PowerShell will be blocking the file itself. Generally this is not an issue but when you're developing a solution and need to cleanup the environment rather frequently I can assure it is.

Solution to the above problem is using a **string** in place of the binary DLL file itself which will give us two distinct advantages **fist** no lock will be placed on the assembly file, **second** there is no need for the DLL file to be present on the system where the script or module is running making distribution and portability even easier. Let's explore how this works.

### Step 1 - Get Base64 encoded string

The first step in *converting* the binary DLL to string is getting Base64 encoded string of it which is achieved with the following command:

```powershell
# Define Assembly path
[string]$assemblyPath = 'C:\Assemblies\WinSCPnet.dll'

# Read all bytes from DLL
[byte]$assemblyBytes = [System.IO.File]::ReadAllBytes($assemblyPath)
```

This will read and convert all bytes from the dll specified in the *$assemblyPath* variable if we pipe variable to the screen output will be similar to this

```powershell
$assemblyBytes
77
90
144
0
3
0
0
<snip>
```

Once all the DLL bytes are available we can convert that to a Base64 string using the [Convert.ToBase64String Method](https://docs.microsoft.com/en-us/dotnet/api/system.convert.tobase64string?view=netframework-4.8)

```powershell
[string]$assekblyString = [System.Convert]::ToBase64String($assemblyBytes)
```

Again piping the variable to the console will produce an output similar to the following

**Spoiler Alert:** Resulting string will be really long.
{: .notice--warning}

```powershell
$assekblyString
TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4gaW4gRE9TIG1vZGUuDQ0KJAAAAAAAAABQRQAATAEDAPAmF50AAAAAAAAAAOAAIiALATAAAPoBAAAGAAAAAAAAVhgCAAAgAAAAIAIAAAAAEAAgAAAAAgAABAAAAAAAAAAEAAAAAAAAAABgAgAAAgAAsYsCAAMAQIUAABAAABAAAAAAEAAAEAAAAAAAABAAAAAAAAAAAAAAAAIYAgBPAAAAACACANADAAAAAAAAAAAAAAAAAAAAAAAAAEACAAwAAAAwFwIAVAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIAAACAAAAAAAAAAAAAAACCAAAEgAAAAAAAAAAAAAAC50ZXh0AAAAXPgBAAAgAAAA
<snip>
```

### Step 2 - Embed assembly string in script

Once we have converted the string to distribute required assemblies all we need to do is making the string available in the code but as that will be Base64 encoded we first need to convert it back with the [Convert.FromBase64String Method](https://docs.microsoft.com/en-us/dotnet/api/system.convert.frombase64string?view=netframework-4.8)

```powershell
# Define asembly string
[string]$assekblyString = 'TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAA...'

# Convert string to bytes
[byte]$assemblyBytes = [System.Convert]::FromBase64String($assekblyString)

# Import assembly
[System.Reflection.Assembly]::Load($assemblyBytes)
```

Note how assembly can only be imported via the **System Reflection Assembly** method and not the *Add-Type* one but apart from this functionality in the script will not be impacted and dependency on the binary DLL being available on the system will be removed.

**Pro Tip:** Keep in mind this method while having advantages comes at the cost of making the script file *larger* due the overhead of having the full DLL string embedded in the code and will suffer from a slight performance hit due to all the conversion between types. I don't have enough data to confirm the penalty hit but will carry on some tests.
{: .notice--danger}
