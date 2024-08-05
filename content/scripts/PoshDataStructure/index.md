---
title: "Structures de données dans Powershell👋"
description: "Les structures de données sont un moyen efficace de stocker et de manipuler des données dans PowerShell."
summary: "You can use blog posts for announcing product updates and features."
date: 2023-09-07T16:27:22+02:00
lastmod: 2023-09-07T16:27:22+02:00
draft: false
weight: 20
categories: []
tags: []
contributors: []
pinned: false
homepage: false

seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---


## Utilisation des structures de données dans PowerShell

PowerShell est un langage de script puissant qui offre un large éventail de fonctionnalités, y compris la prise en charge des structures de données. Les structures de données sont un moyen efficace de stocker et de manipuler des données dans PowerShell.

## Types de structures de données

PowerShell prend en charge une variété de structures de données, notamment :

* Tableaux
* ArrayList
* Hashtables
* Enumerations

### Tableaux

Les tableaux sont une structure de données de base qui stocke une collection d'éléments de même type ou de types différents. Les éléments d'un tableau peuvent être de n'importe quel type, y compris des chaînes, des nombres, des tableaux, etc.

```powershell
 $myArray = @("serveur01", 11, 3.141592, 'A')

```

Pour accéder à un élément du tableau, on utilise l'index de cet élément. Par exemple, on accède au premier et troisième élément du tableau `myArray`, comme suite :

```powershell
$myArray[0]
$myArray[2]
```
### ArrayList

Les ArrayList sont une structure de données similaires aux tableaux, mais elles permettent d'ajouter et de supprimer des éléments plus facilement.

```powershell
PS C:\Windows\system32> $myArrayList = New-Object System.Collections.ArrayList
PS C:\Windows\system32> $myArrayList.Add("server01")
PS C:\Windows\system32> $myArrayList.Add("server02")
PS C:\Windows\system32> $myArrayList.Add("server03")

PS C:\Windows\system32> $myArrayList
server01
server02
server03

PS C:\Windows\system32> $myArrayList.Remove("server02")

PS C:\Windows\system32> $myArrayList
server01
server03

PS C:\Windows\system32> $myArrayList.Insert(0,"server01")

PS C:\Windows\system32> $myArrayList
server01
server01
server03

```

### Hashtables

Les hashtables ou dictionnaires sont une structure de données qui stocke des données sous forme de paires clé-valeur. Les clés des hashtables peuvent être de n'importe quel type mais doivent être uniques.

```powershell
$web = @{
    "server-web01" = "192.168.10.5"
    "server-web02" = "192.168.10.6"
    "server-web03" = "192.168.10.7"
    1=100
    3.14 = "Pi"
}

PS C:\Windows\system32> $web["server-web02"].GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object



PS C:\Windows\system32> $web[1].GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType

```

Pour accéder à une valeur d'une hashtable, on utilise la clé de la valeur. Par exemple, pour accéder à la valeur associée à la clé `server-web02` du hashtable `web`, on utilise la syntaxe suivante :

```powershell
PS C:\Windows\system32> $web["server-web02"]
192.168.10.6
```
### Hashtable Vs ArrayList

* Hashtable est un type de données `non typé`, ce qui signifie que les clés et les valeurs peuvent être de n'importe quel type. ArrayList est un type de données typé, ce qui signifie que les éléments de la liste doivent être du même type.
* Hashtable est `thread-safe`, ce qui signifie qu'il peut être utilisé par plusieurs threads en même temps sans risque de corruption des données. ArrayList n'est pas thread-safe.
Hashtable est plus gourmand en mémoire que ArrayList.

* En général, Hashtable est un bon choix si vous avez besoin de pouvoir rechercher rapidement un élément par sa clé. ArrayList est un bon choix si vous avez besoin d'ajouter ou de supprimer des éléments rapidement.


### Enumerations

Les enumerations sont une structure de données qui sont utilisées pour représenter un ensemble d'éléments. Les éléments d'une enumeration sont des constantes.

Pour accéder à une valeur d'une enumeration, vous pouvez utiliser son nom. Par exemple, pour accéder à la valeur `Value1` de l'enumeration `MyEnumeration`, vous pouvez utiliser la syntaxe suivante :

```powershell
PS C:\Windows\system32> # Déclaration de l'Enum
Enum Etat {
  ALLUME
  ETEINT
}

# créer une instance et la stocker dans une var $etat typé pareil
PS C:\Windows\system32> [Etat]$etat = [Etat]::ALLUME
PS C:\Windows\system32> $etat = [Etat]::ETEINT

# Vérifier le type de la var
$etat.GetType()

# Test sur la valeur de la var
switch ($etat)
{
    "ALLUME" {"Le serveur est allumé"; continue }
    "ETEINT" {"Le serveur est éteint"; continue }

}

Le serveur est éteint

```
Accès aux valeurs des Enum sous forme d'une constante et un entier


```powershell
PS C:\Windows\system32> ($etat).value__
1

PS C:\Windows\system32> [int]$etat
1

PS C:\Windows\system32> [Enum]::GetNames( [Etat] )
ALLUME
ETEINT

PS C:\Windows\system32> [System.Enum]::GetValues([Etat]) |
foreach { [PSCustomObject]@{ValueName = $_; IntValue = [int]$_ } }

ValueName IntValue
--------- --------
   ALLUME        0
   ETEINT        1

```


## Cas d'usage

Voici quelques exemples d'utilisation des structures de données dans PowerShell :

* Pour stocker une liste de noms de fichiers, on peut utiliser un tableau.
* Pour stocker un ensemble de paramètres d'une application, on peut utiliser une hashtable.
* Pour stocker une liste d'éléments qui peuvent être ajoutés ou supprimés facilement, on peut utiliser un ArrayList.
* Pour représenter un ensemble d'éléments, on peut utiliser une enumeration.

## Conclusion

Les structures de données sont un outil puissant qui peut être utilisé pour améliorer la performance et la flexibilité des scripts PowerShell.
