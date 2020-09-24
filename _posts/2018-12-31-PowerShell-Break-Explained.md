---
title: "PowerShell break statement"
excerpt: "An introduction to break statement in PowerShell and its usage"
categories:
  - PowerShell
tags:
  - PowerShell
  - PowerShell Basics
  - PowerShell Core
  - Loops
  - Break
  - Switch
---

## PowerShell Break Statement

When dealing with any kind of loop in PowerShell chances are you came across the **break** statement. Sometimes there is confusion on when **break** should be used especially among people starting to learn PowerShell.

In this series of posts I will try to clarify and illustrate proper usage and pitfalls of the various statements. As the title implies this post is dedicated to the **break** statement.

## PowerShell break in loops

I will start with the easier to describe the **break** statement, its function is simply to **exit a program loop or switch statement immediately**.

Let' see an example with a while loop:

```powershell
# Count from 0 to 100 but exit as soon you
# Counter reaches 10
$myNumber = 0

while ($myNumber -lt 100)
{
    Write-Host "This is inside the loop value is $myNumber"

    # Increment
    $myNumber++

    if ($myNumber -eq 10)
    {
        Write-Host "Value is $myNumber - Exiting now!"

        break
    }

    Write-Host "Value is $myNumber"
}

Write-Host 'This is outside the loop'
```

The above code will start incrementing the *$myNumber* variable when it has  value of *10* the while loop will be stopped and the *This is outside the loop* line will be printed.

The same can also be applied to a *foreach* loop like this:

```powershell
# Users list from AD
$usersList = @(
'user1',
'user2',
'user3'
)

foreach ($user in $userList)
{
    Write-Host "Processing user $user"

    if ($user -ne 'user2')
    {
        Write-Host 'Updating user properties!'
    }
    else
    {
        Write-Host 'Exiting loop now!'

    break
    }
}

Write-Host 'This is outside the loop'
```

In the above example we are cycling through elements of an array and taking some actions. When username is equal to *user2* PowerShell will hit the *break* statement and simply exit the foreach loop executing anything outside it.

## PowerShell break in switch statements

The break statement can also be used in a **switch** statement to stop PowerShell from processing further conditions.

Here's an example:

```powershell
$userDepartment = 'IT'

switch ($userDepartment)
{
    'Sales'
    {
        Write-Host "Department is $userDepartment"

        # Perform some action

        break
    }
    'IT'
    {
        Write-Host "Department is $userDepartment"

        # Perform some action

        break # Processing will stop here
    }
    'HR'
    {
        Write-Host "Department is $userDepartment"

        # Perform some action

        break
    }
    default
    {
        Write-Host 'No Match found!'
    }
}
```

The above code will reach the *IT* clause take any action defined and will continue with anything outside the switch block.

Omitting the *break* statement in the above example would simply cause PowerShell to go through remaining conditions, skipping them as there is no match, before continue execution of any code outside the switch block.

There are situations where this behavior not only is not desirable but could potentially create issues.

Let's see an example:

```powershell
$userCompany = 'Sample Company LLC'

switch -wildcard ($userCompany) {
    'Sample Company*' 
    {
        Write-Host 'Match found!'

        # Perform some action

        break # Processing will stop here
    }
    'Sample*'
    {
        Write-Host 'Match found!'

        # Perform some action

        break
    }
    'Another Company*'
    {
        Write-Host 'Match found!'

        # Perform some action

        break
    }
}

# Will match first occurrence but no the second one
```

The above example is using a partial match for *$userDepartment* name if the *break* statement was omitted script would take whatever action was defined twice as there is a positive match both on the first and second clause of the switch.

**Note:** In this specific example order of the clauses is important while generally this would not be a concern.

## Related Statements

**Break** is not the only supported statement in PowerShell as **return**, **continue** and **exit** can be used as well but I will cover these on separate articles.
