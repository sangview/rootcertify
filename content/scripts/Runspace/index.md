---
title: "Runspace Introduction"
description: ""
summary: "You can use blog posts for announcing product updates and features."
date: 2023-09-07T16:21:44+02:00
lastmod: 2023-09-07T16:21:44+02:00
draft: false
weight: 10
categories: []
tags: ["Powershell", "Script", "Windows"]
contributors: ["Adama SANGARE"]
pinned: false
homepage: false
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

Les runspaces sont des espaces d'exécution PowerShell qui peuvent être utilisés pour exécuter des commandes indépendamment du runspace principal."
excerpt: "Les runspaces sont des espaces d'exécution PowerShell qui peuvent être utilisés pour exécuter des commandes indépendamment du runspace principal. Ils sont utiles pour une variété de tâches, notamment la parallélisation des tâches, l'isolation des ressources et l'exécution de commandes sur des systèmes distants.

# Utilisation des runspaces dans PowerShell

Les runspaces sont des espaces d'exécution PowerShell qui peuvent être utilisés pour exécuter des commandes indépendamment du runspace principal. Ils sont utiles pour une variété de tâches, notamment la parallélisation des tâches, l'isolation des ressources et l'exécution de commandes sur des systèmes distants.

## Création d'un runspace

La façon la plus simple de créer un runspace est d'utiliser la commande `New-Runspace`. Cette commande prend plusieurs paramètres, notamment le nom du runspace, le type de runspace et les modules à charger.

```powershell
# Crée un runspace nommé "MyRunspace"
New-Runspace -Name MyRunspace

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available

PS C:\Windows\system32>   $runspace3 = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace()

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available
  3 Runspace3       localhost       Local         BeforeOpen    None

PS C:\Windows\system32>   $runspace3.Open()

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available
  3 Runspace3       localhost       Local         Opened        Available

PS C:\Windows\system32>   $runspace3.Dispose()

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available


# Crée un runspace nommé "MyRunspace" avec les modules "Microsoft.PowerShell.Management" et "Microsoft.PowerShell.Security" chargés
New-Runspace -Name MyRunspace -Module Microsoft.PowerShell.Management,Microsoft.PowerShell.Security
```
## Initialiser l'environnement d'un runspace

```powershell
PS C:\Windows\system32> $init = [System.Management.Automation.Runspaces.InitialSessionState]::Create()

PS C:\Windows\system32> $init | Get-Member

   TypeName: System.Management.Automation.Runspaces.InitialSessionState

Name                          MemberType Definition
----                          ---------- ----------
Clone                         Method     initialsessionstate Clone()
Equals                        Method     bool Equals(System.Object obj)
ImportPSModule                Method     void ImportPSModule(string[] name), void ImportPSMod...
Commands                      Property   System.Management.Automation.Runspaces.InitialSessio...EnvironmentVariables          Property   System.Management.Automation.Runspaces.InitialSessio...
ExecutionPolicy               Property   Microsoft.PowerShell.ExecutionPolicy ExecutionPolicy...
Formats                       Property   System.Management.Automation.Runspaces.InitialSessio...
LanguageMode                  Property   System.Management.Automation.PSLanguageMode Language...
Modules                       Property   System.Collections.ObjectModel.ReadOnlyCollection[Mi...
Variables                     Property   System.Management.Automation.Runspaces.InitialSessio...
....

PS C:\Windows\system32> $variable = [System.Management.Automation.Runspaces.SessionStateVariableEntry]::new("Test3", 002, '')

PS C:\Windows\system32> $init.LanguageMode = 'FullLanguage'

PS C:\Windows\system32> $init.Variables.Add($variable)

PS C:\Windows\system32> $Runspace3 = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($init)

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available
  4 Runspace4       localhost       Local         BeforeOpen    None

PS C:\Windows\system32>
PS C:\Windows\system32> $Runspace3.Open()

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available
  4 Runspace4       localhost       Local         Opened        Available

PS C:\Windows\system32> $PowerShell3 = [PowerShell]::Create()
PS C:\Windows\system32> $PowerShell3.Runspace = $Runspace3
PS C:\Windows\system32> $PowerShell3.AddScript("`$Test3");

Commands            : System.Management.Automation.PSCommand
Streams             : System.Management.Automation.PSDataStreams
InstanceId          : 55da9744-9280-4cee-882b-60b9722de4ae
InvocationStateInfo : System.Management.Automation.PSInvocationStateInfo
IsNested            : False
HadErrors           : False
Runspace            : System.Management.Automation.Runspaces.LocalRunspace
RunspacePool        :
IsRunspaceOwner     : False
HistoryString       :

PS C:\Windows\system32> $PowerShell3.Invoke();
2
```
## Se connecter au runspace d'une autre session Powershell

```powershell
PS C:\Windows\system32> $pid
10020

PS C:\Windows\system32> $CI = [System.Management.Automation.Runspaces.NamedPipeConnectionInfo]::new(10020)

PS C:\Windows\system32> $Runspace = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($CI)

PS C:\Windows\system32> $Runspace.Open()

PS C:\Windows\system32> $PowerShell = [PowerShell]::Create()

PS C:\Windows\system32> $PowerShell.Runspace = $Runspace

PS C:\Windows\system32> $PowerShell.AddScript("`$PID");


Commands            : System.Management.Automation.PSCommand
Streams             : System.Management.Automation.PSDataStreams
InstanceId          : 5b2320e2-3fad-4459-8752-d4b8f9031ed6
InvocationStateInfo : System.Management.Automation.PSInvocationStateInfo
IsNested            : False
HadErrors           : False
Runspace            : System.Management.Automation.RemoteRunspace
RunspacePool        :
IsRunspaceOwner     : False
HistoryString       :

PS C:\Windows\system32> $PowerShell.Invoke();
10020

PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
 13 Runspace13      localhost       Local         Opened        Available
 14 RemoteHost      localhost       Local         Opened        Available
```

## Conclusion

Les runspaces sont un outil puissant qui peut être utilisé pour une variété de tâches. Ils sont utiles pour la parallélisation des tâches, l'isolation des ressources et l'exécution de commandes sur des systèmes distants.
