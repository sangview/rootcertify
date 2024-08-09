---
title: "Runspace Introduction"
description: ""
summary: "Un Runspace est un espace d'exécution qui permet d'exécuter du code PowerShell de manière isolée et parallèle."
date: 2023-09-07T16:21:44+02:00
lastmod: 2023-09-07T16:21:44+02:00
draft: false
weight: 10
categories: ["Scripts"]
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

# Utilisation des runspaces dans PowerShell

Les runspaces sont des espaces d'exécution PowerShell qui peuvent être utilisés pour exécuter des commandes indépendamment du runspace principal. Ils sont utiles pour une variété de tâches, notamment la parallélisation des tâches, l'isolation des ressources et l'exécution de commandes sur des systèmes distants.

## Matrice  des États de Runspace

Voici une matrice où les lignes et les colonnes représentent les différents états d'un Runspace en PowerShell. Les valeurs à l'intersection de chaque ligne et colonne indiquent la commande ou l'action qui permet de passer d'un état (ligne) à un autre état (colonne).

## Matrice Binaire des États de Runspace (Optimisée)

| **De\Vers**      | **To State & Command**                                    |
|------------------|-----------------------------------------------------------|
| **BeforeOpen**   | `Opening` via `Open()` / `BeginOpen()`                    |
| **Opening**      | `Opened` via Transition Automatique                       |
| **Opened**       | `Closing` via `Close()` / `BeginClose()`                  |
| **Opened**       | `Disconnected` via `Disconnect()`                         |
| **Opened**       | `Broken` via Erreur Fatale                                |
| **Opened**       | `Disposed` via `Dispose()`                                |
| **Closing**      | `Closed` via Transition Automatique                       |
| **Closing**      | `Disposed` via `Dispose()`                                |
| **Closed**       | `BeforeOpen` via `Open()` / `BeginOpen()`                 |
| **Closed**       | `Disposed` via `Dispose()`                                |
| **Disconnected** | `Opened` via `Connect()`                                  |
| **Disconnected** | `Disposed` via `Dispose()`                                |
| **Broken**       | `Disposed` via `Dispose()`                                |
| **Disposed**     | (État Terminal)                                           |


### Explications des Commandes

- **`Open()` / `BeginOpen()`** : Ouvre le Runspace, passant de l'état `BeforeOpen` à `Opening`, puis à `Opened`.
- **Automatique** : Transition automatique après l'appel d'une commande associée, comme le passage de `Opening` à `Opened` ou de `Closing` à `Closed`.
- **`Close()` / `BeginClose()`** : Ferme le Runspace, passant de `Opened` à `Closing`, puis à `Closed`.
- **`Disconnect()`** : Déconnecte un Runspace ouvert, le mettant dans l'état `Disconnected`.
- **`Connect()`** : Reconnecte un Runspace déconnecté, le ramenant à l'état `Opened`.
- **Erreur fatale** : Une erreur grave peut faire passer un Runspace de `Opened` à `Broken`.
- **`Dispose()`** : Libère les ressources du Runspace, passant à l'état `Disposed`. C'est la dernière étape pour n'importe quel état terminé (`Closed`, `Disconnected`, `Broken`).

### Utilité de la Matrice

Cette matrice permet de comprendre les transitions possibles entre les différents états d'un Runspace dans PowerShell et les commandes spécifiques nécessaires pour effectuer ces transitions. Elle permet de planifier et de gérer correctement le cycle de vie des Runspaces dans des scripts complexes.

## Lister les runspaces existants

Pour avoir la liste et détails des runsapces dans notre espace de travail courant, on utilise la commande Get-Runspace
En ouvrant Powershell, on créé simultannéement un runspace géré par Powershell lui même.

```powershell
PS C:\Users\Dell> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy

```
Afin de décourvir les possiblilités avancées offertes par les runspaces, je vais créer un autre runspace différent de celui créé par défaut.

## Création d'un runspace

Il y a plusieurs façon de créer et gérer les runspaces mais la façon la plus simple sinon efficace est d'utiliser la méthode CreateRunspace() du fabricateur runspacefactory de la classe [System.Management.Automation.Runspaces.RunspaceFactory]

```powershell
# Créer un nouveau Runspace
$runspace = [runspacefactory]::CreateRunspace()

PS C:\Users\Dell> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  3 Runspace3       localhost       Local         BeforeOpen    None

```

Le nouveau runspace (Runspace3) prend le prochain nom et ID sequentiellement disponible, en occurrence Runspace3 avec ID 3.
La variable $runspace incarne l'instance ainsi créée.

Pour personnaliser le nom du runspace,
```powershell
$runspace.Name = "NewName"
```

On remarque son état BeforeOpen qui est l'état initial dès la création. Avoir un runspace est bien mais qui peut traiter les instructions, c'est encore mieux ;). Pour cela, on va le mettre à l'état Opened via la commande open().

```powershell
PS C:\Users\Dell> $runspace.Open()

PS C:\Users\Dell> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  3 Runspace3       localhost       Local         Opened        Available
```

A l'état Opened, un runspace peut traiter des instructions.


## Exécuter du code dans le Runspace
On va créer un bloc de script dans un pipeline qui contient des instructions
```powershell
$pipeline = [powershell]::Create().AddScript({
    "Hello, Runspace!"
})

$pipeline.Runspace = $runspace
$result = $pipeline.Invoke()
```
## Fermer le pipeline et le Runspace

```powershell

$pipeline.Dispose()
$runspace.Close()

## Libérer les ressources
$runspace.Dispose()
```

## Création d'un runspace avec argument
Il est possible de préparer l'environnement d'exécution d'un runspace lors de sa création en lui passant un ensemble de paramètres.

```powershell
## Initialiser l'environnement d'un runspace
$init = [System.Management.Automation.Runspaces.InitialSessionState]::Create()
## $init | Get-Member

$var1 = [System.Management.Automation.Runspaces.SessionStateVariableEntry]::new("Pi", 3.14 , '')
$var2 = [System.Management.Automation.Runspaces.SessionStateVariableEntry]::new("double", 6 , '')

$init.LanguageMode = 'FullLanguage'
$init.Variables.Add($var1)
$init.Variables.Add($var2)

$Runspace2 = [runspacefactory]::CreateRunspace($init)
$Runspace2.Open()

# Récuperation des variables dans runspace

PS C:\Users\Dell> $Runspace2.SessionStateProxy.GetVariable("double")
6

PS C:\Users\Dell> $Runspace2.SessionStateProxy.GetVariable("pi")
3,14

```

```powershell
PS C:\Windows\system32> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  2 Runspace2       localhost       Local         Opened        Available

```
## Se connecter au runspace d'une autre session Powershell

```powershell
## Trouvons le PID de notre session Powershell en cours
PS C:\Users\Dell> $pid
17672

PS C:\Users\Dell> $ConnectInfo = [System.Management.Automation.Runspaces.NamedPipeConnectionInfo]::new(17672)

PS C:\Users\Dell> $Runspace2 = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($ConnectInfo)

PS C:\Users\Dell> $Runspace2.Open()

PS C:\Users\Dell> Get-Runspace

 Id Name            ComputerName    Type          State         Availability
 -- ----            ------------    ----          -----         ------------
  1 Runspace1       localhost       Local         Opened        Busy
  6 Runspace6       localhost       Local         Opened        Available
  7 RemoteHost      localhost       Local         Opened        Available


# Création du Pipeline dans lequel passera le code du runspace créé
$Pipeline = [PowerShell]::Create()

# Liaison entre pipeline et runspace
PS C:\Users\Dell> $Pipeline.Runspace = $Runspace2

# Envoi du traitement : chercher le $PID du runspace
PS C:\Users\Dell> $Pipeline.AddScript("`$PID");


Commands            : System.Management.Automation.PSCommand
Streams             : System.Management.Automation.PSDataStreams
InstanceId          : b7680a77-bae4-452d-871b-e30c4530b653
InvocationStateInfo : System.Management.Automation.PSInvocationStateInfo
IsNested            : False
HadErrors           : False
Runspace            : System.Management.Automation.RemoteRunspace
RunspacePool        :
IsRunspaceOwner     : False
HistoryString       :


# Lancer le script
PS C:\Users\Dell> $Pipeline.Invoke();
17672

# Le PID retourné par ce runspace est la session Powershell sur laquelle on s'est conncectéé

```

## Conclusion

Les runspaces sont un outil puissant qui peut être utilisé pour une variété de tâches. Ils sont utiles pour la parallélisation des tâches, l'isolation des ressources et l'exécution de commandes sur des systèmes distants.
