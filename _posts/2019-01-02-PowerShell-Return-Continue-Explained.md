---
title: "PowerShell Return and Continue - Explained"
excerpt: "An introduction to PowerShell statements. In this post we will explore Return and Continue"
categories:
  - PowerShell
  - Post Series - Statements
tags:
  - PowerShell
  - PowerShell Basics
  - PowerShell Core
  - Loops
  - PowerShell Return
  - PowerShell Continue
---

## PowerShell return, continue and exit statements

In the previous post [PowerShell break statement - Explained](https://pscustomobject.github.io/powershell/PowerShell-Break-Explained/) we have seen how we can use the *break* statement to interrupt script execution, we also briefly mentioned other statements like *continue*, *return* and *exit* which we are going to cover in this article.

## Continue statement

Similarly to *break* the **continue** statement is used to alter flow of a loop but instead of forcing flow to immediately quit it will cause next element to be processed.

To illustrate this I will use the same example we used in the previous article for reference only here's the code

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

The above code will count from 0 to 100 printing each number in the sequence but we forced it to quit at **10** via the *break* statement.

Here's a slightly different example with the **continue** statement:

```powershell
# Count from 0 to 10 but exit as soon you
# Counter reaches 10
$myNumber = 0

while ($myNumber -lt 10)
{
    # Increment
    $myNumber++

    if ($myNumber -eq 5)
    {
        Write-Host "I will not print this number!"

        continue
    }

    Write-Host "Value is $myNumber"
}

Write-Host 'This is outside the loop'
```

The above code will produce the following output

>Value is 1
>Value is 2
>Value is 3
>Value is 4
>I will not print this number!
>Value is 6
>Value is 7
>Value is 8
>Value is 9
>Value is 10

As you can see we altered loop flow, number *5* is not printed, without completely interrupting its execution, we simply skipped an element which is very useful in situations like exception handling.

## Return Statement

The **return** statement allows us to *exit* the current scope be it a *function*, *script* or *script block*.

Oversimplifying the concept **return** has a similar effect as the *break* statement but it allows us to return an object as part of the process.

Let's see an example of this with the same code we just used

```powershell
# Count from 0 to 10 but exit as soon you
# Counter reaches 10
$myNumber = 0

while ($myNumber -lt 10)
{
    # Increment
    $myNumber++

    if ($myNumber -eq 5)
    {
        return $myNumber
    }

    Write-Host "Value is $myNumber"
}
```

The above code will produce the following output
> Value is 1
> Value is 2
> Value is 3
> Value is 4
> 5

Number 5 is **returned** as output and not simply printed on screen like other numbers yet the loop will not continue its execution.

This is the basis of working with functions which is something I will better describe in a new post.

## Closing Notes

This article concludes our introduction to PowerShell statements and how they can be used to alter flow in our scripts.

In a subsequent post series I will describe **Functions** where the *return* statement will be the foundation of our coding.
