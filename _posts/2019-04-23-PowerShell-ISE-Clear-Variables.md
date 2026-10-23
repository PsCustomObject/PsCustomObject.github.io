---
title: "PowerShell ISE Clear Variables without restarting"
excerpt: "Clean up selected variables and modules in Windows PowerShell ISE, and understand when a new process is needed for a fresh test."
categories:

  - PowerShell
  - HowTo
tags:
  - PowerShell
  - PowerShell ISE
  - PowerShell Editors
last_modified_at: 2026-10-23
---

## PowerShell ISE and Variables values

This guide applies to **Windows PowerShell ISE**, which hosts Windows PowerShell rather than PowerShell 7. Microsoft maintains ISE for security and high-priority fixes but no longer develops new features. See [Microsoft’s ISE overview](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise).

Repeated runs in the same session can reuse state, particularly when code is dot-sourced or selected lines are executed in the console. Remove the variables owned by the script explicitly instead of deleting every writable variable in the session:

```powershell
# Replace these example names with variables created by your script.
Remove-Variable -Name 'reportRows', 'targetServers' -ErrorAction SilentlyContinue
```

The original `Remove-Variable * -ErrorAction SilentlyContinue` command was too broad and did not clear all variables. System-owned and constant variables cannot be removed; silently suppressing errors also hides unsuccessful removals. See [Remove-Variable](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/remove-variable).

## PowerShell ISE Errors and Modules

You can clear the error collection before another test:

```powershell
$error.Clear()
```

This only clears recorded errors. It does not repair the cause or reset the session.

Remove a specific module when you need to remove its exported commands:

```powershell
# Replace with the module you are developing.
Remove-Module -Name 'MyAutomationModule' -ErrorAction SilentlyContinue
```

[Remove-Module](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/remove-module) affects the current session, but it does not unload a module’s .NET assemblies. Removing all modules can also disrupt unrelated work.

## Use a new process for an isolated test

Clearing variables, errors and modules does not produce a fresh ISE session. Loaded assemblies, other functions, background work and process-level state can remain. For a clean script run, start a separate process with no profile:

```powershell
# Windows PowerShell 5.1; replace the example path.
powershell.exe -NoProfile -File 'C:\Scripts\MyScript.ps1'

# PowerShell 7, if installed and supported by the script.
pwsh -NoProfile -File 'C:\Scripts\MyScript.ps1'
```

A new process isolates in-memory PowerShell state; it does not undo changes to files, services or remote systems. For interactive ISE debugging that requires a clean process, close and reopen ISE.
