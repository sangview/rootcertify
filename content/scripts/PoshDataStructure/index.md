---
title: "Structures de données dans Powershell"
description: ""
summary: "Les structures de données sont un moyen efficace de stocker et de manipuler des données dans PowerShell."
date: 2023-09-07T16:27:22+02:00
lastmod: 2023-09-07T16:27:22+02:00
draft: false
weight: 20
categories: ["Scripts"]
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

Pour user de plein potentiels de PowerShell, apprendre à maitriser les structures de données est une nécessité. Les structures de données sont un moyen efficace de stocker et de manipuler des données dans PowerShell.

## Types de structures de données

PowerShell prend en charge une variété de structures de données, comme :

* Tableaux
* ArrayList
* Hashtables
* Enumerations

### Tableaux

Les tableaux sont une structure de données de base qui stocke une collection d'éléments de même type ou de types différents. Ces éléments peuvent être de n'importe quel type, y compris des chaînes, des nombres, des tableaux, etc.

```powershell
 $myArray = @("serveur01", 11, 3.141592, 'A')

```
Pour accéder à un élément du tableau, on utilise l'index de cet élément. Par exemple, on accède au premier et troisième élément du tableau `myArray`, comme suite :

```powershell
$myArray[0]
$myArray[2]


```
{{< callout context="note" title="Note" icon="outline/info-circle" >}}
 La taille des tableaux n'est pas modifiable et en Powershell comme dans la plupart des langages, le premier index des tableaux est le 0
{{< /callout >}}


### ArrayList

Les ArrayList sont une structure de données similaires aux tableaux, mais elles permettent d'ajouter et de supprimer des éléments plus facilement grâce à son mode de stockage en mémoire sous forme de liste. Les ArrayList n'ont pas une taille fixe contrairement aux tableaux. Tout comme les tableaux, on peut accéder aux éléments avec leur index.

```powershell
$myArrayList = New-Object System.Collections.ArrayList
$myArrayList.Add("server01")
$myArrayList.Add("server02")
$myArrayList.Add("server03")

$myArrayList
server01
server02
server03

$myArrayList.Remove("server02")

$myArrayList
server01
server03

$myArrayList.Insert(0,"server01")

$myArrayList
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

$web["server-web02"].GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object



$web[1].GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32                                    System.ValueType

```

Pour accéder à une valeur d'une hashtable, on utilise la clé de la valeur. Par exemple, pour accéder à la valeur associée à la clé `server-web02` du hashtable `web`, on utilise la syntaxe suivante :

```powershell
$web["server-web02"]
192.168.10.6
```
{{< callout context="tip" title="Vous savez ?" icon="outline/rocket" >}}
Les hashtables sont très utilisés dans les outils et scripts de configuration des environnements
{{< /callout >}}


### Hashtable Vs ArrayList

* Hashtable est `thread-safe`, ce qui signifie qu'il peut être utilisé par plusieurs threads en même temps sans risque de corruption des données. ArrayList n'est pas thread-safe.
Hashtable est plus gourmand en mémoire que ArrayList.

* En général, Hashtable est un bon choix si vous avez besoin de pouvoir rechercher rapidement un élément par sa clé. ArrayList est un bon choix si vous avez besoin d'ajouter ou de supprimer des éléments rapidement.


### Enumerations

Les enumerations sont une structure de données qui sont utilisées pour représenter un ensemble d'éléments constants. Les éléments d'une enumeration sont des constantes.

Pour accéder à une valeur d'une enumeration, on peut utiliser son nom. Par exemple, pour accéder à la valeur `ETEINT` de l'enumeration `Etat`, on utilise la syntaxe suivante :

```powershell
# Déclaration de l'Enum
Enum Etat {
  ALLUME
  ETEINT
}

# créer une instance et la stocker dans une var $etat typé pareil
[Etat]$etat = [Etat]::ALLUME
$etat = [Etat]::ETEINT

# Vérifier le type de la var
$etat.GetType()

# Test sur la valeur de la var
switch ($etat)
{
    "ALLUME" {"Le serveur est allumé"; continue }
    "ETEINT" {"Le serveur est éteint"; continue }

}
```

{{< figure
  src="images/site-data-structures-Enum-1.png"
  alt="ISE - Arrêt des runspaces"
  caption="ISE - test d'état"
>}}




**Accès aux valeurs des Enum sous forme d'une constante et un entier**


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


## Quelques Cas d'usage

Voici quelques exemples d'utilisation des structures de données dans PowerShell :

* Pour stocker une liste de noms de fichiers, on peut utiliser un tableau.
* Pour stocker un ensemble de paramètres d'une application, on peut utiliser une hashtable.
* Pour stocker une liste d'éléments qui peuvent être ajoutés ou supprimés facilement, on peut utiliser un ArrayList.
* Pour représenter un ensemble d'éléments constants, on peut utiliser une enumeration.

## Conclusion

Les structures de données sont un outil puissant qui peut être utilisé pour améliorer la performance et la flexibilité des scripts PowerShell.
