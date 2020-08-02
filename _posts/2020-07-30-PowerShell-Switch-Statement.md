---
title: "PowerShell Switch Statement"
excerpt: "In this post we will see uses of Switch statement in PowerShell, advantages it offers over if/else construct and how to use a switch with multiple conditions"
categories:
  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Performance Tips
---

Please raise your (virtual) hand if you ever found yourself in need to have your code take a different path depending on the value of a specific variable or outcome of a command.

While I cannot see raised hands I am sure we all have been there and maybe ended up using a chained list of *if/elseif/else* statements. Today we will look at a better way of doing this via the **switch** statement.

## The if/elseif/else statement(s)

Let's start with a very basic example:

```powershell
# Declare variable holding week day
[string]$weekDay = 'Sunday'

if ($weekDay -eq 'Sunday')
{
    Write-Host -Message 'You can rest sir, it is Sunday after all!'
}
else 
{
    Write-Host -Message 'Wake up you have work to do!'
}
```

This is a very basic example just to get the idea and all in all there is nothing wrong with the above code, it will execute and is easy enough to read.

What happens if extend on it and have a more complex logic?

```powershell
$weekDay = 0

    if ( $day -eq 0 ) { $dayName = 'Sunday'        }
    elseif ( $weekDay -eq 1 ) { $dayName = 'Monday'    }
    elseif ( $weekDay -eq 2 ) { $dayName = 'Tuesday'   }
    elseif ( $weekDay -eq 3 ) { $dayName = 'Wednesday' }
    elseif ( $weekDay -eq 4 ) { $dayName = 'Thursday'  }
    elseif ( $weekDay -eq 5 ) { $dayName = 'Friday'    }
    elseif ( $weekDay -eq 6 ) { $dayName = 'Saturday'  }

    Write-Host "Today is $dayName"

    # Today is Sunday
```

Well nothing will happen the above is perfect valid code and rather common but it presents two issues, *readability* and, to a less extent, *performance*.

An alternate, and if you ask me better, way to solve the above is a **switch** statement which is support but most programming languages (but not Python).

## The switch statement

First of all let's rewrite the above example using a *switch* statement to immediately appreciate the differences:

```powershell
switch ($weekDay)
{
    0 { $dayName = 'Sunday' }
    1 { $dayName = 'Monday' }
    2 { $dayName = 'Tuesday' }
    3 { $dayName = 'Wednesday'}
    4 { $dayName = 'Thursday' }
    5 { $dayName = 'Friday' }
    6 { $dayName = 'Saturday'}
}
```

The above is trivial example but it should be clear how code is more readable and easier to maintain especially if within parentheses we start implementing custom logic and/or additional operations, as I said there is also the added performance benefit as a switch statement is generally faster than an if/else construct as internally PowerShell will build a lookup table in memory to evaluate the various expressions.

In the above example I used an *integer* as the lookup value but of course we can use strings as well:

```powershell
switch ($weekDay)
{
    '0' { $dayName = 'Sunday' }
    '1' { $dayName = 'Monday' }
    # ...
}
```

### The default statement

The examples we just illustrated pose an interesting question, *what if the values of $weekDay variable is not contained in our switch statement*? The **default** clause exists just for and as the name implies it exists as *catch all* and is used like this

```powershell
$weekDay = 100

switch ($weekDay)
{
    '0' { $dayName = 'Sunday' }
    '1' { $dayName = 'Monday' }
    # ...
    default { Write-Host -Message 'Sorry I cannot recognize the value!' }
}
```

### Switch matching Arrays

One little known and appreciated feature of the *switch* statement is its ability to check values in an array

```powershell
[array]$serverRole = @(
    'Web Server',
    'Application Server',
    'Database Server',
    'Gateway Server'
)

switch ($serverRole)
{
    'Web Server'
    {
        'Install the web server role'
    }
    'Application Server' { 'Install the Applicaiton Server Role' }
    'Database Server' { 'Install Datbase Server' }
    'Gateway Server' {'Install Gateway components' }
    default {'Take no action'}
}
```

**Note:** If an element is repeated in the array it will be matched multiple times by the switch statement.
{: .notice--warning}

## PSItem

We can use the $PSItem or $_ built-in variables to reference the current item that has been processed. When we do a simple match, $PSItem will be the value that we are matching.

## Switch Parameters

Up to this point we've illustrated the basic capabilities of a switch statement but there is much more to explore next let's talk about **parameters** allowing to alter the way statements works.

## -CaseSensitive

As most things in PowerShell even the switch statement is *case insensitive* by default and we can use the **-CaseSensitive** parameter, in conjunction with others, to alter this default behavior and only perform exact matches similar to **-cmatch**.

## -RegEx

This is one of my favorite and I use it a lot when parsing log files from my *[New-LogEntry](https://github.com/PsCustomObject/New-LogEntry)* function here is a sample snippet

```powershell
[string]$logMessage = '[Error] - Exception reported'

switch -Regex ( $logMessage )
{
    '^\[Error]'
    {
        # Found error message
    }
    '^\[Warning]'
    {
        # Found warning message
    }
    default
    {
        # Found Info message
    }
}
```

As you can see we can leverage the power of regular expressions to match data via the switch statement.

## -WildCard

Another parameter I tend to abuse is the *-WildCard* which is similar to the *-Like* operator from which it it inherits the matching logic

```powershell
[string]$logMessage = '[Error] - Exception reported'

switch -Regex ( $logMessage )
{
    '*[Error]*'
    {
        # Found error message
    }
    '*[Warning]*'
    {
        # Found warning message
    }
    default
    {
        # Found Info message
    }
}
```

## Switch advanced usage

Up to know we've learned about the basic switch usage but there is even  more let's see some advanced examples.

### Expressions and multiple matches

We can use a switch statement in conjunction with expressions just like we would with an if/else statement

```powershell
 switch ( ([string]::IsNullOrEmpty($myString)) )
 {
    $True
    {...}
    $False
    {...}
 }
```

Multiple matches would work as you probably have already guessed, here I will be taking advantage of PowerShell case insensitivity

```powershell
    switch ( 'PsCustomObject' )
    {
        'pscustomobject' { 'lower case' }
        'PsCustomObject' { 'camel case' }
        'PSCUSTOMOBJECT' { 'upper case' }
    }

    # Output
    lower case
    mixed case
    upper case
```

## Break and Continue

We've already covered *[break and continue](https://pscustomobject.github.io/powershell/post%20series%20-%20statements/PowerShell-Return-Continue-Explained/)* and they work similarly with a switch statement.

Let's examine a reworked version of the previous example:

```powershell
    switch ( 'PsCustomObject' )
    {
        'pscustomobject'
        {
            'lower case'

            continue
        }
        'PsCustomObject'
        {
            'camel case'

            continue
        }
        'PSCUSTOMOBJECT'
        {
            'upper case'

            continue
        }
    }

    # Output
    lower case
```

See the difference? What is happening here is that our switch statement will run the first match, lower case, and *continue* to the next element exactly as it would happen in an *foreach* loop for example but as there are no more elements to process code will simply move over.

What the break statement will do is exiting the switch statement immediately without processing any further element

```powershell
    $logMessages = @(
        'Item is being downloaded'
        'Warning - Error downloading file'
        'Error - Error out of disk space'
        'Notfying admins'
        '...'
    )

    switch -Wildcard ($Messages)
    {
        'Warning*'
        {
            Write-Error -Message $PSItem

            continue
        }
        'Error*'
        {
            Write-Warning -Message $PSItem

            break
        }
        default
        {
            Write-Output $PSItem
        }
    }

    # Output
    'Item is being downloaded'
    'Warning - Error downloading file'
    'Error - Error out of disk space'
```

In the above example I have used a mix of *continue* and *break* statements. As you can see PowerShell is printing the first element as well the *Warning* message as it uses a continue statement but once it hits the error message it will stop processing additional elements and will continue with code execution.

## ScriptBlocks

Up to this point we have observed switch while performing *static* comparisons but we can also use a *scriptblock* expression like in the following example:

```powershell
    $age = 37

    switch ( $age )
    {
        {$PSItem -le 18}
        {
            'child'
        }
        {$PSItem -gt 18}
        {
            'adult'
        }
    }

    # Output
    'adult'
```

## Closing thoughts on the Switch statement

There are still some *edge* uses for the switch statement that I'm not going to cover here as they're rarely used. Maybe that will be material for a future post.

What I find important to highlight that, in addition to being more *elegant* and easier to read, a *switch* statement is usually **faster** than an *if/else* construct due the way PowerShell internally indexes data so try to stick to it whenever possible.

As usual I hope this post helped you and if you feel like I missed something or have any question don't hesitate to leave a comment.
